---
title: "How to build & deploy a high-availability web application using docker ECS with EC2 on AWS."
description: "How to build & deploy a high-availability web application using docker ECS with EC2 on AWS."
dateString: October 2024
draft: false
tags: ["AWS","ECS"]
weight: 101
---
## Kiến trúc dự án
![images](/images/ecs/architecture.png)
-   Lâu lâu hands-on vài projects để nâng cao chuyên môn & kỹ năng trong việc sử dụng AWS. Và đây là project mà mình đã làm.
-   Topic: How to build & deploy a high-availability web application using docker ECS with EC2 on AWS.
-   Time to completed: 45 minutes
-   Region: Singapore
-   Cost: 1$
### Các bước chuẩn bị:
-   1 chiếc Laptop or PC (không cần mạnh) có thể truy cập vào AWS console.
-   IAM user full permission Administrator (không sử dụng User root khi làm lab).
-   Registry domain tại Route 53 khoảng 3$ ( mục đích sử dụng để access dns route 53 thay vì sử dụng dns alb) (optional).
    -   Tại đây mình đã registry domain: https://cloud-sangnguyen.click
### Các bước thực hiện:
**1.  VPC Setting (VPC, Route table, Public & Private Subnet, IGW, NGW)**
-   Click "Create VPC"
    ![vpc](/images/ecs/vpc.png)
-   **Resources to create:** VPC and more
-   **Name:** d-vpc-01
-   **IPv4 CIDR block:** 10.0.0.0/16
    ![vpc](/images/ecs/vpc-01.png)
-   **Number of Public subnets:** 2
-   **Number of Private subnets:** 2
    -   **Public subnet CIDR block in ap-southeast-1a:** 10.0.0.0/20
    -   **Public subnet CIDR block in ap-southeast-1b:** 10.0.32.0/20
    -   **Private subnet CIDR block in ap-southeast-1a:** 10.0.16.0/20
    -   **Private subnet CIDR block in ap-southeast-1b:** 10.0.48.0/20
  ![vpc](/images/ecs/vpc-02.png)
  ![vpc](/images/ecs/vpc-03.png)
- **Nat gateways:** 1 per AZ
- **VPC endpoints:** None
  ![vpc](/images/ecs/vpc-04.png)
- Preview lại một lần nuk => Click "Create VPC" (để hoàn tất việc triển khai)
  ![vpc](/images/ecs/vpc-05.png)
- Quá trình tạo VPC mất tầm 2 - 3 phút
  ![vpc](/images/ecs/vpc-06.png)
- Confirm VPC
  ![vpc](/images/ecs/vpc-07.png)
 {{% notice warning %}}
 Lưu ý: Việc triển khai 2 Nat gateways trên 2 AZ sẽ phát sinh thêm chi phí, cần cân nhắc khi làm việc này, hiện tại mình sử dụng credits trong qá trình làm lab.
  {{% /notice %}}
2.  **Launch EC2 instance (vai trò Bastion-host)**
-   Click "Launch instances"
    ![ec2](/images/ecs/ec2.png)
-   **Name:** d-ec2-bastion-host
    ![ec2](/images/ecs/ec2-01.png)
-   **AMI:** Amazon Linux 2023
    ![ec2](/images/ecs/ec2-02.png)
-   **Instance type:** t3.medium (mình sử dụng t3.medium để xào nấu docker cho nó mượt) (sẽ phát sinh thêm chi phí)
    ![ec2](/images/ecs/ec2-03.png)
-   **Keypair:** keypair.singapore.pem
    ![ec2](/images/ecs/ec2-04.png)
-   **Network setting:**
    -   **VPC - required:** d-vpc-01 (đã khởi tạo từ đầu)
    -   **Subnet:** Public Subnet
    -   **Auto-assign public IP:** Enable
    -   **Security groups:** click create security group
    -   **Security group name:** d-sg-bastion-host
    -   **Description:** No open port 22 (Vì mình sẽ connect thông qua Session manager)
    -   **Inbound rule:** none
        ![ec2](/images/ecs/ec2-05.png)
    -   Preview => click "Launch instance"
        ![ec2](/images/ecs/ec2-06.png)
