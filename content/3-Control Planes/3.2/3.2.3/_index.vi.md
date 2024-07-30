---
title: "Crossplane"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.2.3 </b>"
---

### Crossplane và Kubernetes

Ngoài việc cung cấp các tài nguyên điện toán đám mây cá nhân, Crossplane còn cung cấp một lớp trừu tượng cao hơn gọi là Compositions. Compositions cho phép người dùng xây dựng các mẫu có quan điểm cho việc triển khai tài nguyên đám mây. Ví dụ, tổ chức có thể yêu cầu một số thẻ cụ thể phải hiện diện trong tất cả các tài nguyên AWS hoặc thêm các khóa mã hóa cụ thể cho tất cả các bucket Amazon Simple Storage (S3). Các nhóm nền tảng có thể xác định những trừu tượng API tự dịch vụ này trong Compositions và đảm bảo rằng tất cả các tài nguyên được tạo thông qua những Compositions này đều đáp ứng các yêu cầu của tổ chức.

Một `CompositeResourceDefinition` (hoặc XRD) xác định loại và schema của tài nguyên Composite của bạn (XR). Nó cho phép Crossplane biết rằng bạn muốn một loại XR cụ thể nào đó tồn tại và các trường mà XR đó nên có. Một XRD giống như một CustomResourceDefinition (CRD), nhưng một chút có quan điểm hơn. Việc viết một XRD chủ yếu là vấn đề chỉ định một ["schema cấu trúc"](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) OpenAPI.

Trước hết, hãy cung cấp một định nghĩa có thể được sử dụng để tạo bảng DynamoDB bởi các thành viên của nhóm ứng dụng trong namespace tương ứng của họ. Trong ví dụ này, người dùng chỉ cần chỉ định các trường **tên**, **thuộc tính khóa** và **tên chỉ mục**.

```yaml
manifests/modules/automation/controlplanes/crossplane/compositions/composition/definition.yaml
```

Một Composition cho phép Crossplane biết phải làm gì khi có người tạo một tài nguyên Composite. Mỗi Composition tạo ra một liên kết giữa một XR và một tập hợp của một hoặc nhiều Managed Resources - khi XR được tạo, cập nhật hoặc xóa, tập hợp các Managed Resources cũng được tạo, cập nhật hoặc xóa tương ứng.

Composition sau đây cung cấp các tài nguyên được quản lý `Table`

```yaml
manifests/modules/automation/controlplanes/crossplane/compositions/composition/table.yaml
```

Áp dụng điều này vào cụm EKS của chúng ta:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/automation/controlplanes/crossplane/compositions/composition
compositeresourcedefinition.apiextensions.crossplane.io/xdynamodbtables.awsblueprints.io created
composition.apiextensions.crossplane.io/table.dynamodb.awsblueprints.io created
```

Sau khi chúng tôi đã cấu hình Crossplane với các chi tiết của XR mới, chúng tôi có thể tạo một XR trực tiếp hoặc sử dụng một Claim. Thông thường chỉ nhóm chịu trách nhiệm cấu hình Crossplane (thường là một nhóm nền tảng hoặc SRE) có quyền tạo XR trực tiếp. Người khác quản lý XR qua một tài nguyên proxy nhẹ gọi là Composite Resource Claim (hoặc claim tắt).

Với claim này, nhà phát triển chỉ cần chỉ định một **tên bảng DynamoDB mặc định, khóa băm, tên chỉ mục toàn cầu** để tạo bảng. Điều này cho phép nhóm nền tảng hoặc SRE chuẩn hóa các khía cạnh như chế độ thanh toán, dung lượng đọc/viết mặc định, loại chiếu, chi phí và các thẻ liên quan đến cơ sở hạ tầng.

```yaml
manifests/modules/automation/controlplanes/crossplane/compositions/claim/claim.yaml
```

Dọn dẹp bảng Dynamodb được tạo từ phần Managed Resource trước đó.

```bash
$ kubectl delete tables.dynamodb.aws.upbound.io --all --ignore-not-found=true
$ kubectl wait --for=delete tables.dynamodb.aws.upbound.io --all --timeout=5m
```

Tạo bảng bằng cách tạo một `Claim`:

```bash timeout=400
$ cat ~/environment/eks-workshop/modules/automation/controlplanes/crossplane/compositions/claim/claim.yaml \
  | envsubst | kubectl -n carts apply -f -
dynamodbtable.awsblueprints.io/eks-workshop-carts-crossplane created
$ kubectl wait dynamodbtables.awsblueprints.io ${EKS_CLUSTER_NAME}-carts-crossplane -n carts \
  --for=condition=Ready --timeout=5m
