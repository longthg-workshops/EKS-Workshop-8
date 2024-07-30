---
title: "ACK"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.1.2 </b>"
---

Mặc định, thành phần **Carts** trong ứng dụng mẫu sử dụng một phiên bản cục bộ của DynamoDB chạy như một pod trong cụm EKS gọi là `carts-dynamodb`. Trong phần này của bài lab, chúng ta sẽ cung cấp một bảng DynamoDB dựa trên đám mây của Amazon cho ứng dụng của chúng ta bằng cách sử dụng tài nguyên tùy chỉnh của Kubernetes và chỉ định cho triển khai **Carts** sử dụng bảng DynamoDB mới được cung cấp thay vì bản sao cục bộ.

![EKS](/images/0006/00053.png?featherlight=false&width=90pc)

SDK Java của AWS trong thành phần **Carts** có thể sử dụng IAM Roles để tương tác với các dịch vụ AWS, điều này có nghĩa là chúng ta không cần phải truyền thông tin đăng nhập, từ đó giảm bớt bề mặt tấn công. Trong ngữ cảnh của EKS, IRSA cho phép chúng ta định nghĩa các IAM Roles riêng biệt cho các pod để ứng dụng tiêu thụ. Để tận dụng IRSA, trước tiên chúng ta cần:

- Tạo một Tài khoản Dịch vụ Kubernetes trong không gian tên Carts
- Tạo một Chính sách IAM với các quyền DynamoDB cần thiết
- Tạo một Vai trò IAM trong AWS với các quyền trên
- Ánh xạ Tài khoản Dịch vụ để sử dụng vai trò IAM bằng cách sử dụng Annotations trong định nghĩa Tài khoản Dịch vụ.

May mắn thay, chúng ta có một dòng lệnh ngắn gọn để hỗ trợ quá trình này. Chạy lệnh dưới đây:

```bash
$ eksctl create iamserviceaccount --name carts-ack \
  --namespace carts --cluster $EKS_CLUSTER_NAME \
  --role-name ${EKS_CLUSTER_NAME}-carts-ack \
  --attach-policy-arn $DYNAMODB_POLICY_ARN --approve
2023-10-31 16:20:46 [i]  1 iamserviceaccount (carts/carts-ack) đã được bao gồm (dựa trên các quy tắc bao gồm/loại trừ)
2023-10-31 16:20:46 [i]  1 tác vụ: {
    2 các tác vụ tuần tự: {
        tạo IAM role cho tài khoản dịch vụ "carts/carts-ack",
        tạo tài khoản dịch vụ "carts/carts-ack",
    } }2023-10-31 16:20:46 [ℹ]  đang xây dựng ngăn xếp iamserviceaccount "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-ack"
2023-10-31 16:20:46 [i]  triển khai ngăn xếp "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-ack"
2023-10-31 16:20:47 [i]  đang chờ ngăn xếp CloudFormation "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-ack"
2023-10-31 16:21:17 [i]  đang chờ ngăn xếp CloudFormation "eksctl-eks-workshop-addon-iamserviceaccount-carts-carts-ack"
2023-10-31 16:21:17 [i]  đã tạo tài khoản dịch vụ "carts/carts-ack"
```

`eksctl` cung cấp một ngăn xếp CloudFormation để hỗ trợ quản lý các tài nguyên này có thể được thấy trong đầu ra ở trên.

Để tìm hiểu thêm về cách IRSA hoạt động, hãy đi đến [đây](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html).

---

Bây giờ, hãy khám phá cách chúng ta sẽ tạo Bảng DynamoDB thông qua một tài liệu Kubernetes

```file
manifests/modules/automation/controlplanes/ack/dynamodb/dynamodb-create.yaml
```

:::info

Các độc giả sắc sảo sẽ nhận thấy YAML Spec tương tự như các điểm cuối API và cuộc gọi cho DynamoDB như `tableName` và `attributeDefinitions`.

:::

Tiếp theo, chúng ta sẽ cần cập nhật điểm cuối khu vực cho DynamoDB trong Configmap được sử dụng bởi Kustomization để cập nhật triển khai **Carts**.

```file
manifests/modules/automation/controlplanes/ack/dynamodb/dynamodb-ack-configmap.yaml
```

Sử dụng tiện ích `envsubst`, chúng ta sẽ viết lại biến môi trường AWS_REGION vào tài liệu và áp dụng tất cả các cập nhật vào cụm. Chạy lệnh sau

```bash wait=10
$ kubectl kustomize ~/environment/eks-workshop/modules/automation/controlplanes/ack/dynamodb \
  | envsubst | kubectl apply -f-
namespace/carts không thay đổi
serviceaccount/carts không thay đổi
configmap/carts không thay đổi
configmap/carts-ack đã được tạo
service/carts không thay đổi
service/carts-dynamodb không thay đổi
triển khai.apps/carts đã được cấu hình
triển khai.apps/carts-dynamodb không thay đổi
table.dynamodb.services.k8s.aws/items đã được tạo
$ kubectl rollout status -n carts deployment/carts --timeout=120s
```

:::info
Lệnh này 'xây dựng' các tài liệu sử dụ

ng lệnh kubectl kustomize, chuyển nó đến `envsubst` và sau đó đến kubectl apply. Điều này làm cho việc tạo mẫu tài liệu và điền thông tin vào chúng khi chạy trở nên dễ dàng.
:::

Các điều khiển ACK trong cụm sẽ phản ứng với các tài nguyên mới này và cung cấp cơ sở hạ tầng AWS mà chúng ta đã biểu thị bằng các tài liệu trước đó. Hãy kiểm tra xem ACK đã tạo bảng chưa bằng cách chạy

```bash
$ kubectl wait table.dynamodb.services.k8s.aws items -n carts --for=condition=ACK.ResourceSynced --timeout=15m
table.dynamodb.services.k8s.aws/items điều kiện đã được đáp ứng
$ kubectl get table.dynamodb.services.k8s.aws items -n carts -ojson | yq '.status."tableStatus"'
ACTIVE
```

Và bây giờ để kiểm tra xem Bảng đã được tạo bằng cách sử dụng AWS CLI, chạy

```bash
$ aws dynamodb list-tables

{
    "TableNames": [
        "eks-workshop-carts-ack"
    ]
}
```

Đầu ra này cho chúng ta biết rằng bảng mới đã được tạo!