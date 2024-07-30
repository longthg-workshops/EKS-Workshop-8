---
title: "How does ACK work?"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1.1 </b>"
---

#### Triển khai ACK Controllers

Mỗi ACK service controller được đóng gói vào một hình ảnh container riêng biệt và được xuất bản trong một kho chứa công khai tương ứng với mỗi ACK service controller riêng. Đối với mỗi dịch vụ AWS mà chúng tôi muốn triển khai, các tài nguyên cho controller tương ứng phải được cài đặt trong cụm Amazon EKS. Chúng tôi đã thực hiện điều này trong bước `prepare-environment`. Helm charts và các hình ảnh container chính thức cho ACK có sẵn [tại đây](https://gallery.ecr.aws/aws-controllers-k8s).

Trong phần này của workshop, khi chúng tôi sẽ làm việc với Amazon DynamoDB, các ACK controllers cho DynamoDB đã được cài đặt sẵn trong cụm, chạy dưới dạng một deployment trong không gian tên Kubernetes riêng của nó. Để xem những gì đang diễn ra, hãy chạy lệnh sau.

```bash
$ kubectl describe deployment ack-dynamodb -n ack-dynamodb
```


kubectl cũng chứa các tùy chọn `-oyaml` và `-ojson` hữu ích, giúp trích xuất entin YAMl hoặc JSON đầy đủ của định nghĩa deployment thay vì đầu ra được định dạng.


Controller này sẽ theo dõi các tài nguyên tùy chỉnh của Kubernetes cho DynamoDB như `dynamodb.services.k8s.aws.Table` và sẽ thực hiện các cuộc gọi API đến điểm cuối DynamoDB dựa trên cấu hình trong các tài nguyên này được tạo ra. Khi các tài nguyên được tạo ra, controller sẽ cung cấp các cập nhật trạng thái cho các tài nguyên tùy chỉnh trong các trường `Status`. Để biết thêm thông tin về cấu trúc của manifest, hãy nhấp vào [đây](https://aws-controllers-k8s.github.io/community/reference/).

Nếu bạn muốn đào sâu hơn vào cơ chế của những đối tượng và cuộc gọi API mà controller lắng nghe, hãy chạy:

```bash
$ kubectl get crd
```
