---
title: "Configure ConfigMaps in Applications"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 1.4 </b>"
---

#### Configure ConfigMaps in Applications

Trong phần này, chúng ta sẽ tìm hiểu về cách cấu hình configmaps trong các ứng dụng

#### ConfigMaps
- Có 2 giai đoạn trong việc cấu hình ConfigMaps.
  - Đầu tiên, tạo các configMaps
  - Thứ hai, Inject chúng vào pod.
- Có 2 cách tạo ra một configmap.
  - Cách mạnh mẽ (Imperative way)
    ```
    $ kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
    $ kubectl create configmap app-config --from-file=app_config.properties (Cách khác)
    ```
![EKS](/images/part1/1.4/00016.png?featherlight=false&width=90pc)
    
  - Cách Tuyên bố (Declarative way)
    
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: app-config
    data:
     APP_COLOR: blue
     APP_MODE: prod
    ```
    ```
    Tạo một tệp định nghĩa config map và chạy lệnh 'kubectl create` để triển khai nó.
    $ kubectl create -f config-map.yaml
    ```
![EKS](/images/part1/1.4/00017.png?featherlight=false&width=90pc)
    
 #### Xem ConfigMaps
 - Để xem configMaps
   ```
   $ kubectl get configmaps (hoặc)
   $ kubectl get cm
   ```
 - Để mô tả configmaps
   ```
   $ kubectl describe configmaps
   ```
   
![EKS](/images/part1/1.4/00018.png?featherlight=false&width=90pc)
   
 #### ConfigMap trong Pods
 - Inject configmap vào pod
   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: simple-webapp-color
   spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
      - containerPort: 8080
      envFrom:
      - configMapRef:
          name: app-config
   ```
   ```
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     APP_COLOR: blue
     APP_MODE: prod
   ```
   ```
   $ kubectl create -f pod-definition.yaml
   ```
  
![EKS](/images/part1/1.4/00019.png?featherlight=false&width=90pc)
   
 #### Có các cách khác để inject các biến cấu hình vào pod   
 - Bạn có thể inject nó như một **`Biến Môi trường`** 
 - Bạn có thể inject nó như một tệp trong một **`Volume`**
   
 #### Tài liệu Tham khảo K8s
 - https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
 - https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data
 - https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-files