3.  **Create Repository ECR**
#### Click "Create a repository"
-   **Repository name:** d-ecr-web-application
-   **Image tag mutability:** Mutable
    ![ecr](/images/ecs/ecr-01.png)
-   **Encryption settings:** AES-256
    ![ecr](/images/ecs/ecr-02.png)
-   Confirm repository
    ![ecr](/images/ecs/ecr-03.png)

####  Tiếp đến quay lại Bastion-host (để xào nấu docker)
-   Connect Bastion-host thông qua Session manager
    ![ec2](/images/ecs/ec2-10.png)
    ![ec2](/images/ecs/ec2-11.png)
- Command line:
  {{< highlight go >}}
   Yum update -y
   Yum install git -y
   Yum install docker -y
   systemctl start docker
   git --version
   docker --version
  {{< /highlight >}}
- Check
  ![ec2](/images/ecs/ec2-12.png)
  - Git clone sources html từ github (vì mình không chuyên sâu về dev nên mình download sources free ở trên web)
  - ls -la (list danh mục hiện tại)
  - cd **Host-a-Static-Web-App-on-AWS-with-Docker-and-AWS-ECS**
  ![ec2](/images/ecs/ec2-13.png)
  - ls -lart
  ![ec2](/images/ecs/ec2-14.png)
  - vi Dockerfile
   {{< highlight go >}}
    - FROM nginx:1.25.3
    - COPY ./ /usr/share/nginx/html
    - EXPOSE 80
    - CMD ["nginx", "-g", "daemon off;"]
  {{< /highlight >}}
  - docker build -t web-application .
  - docker images (check images)
  - docker run -d --name web-application -p 8080:80 web-application
  ![ec2](/images/ecs/ec2-16.png)
  - docker ps -a (check status container)
  ![ec2](/images/ecs/ec2-17.png)
  - Edit security group for Bastion-host
  ![ec2](/images/ecs/ec2-18.png)
  - Inbound rules:
    - **Type:** custom TCP
    - **Port range:** 8080
    - **Source:** Anywhere IPv4
  ![ec2](/images/ecs/ec2-19.png)
  ![ec2](/images/ecs/ec2-20.png)
  ![ec2](/images/ecs/ec2-21.png)
  * Copy IP Public:8080 (truy cập vào trình duyệt)
  ![ec2](/images/ecs/ec2-22.png)
  ![ec2](/images/ecs/ec2-23.png)
#### Modify Iam Role for EC2 (permission allows EC2 access to ECR)
  ![iam-role](/images/ecs/modify-iam-role.png)
  ![iam-role](/images/ecs/modify-iam-role-1.png)
#### Quay lại EC2 Bastion-host (để push image lên ECR)
-   View push command trong ECR thực hiện thứ tự dòng lệnh
  ![ecr](/images/ecs/ecr-04.png)
  ![ecr](/images/ecs/ecr-05.png)
  ![ecr](/images/ecs/ecr-06.png)
-   Check image bên trong ECR
  ![ecr](/images/ecs/ecr-07.png)
4.  **Create cluster in ECS**
-   Click create cluster
    ![ecs](/images/ecs/ecs.png)
    -   **Cluster name:** d-ecs-cluster-web-application
        ![ecs](/images/ecs/ecs-01.png)
-   **Infrastructure: Amazon EC2 instances**
    -   **Auto Scaling group (ASG):** Create new ASG
    -   **Provisioning model:** On-demand
    -   **Container instance Amazon Machine Image (AMI):** Amazon Linux 2023
    -   **EC2 instance type:** t3.medium
    -   **EC2 instance role:** ecsInstanceRole
    -   **Desired capacity:**
        -   **Minimum:** 4 (scale out theo kiến trúc)
        -   **Maximum:** 4
    - **SSH Key pair:** Keypair-singapore
    - **Root EBS volume size:** 30
      ![ecs](/images/ecs/ecs-02.png)
-   **Network settings for Amazon EC2 instances**
    -   **VPC:** d-vpc-01
    -   **Subnets:** Specify 2 private subnets
      ![ecs](/images/ecs/ecs-03.png)
    -   **Security group:** Create a new security group
        -   **Security group name:** d-sg-ecs-web-application
        -   **Security group description:** Allows traffic from ALB & Bastion-host
    - **Inbound rules for security groups:**
      - **Type:** SSH
      - **Protocol:** TCP
      - **Port range:** 22
      - **Source:** Choose "d-sg-bastion-host"
    ![ecs](/images/ecs/ecs-04.png)
  - Create
    ![ecs](/images/ecs/ecs-05.png)
*   Lúc này nó sẽ tự động scale out 4 instances
    ![ecs](/images/ecs/ecs-06.png)
####  Create Task definition
- Click "Create task definition"
  ![task](/images/ecs/task.png)
#### Task definition configuration
- **Task definition family:** d-task-definition-web-application
  ![task](/images/ecs/task-01.png)
#### Infrastructure requirements
  - **Launch type:** Amazon EC2 instances
  - **Operating system/Architecture:** Linux/X86_64
  - **Network mode:** bridge
  - **Task size**
    - **CPU:** 1vCPU
    - **Memory:** 3GB
  - **Task role:** none (tùy bạn có muốn task bên trong container call API AWS services khác không, nếu không thì để trống)
  ![task](/images/ecs/task-02.png)
#### Container - 1
  - **Name:** Container-web-application
  - **Image URI:** copy Image URI tại ECR
  - **Essential container:** Yes
  ![task](/images/ecs/task-03.png)
  ![task](/images/ecs/task-04.png)
  - Create
  ![task](/images/ecs/task-05.png)
5.  **Create Load Balancers**
####  Create target group
- Create target group
  ![target group](/images/ecs/tg.png)
- **Choose a target type:** Instances
  ![target group](/images/ecs/tg-01.png)
- **Target group name:** d-tg-web-application
- **Protocol:** HTTP
- **Port:** 80
- **IP address type:** IPv4
- **VPC:** d-vpc-01
- **Protocol version:** HTTP1
  ![target group](/images/ecs/tg-02.png)
- Next
  ![target group](/images/ecs/tg-03.png)
- Register targets (tại đây bạn không cần đăng ký EC2 instance vào trong target group, khi kết nối ALB nó sẽ tự động đăng ký vào target)
  ![target group](/images/ecs/tg-04.png)
  ![target group](/images/ecs/tg-05.png)
####  Create Security group ALB
  ![security group](/images/ecs/alb-04.png)
- **Security group name:** d-sg-alb-web-application
- **Description:** Allows traffic from Internet
- **VPC:** d-vpc-01
- **Inbound rules:**
  - **Type:** HTTP
  - **Protocol:** TCP
  - **Port range:** 80
  - **Sources:** Anywhere IPv4
- Create security group (hoàn tất việc tạo)
  ![security group](/images/ecs/alb-05.png)
#### Create Load balancer
- **Load balancer types:** Application Load Balancer => Create
  ![Alb](/images/ecs/alb-01.png)
- **Load balancer name:** d-alb-web-application
- **Scheme:** Internet-facing
- **Load balancer IP address type:** IPv4
  ![Alb](/images/ecs/alb-02.png)
##### Network mapping
- **VPC:** d-vpc-01
- **Availability Zones:** Specify 2 AZ có 2 Public Subnets
  ![Alb](/images/ecs/alb-03.png)
- **Security groups:** d-sg-alb-web-application
  ![Alb](/images/ecs/alb-06.png)
#####  Listeners and routing
- **Protocol:** HTTP
- **Port:** 80
- **Forward to:** d-tg-web-application
  ![Alb](/images/ecs/alb-07.png)
- Create load balancer (hoàn tất việc tạo)
  ![Alb](/images/ecs/alb-08.png)
##### Edit security group của d-sg-ecs-web-application
  ![security group](/images/ecs/edit-sg-ecs.png)
- **Inbound rules => Add rule**
  - **Type:** All TCP
  - **Protocol:** TCP
  - **Port range:** 0 - 65535
  - **Sources:** d-sg-alb-web-application (chọn security của ALB)
  ![security group](/images/ecs/edit-sg-ecs-1.png)
6.  **Create services in cluster**
- Create services
  ![ecs](/images/ecs/ecs-10.png)
#### Environment
##### Compute configuration (advanced)
- **Compute options:** Launch type
- **Launch type:** EC2
  ![ecs](/images/ecs/ecs-11.png)
##### Deployment configuration
- **Application type:** Click Service
- **Family:** d-task-definition-web-application
- **Revision:** 1 (LATEST)
- **Service name:** d-ecs-service-web-application
- **Service type:** Replica
- **Desired tasks:** 4
  ![ecs](/images/ecs/ecs-12.png)
##### Load balancing (optional)
  ![ecs](/images/ecs/ecs-13.png)
- **Load balancer type:** Application Load Balancer
- **Load balancer:** d-alb-web-application
- **Listener:** Use an existing listener
- **Listener:** 80:HTTP
  ![ecs](/images/ecs/ecs-14.png)
- **Target group:** Use an existing target group
- **Target group name:** d-tg-web-application
  ![ecs](/images/ecs/ecs-15.png)
- Create (hoàn tất việc tạo service)
  ![ecs](/images/ecs/ecs-16.png)
- Quá trình task running mất tầm 1 - 2 phút
  ![ecs](/images/ecs/ecs-17.png)
  ![ecs](/images/ecs/ecs-18.png)
- Truy cập dns alb để kiểm tra
  ![ecs](/images/ecs/access-complete.png)
7. **Cấu hình SSL/TLS mã hóa đường truyền truy cập thông qua giao thức HTTPS**
- Tại giao diện ALB
- Add Listener
  ![https](/images/ecs/https.png)
#### Listener configuration
- **Protocol:** HTTPS
- **Port:** 443
- **Forward to target group:** d-tg-web-application
  ![https](/images/ecs/https-01.png)
#### Secure listener settings:
##### Default SSL/TLS server certificate
- **Certificate source:** From ACM
- **Certificate (from ACM):** cloud-sangnguyen.click
  ![https](/images/ecs/https-02.png)
- **Edit inbound rules:** d-sg-alb-web-application (open port 443)
  ![https](/images/ecs/https-03.png)
- **Add inbound rule:**
  - **Type:** HTTPS
  - **Protocol:** TCP
  - **Port range:** 443
  - **Source:** Anywhere IPv4
- Save rules
  ![https](/images/ecs/https-04.png)
###  Sử dụng DNS đã đăng ký tại route 53 thay thế cho DNS ALB (optional)
- Route 53 trên console
- Create record
  ![https](/images/ecs/https-05.png)
- **Routing policy:** Simple routing
  ![https](/images/ecs/https-06.png)
- **Configure records:** define simple record
  ![https](/images/ecs/https-7.png)
#### Value/Route traffic to
  - Alias to Application and Classic Load Balancer
  - Asia Pacific (Singapore)
  - Click "Define simple record"
  ![https](/images/ecs/https-08.png)
- Create records (để hoàn tất việc tạo)
  ![https](/images/ecs/https-09.png)
- Confirm record
  ![https](/images/ecs/https-10.png)
- Truy cập https://cloud-sangnguyen.click (test hoạt động)
  ![https](/images/ecs/https-complete.png)
### Clean up resources
1.  Delete Services in cluster
2.  Delete Cluster in ECS
3.  Delete Task-definition
4.  Delete Application Load Balancer
5.  Delete Target group
6.  Terminate EC2-Bastion-host
7.  Delete Nat gateways
8.  Release elastic IP address
9.  Delete VPC

{{% notice error %}}
Kiểm tra kỹ lại all resources một lần nuk, để tránh mất tiền :)
{{% /notice %}}
- Ok, vừa rồi là một project khá dài cũng khá nhiều công đoạn trong việc triển khai một ứng dụng high-availability sử dụng docker ECS with EC2.
- Chúc các bạn thành công.





