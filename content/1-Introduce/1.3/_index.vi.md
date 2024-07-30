---
title: "Commands and Arguments in Kubernetes"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 1.3 </b>"
---

#### Các Lệnh và Đối Số trong Kubernetes

Trong phần này, chúng ta sẽ xem xét về các lệnh và đối số trong Kubernetes.

- Bất cứ điều gì được thêm vào lệnh docker run sẽ được đưa vào thuộc tính **`args`** của tệp định nghĩa pod dưới dạng một mảng.
- Trường lệnh tương ứng với hướng dẫn entrypoint trong Dockerfile nên để tóm lại có 2 trường tương ứng với 2 hướng dẫn trong Dockerfile.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: ubuntu-sleeper-pod
  spec:
   containers:
   - name: ubuntu-sleeper
     image: ubuntu-sleeper
     command: ["sleep2.0"]
     args: ["10"]
  ```
![EKS](/images/part1/1.3/00015.png?featherlight=false&width=90pc)
  
#### Tài liệu Tham Khảo Kubernetes
- https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/