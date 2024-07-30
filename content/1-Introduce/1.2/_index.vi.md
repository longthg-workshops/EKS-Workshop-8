---
title: "Commands and Arguments in Docker"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---

#### Commands and Arguments in Docker

Trong phần này, chúng ta sẽ tìm hiểu về các lệnh và đối số trong docker

- Để chạy một container docker
  ```
  $ docker run ubuntu
  ```
- Để liệt kê các container đang chạy
  ```
  $ docker ps 
  ```
- Để liệt kê tất cả các container bao gồm cả những container đã dừng lại
  ```
  $ docker ps -a
  ```
  
![EKS](/images/part1/1.2/00010.png?featherlight=false&width=90pc)
  
#### Khác biệt so với các máy ảo, các container không dùng để lưu trữ hệ điều hành.
- Các container được dùng để chạy một nhiệm vụ hoặc quy trình cụ thể như để lưu trữ một phiên bản của máy chủ web hoặc máy chủ ứng dụng hoặc máy chủ cơ sở dữ liệu vv.

![EKS](/images/part1/1.2/00011.png?featherlight=false&width=90pc)
  
  
#### Làm thế nào để chỉ định một lệnh khác để bắt đầu container?
- Một tùy chọn là nối thêm một lệnh vào lệnh docker run và theo cách đó nó sẽ ghi đè lệnh mặc định được chỉ định trong hình ảnh.
  ```
  $ docker run ubuntu sleep 5
  ```
- Như vậy khi container bắt đầu nó chạy chương trình sleep, đợi 5 giây và sau đó thoát. Làm thế nào để thực hiện thay đổi đó một cách vĩnh viễn?
  
![EKS](/images/part1/1.2/00012.png?featherlight=false&width=90pc)
  
- Có các cách khác nhau để chỉ định lệnh, hoặc lệnh đơn giản như trong dạng shell hoặc dưới dạng mảng JSON.
 
![EKS](/images/part1/1.2/00013.png?featherlight=false&width=90pc)
  
- Bây giờ, xây dựng hình ảnh docker
  ```
  $ docker build -t ubuntu-sleeper .
  ```
- Chạy container docker
  ```
  $ docker run ubuntu-sleeper
  ```
  
![EKS](/images/part1/1.2/00014.png?featherlight=false&width=90pc)
  
#### Hướng dẫn Entrypoint
- Lệnh entrypoint giống như lệnh command vì bạn có thể chỉ định chương trình sẽ được chạy khi container bắt đầu và bất kỳ điều gì bạn chỉ định trên dòng lệnh.

#### Tài liệu tham khảo K8s
- https://docs.docker.com/engine/reference/builder/#cmd