---
title: "Control Planes"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3. </b>"
---

#### Control Planes

Các framework Control Plane cho phép bạn quản lý tài nguyên AWS trực tiếp từ Kubernetes bằng cách sử dụng giao diện dòng lệnh tiêu chuẩn của Kubernetes, `kubectl`. Điều này được thực hiện bằng cách mô hình hóa các dịch vụ quản lý AWS như [Định nghĩa Tài nguyên Tùy chỉnh (CRDs)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) trong Kubernetes và áp dụng các định nghĩa đó vào cụm của bạn. Điều này có nghĩa là một nhà phát triển có thể mô hình hóa toàn bộ kiến ​​trúc ứng dụng từ container đến các dịch vụ quản lý AWS, hỗ trợ từ một tài liệu YAML duy nhất. Chúng tôi dự đoán rằng các Control Planes sẽ giúp giảm thời gian cần thiết để tạo ứng dụng mới và hỗ trợ duy trì các giải pháp native cloud trong trạng thái mong muốn.

Hai dự án mã nguồn mở phổ biến cho Control Planes là [AWS Controllers for Kubernetes (ACK)](https://aws-controllers-k8s.github.io/community/) và dự án CNCF đang được ủy quyền là [Crossplane](https://www.crossplane.io/), cả hai đều hỗ trợ Dịch vụ AWS. Module hướng dẫn này tập trung vào hai dự án này.