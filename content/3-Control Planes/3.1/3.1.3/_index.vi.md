---
title: "ACK"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.1.3 </b>"
---

Khi tạo mới hoặc cập nhật các tài nguyên, cấu hình ứng dụng cũng cần được cập nhật để sử dụng các tài nguyên mới này. Biến môi trường là lựa chọn phổ biến cho các nhà phát triển ứng dụng để lưu trữ cấu hình, và trong Kubernetes, chúng ta có thể truyền biến môi trường vào các container thông qua trường `env` của `container` [spec](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/) khi tạo các deployments.

Bây giờ, có hai cách để thực hiện điều này.

- Đầu tiên, là Configmaps. Configmaps là một tài nguyên cốt lõi trong Kubernetes cho phép chúng ta truyền các thành phần cấu hình như biến môi trường, trường văn bản và các mục khác trong định dạng key-value để sử dụng trong pod specs.
- Sau đó, chúng ta có secrets (không được mã hóa theo thiết kế - điều này quan trọng để nhớ) để đẩy các điều như mật khẩu/bí mật.

Tài nguyên tùy chỉnh ACK `FieldExport` [custom resource](https://aws-controllers-k8s.github.io/community/docs/user-docs/field-export/) được thiết kế để nối các khoảng cách giữa việc quản lý mặt phẳng kiểm soát của các tài nguyên ACK của bạn và việc sử dụng _các thuộc tính_ của những tài nguyên đó trong ứng dụng của bạn. Điều này cấu hình một bộ điều khiển ACK để xuất bất kỳ trường `spec` hoặc `status` nào từ một tài nguyên ACK vào một Kubernetes ConfigMap hoặc Secret. Những trường này sẽ được tự động cập nhật khi bất kỳ giá trị trường nào thay đổi. Sau đó, bạn có thể mount ConfigMap hoặc Secret vào Pods Kubernetes của bạn như các biến môi trường có thể tiếp nhận các giá trị đó.

Tuy nhiên, trong trường hợp của DynamoDB trong phần này của lab, chúng ta sẽ sử dụng một ánh xạ trực tiếp của điểm cuối API bằng cách sử dụng ConfigMaps, và chỉ cần cập nhật điểm cuối DynamoDB như một biến môi trường.

```file
manifests/modules/automation/controlplanes/ack/dynamodb/deployment.yaml
```

Trong mẫu triển khai mới (mà chúng ta đã áp dụng), chúng ta đang cập nhật thuộc tính `envFrom` `configMapRef` thành `carts-ack`. Điều này cho Kubernetes biết để lấy biến môi trường từ ConfigMap mới mà chúng ta đã tạo.

```file
manifests/modules/automation/controlplanes/ack/dynamodb/dynamodb-ack-configmap.yaml
```

Và khi chúng ta áp dụng những mẫu này bằng Kustomize như chúng ta đã làm trong phần trước, thành phần **Carts** cũng được cập nhật.

---

Bây giờ, làm sao để biết ứng dụng đang hoạt động với bảng DynamoDB mới?

Một NLB đã được tạo ra để tiếp cận ứng dụng mẫu để kiểm tra, cho phép chúng ta tương tác trực tiếp với ứng dụng thông qua trình duyệt:

```bash
$ kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}"
k8s-ui-uinlb-a9797f0f61.elb.us-west-2.amazonaws.com
```

:::info
Vui lòng lưu ý rằng điểm cuối thực sự sẽ khác khi bạn chạy lệnh này vì một điểm cuối Máy chủ Tải Trọng Mạng mới sẽ được cung cấp.
:::

Để chờ đến khi máy chủ tải trọng được hoàn thành, bạn có thể chạy lệnh này:

```bash timeout=610
$ wait-for-lb $(kubectl get service -n ui ui-nlb -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}")
```

Khi máy chủ tải trọng được cung cấp, bạn có thể truy cập nó bằng cách dán URL vào trình duyệt web của bạn. Bạn sẽ thấy giao diện người dùng từ cửa hàng web được hiển thị và sẽ có thể điều hướng xung quanh trang web như một người dùng.

![EKS](/images/0006/00054.png?featherlight=false&width=90pc)

Để xác minh rằng mô-đun **Carts** thực sự đang sử dụng bảng DynamoDB chúng ta vừa cung cấp, hãy thử thêm một vài mục vào giỏ hàng.

![EKS](/images/0006/00055.png?featherlight=false&width=90pc)

Và để kiểm tra xem các mục có trong giỏ hàng không, chạy

```bash
$ aws dynamodb scan --table-name "${EKS_CLUSTER_NAME}-carts-ack"
```

Chúc mừng! Bạn đã thành công trong việc tạo các Tài nguyên AWS mà không cần rời khỏi Kubernetes API!
