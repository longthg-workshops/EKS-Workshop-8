---
title: "Tích hợp liên tục và GitOps"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 2.1.4 </b>"
---

#### Continuous Integration and GitOps

Chúng ta đã thành công trong việc khởi động Flux trên cụm EKS và triển khai ứng dụng. Để minh họa cách thực hiện thay đổi trong mã nguồn ứng dụng, xây dựng hình ảnh container mới và tận dụng GitOps để triển khai một hình ảnh mới vào một cụm, chúng ta giới thiệu pipeline tích hợp liên tục. Chúng ta sẽ tận dụng Bộ Công cụ Phát triển AWS (AWS Development Tools) và [nguyên tắc DevOps](https://aws.amazon.com/devops/what-is-devops/) để xây dựng [hình ảnh container đa kiến trúc](https://aws.amazon.com/blogs/containers/introducing-multi-architecture-container-images-for-amazon-ecr/) cho Amazon ECR.
a
Chúng ta đã tạo pipeline tích hợp liên tục trong bước chuẩn bị môi trường và bây giờ chúng ta cần triển khai nó và đang chạy.

![EKS](/images/0006/00035.png?featherlight=false&width=90pc)

Đầu tiên, sao chép kho lưu trữ CodeCommit cho mã nguồn ứng dụng:

```bash
$ git clone ssh://${GITOPS_IAM_SSH_KEY_ID}@git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${EKS_CLUSTER_NAME}-retail-store-sample ~/environment/retail-store-sample-codecommit
```

Tiếp theo, điền kho lưu trữ CodeCommit bằng các nguồn từ kho lưu trữ công khai của [Ứng dụng mẫu](https://github.com/aws-containers/retail-store-sample-app):

```bash
$ git clone https://github.com/aws-containers/retail-store-sample-app ~/environment/retail-store-sample-app
$ git -C ~/environment/retail-store-sample-codecommit checkout -b main
$ cp -R ~/environment/retail-store-sample-app/src ~/environment/retail-store-sample-codecommit
$ cp -R ~/environment/retail-store-sample-app/images ~/environment/retail-store-sample-codecommit
```

Chúng ta sử dụng AWS CodeBuild và định nghĩa `buildspec.yml` để xây dựng hình ảnh `x86_64` và `arm64` mới song song.

```file
manifests/modules/automation/gitops/flux/buildspec.yml
```

```bash
$ cp ~/environment/eks-workshop/modules/automation/gitops/flux/buildspec.yml \
  ~/environment/retail-store-sample-codecommit/buildspec.yml
```

Chúng ta cũng sử dụng AWS CodeBuild để xây dựng `Index Hình ảnh` cho `hình ảnh đa kiến trúc` sử dụng `buildspec-manifest.yml`

```file
manifests/modules/automation/gitops/flux/buildspec-manifest.yml
```

```bash
$ cp ~/environment/eks-workshop/modules/automation/gitops/flux/buildspec-manifest.yml \
  ~/environment/retail-store-sample-codecommit/buildspec-manifest.yml
```

Bây giờ chúng ta đã sẵn sàng đẩy các thay đổi của mình lên CodeCommit và bắt đầu CodePipeline

```bash
$ git -C ~/environment/retail-store-sample-codecommit add .
$ git -C ~/environment/retail-store-sample-codecommit commit -am "commit ban đầu"
$ git -C ~/environment/retail-store-sample-codecommit push --set-upstream origin main
```

Bạn có thể truy cập `CodePipeline` trong Bảng điều khiển AWS và khám phá ống dẫn `eks-workshop-retail-store-sample`:

https://console.aws.amazon.com/codesuite/codepipeline/pipelines/eks-workshop-retail-store-sample/view

Nó sẽ trông giống như này:

![EKS](/images/0006/00036.png?featherlight=false&width=90pc)

Kết quả của một lần chạy CodePipeline với CodeBuild là bạn sẽ có một hình ảnh mới trong ECR

```bash
$ echo IMAGE_URI_UI=$IMAGE_URI_UI
```

Hậu tố `z7llv2` trong tên `retail-store-sample-ui-z7llv2` là ngẫu nhiên và sẽ khác nhau trong trường hợp của bạn.

![EKS](/images/0006/00037.png?featherlight=false&width=90pc)

Trong khi chúng ta đang chờ đợi ống dẫn để tạo ra các hình ảnh mới (5-10 phút), hãy [tự động hóa cập nhật hình ảnh vào Git](https://fluxcd.io/flux/guides/image-update/) bằng Flux Image Automation Controller.

Đầu tiên, chúng ta cần cài đặt các thành phần Flux.

```bash
$ flux install --components-extra=image-reflector-controller,image-automation-controller \
  --network-policy=false
```

Tiếp theo, chỉnh sửa tệp `deployment.yaml` và thêm giá trị đại diện cho URL hình ảnh container mới:

```bash
$ git -C ~/environment/flux pull
$ sed -i 's/^\(\s*\)image: "public.ecr.aws\/aws-containers\/retail-store-sample-ui:0.4.0"/\1image: "public.ecr.aws\/aws-containers\/retail-store-sample-ui:0.4.0" # {"$imagepolicy": "flux-system:ui"}/' ~/environment/flux/apps/ui/deployment.yaml
$ less ~/environment/flux/apps/ui/deployment.yaml | grep imagepolicy
          image: "public.ecr.aws/aws-containers/retail-store-sample-ui:0.4.0" # {"$imagepolicy": "flux-system:ui"}
```

Điều này sẽ thay đổi hình ảnh trong đặc tả pod thành:

```text
image: "public.ecr.aws/aws-containers/retail-store-sample-ui:0.4.0" `# {"$imagepolicy": "flux-system:ui"}`
```

Commit các thay đổi này vào triển khai:

```bash
$ git -C ~/environment/flux add .
$ git -C ~/environment/flux commit -am "Thêm ImagePolicy"
$ git -C ~/environment/flux push
$

 flux reconcile kustomization apps --with-source
```

Chúng ta cần triển khai các định nghĩa tài nguyên tùy chỉnh (ImageRepository, ImagePolicy, ImageUpdateAutomation) cho Flux để kích hoạt giám sát hình ảnh container mới trong ECR và triển khai tự động bằng cách sử dụng GitOps.

1. **ImageRepository**:

```file
manifests/modules/automation/gitops/flux/imagerepository.yaml
```

2. **ImagePolicy**:

```file
manifests/modules/automation/gitops/flux/imagepolicy.yaml
```

3. **ImageUpdateAutomation**:

```file
manifests/modules/automation/gitops/flux/imageupdateautomation.yaml
```

Thêm các tệp này vào kho ứng dụng và áp dụng chúng:

```bash
$ cp ~/environment/eks-workshop/modules/automation/gitops/flux/image*.yaml \
  ~/environment/retail-store-sample-codecommit/
$ yq -i ".spec.image = env(IMAGE_URI_UI)" \
  ~/environment/retail-store-sample-codecommit/imagerepository.yaml
$ less ~/environment/retail-store-sample-codecommit/imagerepository.yaml | grep image:
$ kubectl apply -f ~/environment/retail-store-sample-codecommit/imagerepository.yaml
$ kubectl apply -f ~/environment/retail-store-sample-codecommit/imagepolicy.yaml
$ kubectl apply -f ~/environment/retail-store-sample-codecommit/imageupdateautomation.yaml
```

Chúng ta đã tạo kiến trúc sau:

![EKS](/images/0006/00038.png?featherlight=false&width=90pc)

Bây giờ, hãy điều chỉnh các thay đổi.

```bash
$ flux reconcile image repository ui
$ flux reconcile kustomization apps --with-source
$ kubectl wait deployment -n ui ui --for condition=Available=True --timeout=120s
$ git -C ~/environment/flux pull
$ kubectl -n ui get pods
```

Chúng ta có thể kiểm tra rằng `image:` trong `deployment` đã được cập nhật đến một tag mới.

```bash
$ kubectl -n ui describe deployment ui | grep Image
```

Để truy cập `UI` bằng trình duyệt, chúng ta cần truy cập bằng cách tạo một tài nguyên Ingress với mẫu sau:

```file
manifests/modules/automation/gitops/flux/ci-ingress/ingress.yaml
```

Điều này sẽ khiến AWS Load Balancer Controller cung cấp một Application Load Balancer và cấu hình nó để định tuyến lưu lượng đến các Pods cho ứng dụng `ui`.

```bash timeout=180 wait=10
$ kubectl apply -k ~/environment/eks-workshop/modules/automation/gitops/flux/ci-ingress
```

Hãy kiểm tra đối tượng Ingress đã được tạo:

```bash
$ kubectl get ingress ui -n ui
NAME   CLASS   HOSTS   ADDRESS                                            PORTS   AGE
ui     alb     *       k8s-ui-ui-1268651632.us-west-2.elb.amazonaws.com   80      15s
```

Chúng ta chờ đợi 2-5 phút cho Application Load Balancer sẽ được cung cấp và kiểm tra trang UI bằng URL của ingress.

```bash timeout=300
$ export UI_URL=$(kubectl get ingress -n ui ui -o jsonpath="{.status.loadBalancer.ingress[*].hostname}{'\n'}")
$ wait-for-lb $UI_URL
```

![EKS](/images/0006/00039.png?featherlight=false&width=90pc)

Hãy giới thiệu các thay đổi vào mã nguồn của Ứng dụng Mẫu.

Chỉnh sửa tệp:

```bash
$ sed -i 's/\(^\s*<a class="navbar-brand" href="\/home">\)Retail Store Sample/\1Retail Store Sample New/' \
  ~/environment/retail-store-sample-codecommit/src/ui/src/main/resources/templates/fragments/layout.html
$ less ~/environment/retail-store-sample-codecommit/src/ui/src/main/resources/templates/fragments/layout.html | grep New
```

Thay đổi dòng 24

`<a class="navbar-brand" href="/home">Retail Store Sample</a>` thành `<a class="navbar-brand" href="/home">Retail Store Sample New</a>`

Commit các thay đổi.

```bash wait=30
$ git -C ~/environment/retail-store-sample-codecommit status
$ git -C ~/environment/retail-store-sample-codecommit add .
$ git -C ~/environment/retail-store-sample-codecommit commit -am "Update UI src"
$ git -C ~/environment/retail-store-sample-codecommit push
```

Chờ cho đến khi thực thi CodePipeline hoàn thành:

```bash timeout=900 wait=30
$ kubectl -n ui describe deployment ui | grep Image
$ while [[ "$(aws codepipeline get-pipeline-state --name ${EKS_CLUSTER_NAME}-retail-store-sample --query 'stageStates[1].actionStates[0].latestExecution.status' --output text)" != "InProgress" ]]; do echo "Đang chờ pipeline bắt đầu ..."; sleep 10; done && echo "Pipeline đã bắt đầu."
$ while [[ "$(aws codepipeline get-pipeline-state --name ${EKS_CLUSTER_NAME}-retail-store-sample --query 'stageStates[1].actionStates[2].latestExecution.status' --output text)" != "Succeeded" ]]; do echo "Đang chờ pipeline đạt trạng thái 'Succeeded' ..."; sleep 10; done && echo "Pipeline đã đạt trạng thái 'Succeeded'."
```

Sau đó, chúng ta có thể kích hoạt Flux để đảm bảo nó điều chỉnh hình ảnh mới:

```bash
$ flux reconcile image repository ui
$ sleep 5
$ flux reconcile kustomization apps --with-source
$ kubectl wait deployment -n ui ui --for condition=Available=True --timeout=120s
```

Nếu chúng ta kéo kho Git, chúng ta có thể thấy các thay đổi được thực hiện trong log:

```bash
$ git -C ~/environment/flux pull
$ git -C ~/environment/flux log
commit f03661ddb83f8251036e2cc3c8ca70fe32f2df6c (HEAD -> main, origin/main, origin/HEAD)
Author: fluxcdbot <fluxcdbot@users.noreply.github.com>
Date:   Fri Nov 3 17:18:08 2023 +0000

    1234567890.dkr.ecr.us-west-2.amazonaws.com/retail-store-sample-ui-c5nmqe:i20231103171720-ac8730

e8
[...]
```

Tương tự, chế độ xem commit của CodeCommit sẽ hiển thị hoạt động:

https://console.aws.amazon.com/codesuite/codecommit/repositories/eks-workshop-gitops/commits

Chúng ta cũng có thể kiểm tra các pod để xem hình ảnh đã được cập nhật:

```bash
$ kubectl -n ui get pods
$ kubectl -n ui describe deployment ui | grep Image
```

Sau khi xây dựng và triển khai thành công (5-10 phút), chúng ta sẽ có phiên bản mới của ứng dụng UI hoạt động.

![EKS](/images/0006/00040.png?featherlight=false&width=90pc)

Hãy cùng nhau tận hưởng phiên bản mới này! 🎉