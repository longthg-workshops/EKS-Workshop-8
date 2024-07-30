---
title: "Accessing AWS CodeCommit"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 2.2.1 </b>"
---

#### Accessing AWS CodeCommit

Trong môi trường thí nghiệm của chúng ta, một kho lưu trữ AWS CodeCommit đã được tạo, nhưng trước khi Cloud9 có thể kết nối được với nó, chúng ta cần hoàn thành một số bước.

Chúng ta có thể thêm các khóa SSH cho CodeCommit vào tệp known_hosts để tránh những cảnh báo sau này:

```bash
$ ssh-keyscan -H git-codecommit.${AWS_REGION}.amazonaws.com &> ~/.ssh/known_hosts
```

Và chúng ta có thể thiết lập một danh tính mà Git sẽ sử dụng cho các commit của chúng ta:

```bash
$ git config --global user.email "you@eksworkshop.com"
$ git config --global user.name "Tên của bạn"
```



