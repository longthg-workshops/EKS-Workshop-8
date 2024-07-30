---
title: "Rolling Updates and Rollback"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

#### Rolling Updates and Rollback

Trong phần này, chúng ta sẽ tìm hiểu về cập nhật liên tục và quay lại phiên bản trước trong quá trình triển khai

#### Triển khai và Phiên bản trong Quá trình Triển khai

![EKS](/images/part1/1.1/0001.png?featherlight=false&width=90pc)
  
#### Các lệnh Triển khai

- Bạn có thể xem trạng thái của quá trình triển khai bằng lệnh dưới đây
  ```
  $ kubectl rollout status deployment/myapp-deployment
  ```
- Để xem lịch sử và các bản sửa đổi
  ```
  $ kubectl rollout history deployment/myapp-deployment
  ```
 
![EKS](/images/part1/1.1/0002.png?featherlight=false&width=90pc)
  
#### Chiến lược Triển khai
- Có 2 loại chiến lược triển khai
  1. Tạo lại (Recreate)
  2. Cập nhật liên tục (RollingUpdate) (Chiến lược Mặc định)
  
![EKS](/images/part1/1.1/0003.png?featherlight=false&width=90pc)
  
#### kubectl apply
- Để cập nhật một quá trình triển khai, chỉnh sửa quá trình triển khai và thực hiện các thay đổi cần thiết và lưu lại nó. Sau đó chạy lệnh dưới đây.
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
   name: myapp-deployment
   labels:
    app: nginx
  spec:
   template:
     metadata:
       name: myap-pod
       labels:
         app: myapp
         type: front-end
     spec:
      containers:
      - name: nginx-container
        image: nginx:1.7.1
   replicas: 3
   selector:
    matchLabels:
      type: front-end       
  ```
  ```
  $ kubectl apply -f deployment-definition.yaml
  ```
- Cách thay thế khác để cập nhật một quá trình triển khai, ví dụ để cập nhật một hình ảnh.
  ```
  $ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
  ```
![EKS](/images/part1/1.1/0004.png?featherlight=false&width=90pc)
  
#### Tạo lại vs Cập nhật liên tục
  
![EKS](/images/part1/1.1/0005.png?featherlight=false&width=90pc)
  
#### Nâng cấp

![EKS](/images/part1/1.1/0006.png?featherlight=false&width=90pc)
  
#### Quay lại phiên bản trước
  
![EKS](/images/part1/1.1/0007.png?featherlight=false&width=90pc)
  
- Để hoàn tác một thay đổi
  ```
  $ kubectl rollout undo deployment/myapp-deployment
  ```
  
#### kubectl create
- Để tạo một quá trình triển khai
  ```
  $ kubectl create deployment nginx --image=nginx
  ```
#### Tóm tắt các lệnh kubectl
```
$ kubectl create -f deployment-definition.yaml
$ kubectl get deployments
$ kubectl apply -f deployment-definition.yaml
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
$ kubectl rollout status deployment/myapp-deployment
$ kubectl rollout history deployment/myapp-deployment
$ kubectl rollout undo deployment/myapp-deployment
```

![EKS](/images/part1/1.1/0008.png?featherlight=false&width=90pc)
 
#### Tài liệu Tham khảo K8s
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment
- https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment