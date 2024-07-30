---
title: "Multi-Container Pods"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 1.5 </b>"
---

#### Multi-Container Pods

Trong phần này, chúng ta sẽ xem xét về các pod chứa nhiều container.

#### Kiến trúc Monolith và Microservices

![EKS](/images/part1/1.5/00032.png?featherlight=false&width=90pc)
  
#### Các Pod Chứa Nhiều Container

![EKS](/images/part1/1.5/00033.png?featherlight=false&width=40pc)
  
- Để tạo một pod mới chứa nhiều container, hãy thêm thông tin về container mới vào tệp định nghĩa của pod.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp
    labels:
      name: simple-webapp
  spec:
    containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
      - ContainerPort: 8080
    - name: log-agent
      image: log-agent
  ```
 
#### Tài liệu Tham Khảo Kubernetes
- https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/