---
title: "Build & deploy a high-availability web application using CloudFormation AWS."
description: "Build & deploy a high-availability web application using CloudFormation AWS."
dateString: October 2024
draft: false
tags: ["AWS","CloudFormation"]
weight: 101
---
## Kiến trúc dự án
![Images](/images/cfn/aws-cfn.png)
### CloudFormation là gì?
-   CloudFormation là một dịch vụ Infrastructure as Code được phát triển bởi Amazon Web Service, nó cho phép chúng ta define all resouces và tự động triển khai trên AWS bằng cách thông qua giao diện dòng lệnh. Việc sử dụng CloudFormation mang lại nhiều lợi ích ví dụ như:
    -   Triển khai được trên nhiều môi trường. 
    -   Tối ưu hóa thời gian.
    -   Nhất quán và dễ dàng triển khai.
    -   Tái sử dụng templates.
- CloudFormation sử dụng 2 loại type: yaml or json ( trong dự án này mình sử dụng loại type yaml vì nó dễ nhìn).
- Ngoài ra, còn rất nhiều các chức năng trong CloudFormation mọi người có thể view tại đây:
  
  https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html
### Giới thiệu về dự án:
-   Bạn đang muốn triển khai một ứng dụng web trên AWS, tuy nhiên bạn không muốn triển khai theo thủ công. Vì bạn muốn có thể tái sử dụng lại những resources đó vào những dự án cá nhân khác. Bạn sẽ sử dụng IaC để giúp bạn triển khai ứng dụng này.
#### CIDR Network
-   CIDR VPC: 172.16.0.0/24
-   CIDR Public Subnet 1: 172.16.0.0/26
-   CIDR Public Subnet 2: 172.16.0.64/26
-   CIDR Private Subnet 1: 172.16.0.128/26
-   CIDR Private Subnet 2: 172.16.0.192/26
### Yêu cầu dự án:
#### Network-Infra
-   Tạo VPC 
-   Tạo 4 subnets trên 2 Zone
-   Tạo Route table cho mỗi subnets
-   Tạo Internet Gateway
-   Tạo Nat Gateway (để các resources trong private subnet có thể truy cập Internet)
#### Server-Infra
-   Tạo Bastion-host ở public Subnet (để access tới các server trong private subnet)
-   Tạo Security Group cho all các objects: Bastion-host, ALB, and Prod
-   Tạo Launch-Template để sử dụng khi triển khai ASG
-   Tạo Application Load Balancer để route traffic đến các instances
-   Tạo Auto Scaling Group để scale out/in instances.
-   Tạo Certificate manager để cấp SSL/TLS mã hóa trên đường truyền
-   Tạo Iam Role cho instance có quyền read/write đến S3
### Điều kiện hoàn thành:
-   Ứng dụng có tính khả dụng cao
-   Application Load Balancer cung cấp dns https://dns-alb.com cho các client truy cập vào.
-   Scale out các instances như theo kiến trúc
-   Instances trong private subnet có thể read/write đến S3.
-   Nat Gateway cho các instances trong private subnet
### Các bước thực hiện:
-   Sử dụng Visual Studio Code để code 2 File: Network-infra.yaml and Server-infra.yaml
-   File 1: Network-infra.yaml
-   File 2: Server-infra.yaml
-   Mọi người có thể tham khảo source code của mình tại đây: 

https://github.com/hs-nguyen/Build-deploy-a-high-availability-web-application-using-CloudFormation-AWS

#### Tại Console AWS: Click CloudFormation.
-   Click CloudFormation
![Images](/images/cfn/cfn.png)
-   Click Create Stack
![Images](/images/cfn/create-stack.png)
-   Upload file Network-infra.yaml => Next
![Images](/images/cfn/up-load-file-network.png)
-   Stack name: Demo-Network-Infra
![Images](/images/cfn/stack-name.png)
![Images](/images/cfn/parameters.png)
-   Cấu hình stack (options) => Next
![Images](/images/cfn/configure-stack.png)
![Images](/images/cfn/configure-stack-1.png)
-   Review and create
![Images](/images/cfn/review-stack.png)
![Images](/images/cfn/review-stack-1.png)
-   Click Submit
![Images](/images/cfn/submit.png)

- Quá trình triển khai hạ tầng network sẽ mất từ 2 -> 3 phút.
![Images](/images/cfn/submit-network.png)
- Tiếp đến Create Stack: Server-infra.yaml các bước thực hiện giống như tạo stack Network-infra
- Quá trình triển khai hạ tầng Server mất từ 1 - 2 phút.
![Images](/images/cfn/submit-server.png)
- Check & Confirm lại configure
- Truy cập dns name: https://d-alb-prod-973693012.us-east-1.elb.amazonaws.com/
- Có thể sử dụng Route 53 để registry domain thay thế DNS của Load Balancer.
- Tiếp đến access vào EC2 Instance trong Private subnet list s3 bucket.
- Sử dụng Bastion-host EC2 làm trung gian access đến instances trong Private Subnet.
![Images](/images/cfn/bastion-host.png)
- Tạo Keypair file => copy key vào bên trong file
![Images](/images/cfn/create-keypair-file.png)
![Images](/images/cfn/cp-keypair.png)
- Sau đó SSH vào các EC2 Instances trong Private Subnet
![Images](/images/cfn/ssh-private-instance.png)
- List s3 bucket 
- **CLI**: aws s3 ls
![Images](/images/cfn/check-list-s3.png)

Okay, vậy là mình đã triển khai thành công ứng dụng web đảm bảo tính khả dụng cao thông qua giao diện dòng lệnh. Đây là project cũng không có gì quá cao siêu nhưng nó cũng là thành quả thời gian mà mình bỏ ra, hy vọng project này có thể giúp các bạn một phần nào trong hành trình lên mây của mình.

Chúc các bạn thành công! 