```

Việc cung cấp các dịch vụ được quản lý bởi AWS mất một khoảng thời gian, trong trường hợp của DynamoDB lên đến 2 phút. Crossplane sẽ báo cáo trạng thái của việc làm cân nhắc trong trường hợp của tài nguyên Composite và Managed resource của Kubernetes trong trường hợp `SYNCED`.

```bash
$ kubectl get table
NAME                                        READY   SYNCED   EXTERNAL-NAME                   AGE
eks-workshop-carts-crossplane-bt28w-lnb4r   True   True      eks-workshop-carts-crossplane   6s
```

---

Bây giờ, hãy thử hiểu cách bảng DynamoDB được

 triển khai bằng cách sử dụng claim này:

![EKS](/images/0006/00061.png?featherlight=false&width=90pc)

Khi truy vấn claim `DynamoDBTable` được triển khai trong namespace carts, chúng ta có thể quan sát rằng nó chỉ đến và tạo một Composite Resource (XR) `XDynamoDBTable`

```bash
$ kubectl get DynamoDBTable -n carts -o yaml | grep "resourceRef:" -A 3

    resourceRef:
      apiVersion: awsblueprints.io/v1alpha1
      kind: XDynamoDBTable
      name: eks-workshop-carts-crossplane-bt28w
```

Composition `table.dynamodb.awsblueprints.io` hiển thị Loại Tài nguyên Composite (XR-KIND) là `XDynamoDBTable`. Composition này cho phép Crossplane biết phải làm gì khi chúng ta tạo XR `XDynamoDBTable`. Mỗi Composition tạo ra một liên kết giữa một XR và một tập hợp của một hoặc nhiều Managed Resources.

```bash
$ kubectl get composition
NAME                              XR-KIND          XR-APIVERSION               AGE
table.dynamodb.awsblueprints.io   XDynamoDBTable   awsblueprints.io/v1alpha1   143m
```

Trên truy vấn XR `XDynamoDBTable` không giới hạn trong bất kỳ namespace nào, chúng ta có thể quan sát rằng nó tạo ra Managed Resource DynamoDB `Table`.

```bash
$ kubectl get XDynamoDBTable -o yaml | grep "resourceRefs:" -A 3

    resourceRefs:
    - apiVersion: dynamodb.aws.upbound.io/v1beta1
      kind: Table
      name: eks-workshop-carts-crossplane-bt28w-lnb4r
```

---

Khi tài nguyên mới được tạo hoặc cập nhật, các cấu hình ứng dụng cũng cần được cập nhật để sử dụng các tài nguyên mới này. Chúng ta đã cấu hình công việc để sử dụng tên bảng chính xác trong phần trước nên chỉ cần khởi động lại các pod:

```bash
$ kubectl rollout restart -n carts deployment/carts
$ kubectl rollout status -n carts deployment/carts --timeout=2m
deployment "carts" successfully rolled out
```

---

Bây giờ, làm thế nào để biết rằng ứng dụng đang hoạt động với bảng DynamoDB mới?

Một NLB đã được tạo ra để phơi bày ứng dụng mẫu cho việc kiểm tra, cho phép chúng ta tương tác trực tiếp với ứng dụng thông qua trình duyệt:

```bash
$ kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}"
k8s-ui-uinlb-a9797f0f61.elb.us-west-2.amazonaws.com
```

:::info
Vui lòng lưu ý rằng địa chỉ cuối cùng sẽ khác khi bạn chạy lệnh này vì một địa chỉ kết nối Mạng mới sẽ được triển khai.
:::

Để chờ đến khi trình cân bằng tải hoàn thành triển khai, bạn có thể chạy lệnh này:

```bash timeout=610
$ wait-for-lb $(kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}")
```

Khi trình cân bằng tải đã được triển khai, bạn có thể truy cập nó bằng cách dán URL vào trình duyệt web của bạn. Bạn sẽ thấy giao diện người dùng từ cửa hàng web được hiển thị và có thể di chuyển xung quanh trang web như một người dùng.

![EKS](/images/0006/00062.png?featherlight=false&width=90pc)

Để xác minh rằng **Carts** module đang sử dụng bảng DynamoDB mà chúng ta vừa triển khai, hãy thử thêm một vài mục vào giỏ hàng.

![EKS](/images/0006/00063.png?featherlight=false&width=90pc)

Và để kiểm tra xem các mục có trong bảng DynamoDB không, hãy chạy

```bash
$ aws dynamodb scan --table-name "${EKS_CLUSTER_NAME}-carts-crossplane" --query 'Items[].{itemId:itemId,Price:unitPrice}' --output text
PRICE   795
ITEMID  510a0d7e-8e83-4193-b483-e27e09ddc34d
PRICE   385
ITEMID  6d62d909-f957-430e-8689-b5129c0bb75e
PRICE   50
ITEMID  a0a4f044-b040-410d-8ead-4de0446aec7e
```

Chúc mừng! Bạn đã tạo thành công các Tài nguyên AWS mà không cần rời khỏi Kubernetes API!