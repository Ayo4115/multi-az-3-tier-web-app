 # ***multi-az-3-tier-web-app*** #
Scalable 3-tier cloud infrastructure on AWS. Features isolated networking layers, auto-scaling web/app tiers, and high-availability database clustering.



## **Architecture Overview** ##

<img width="3519" height="2684" alt="aws-three-tier-architecture" src="https://github.com/user-attachments/assets/c3a4399b-c564-4e74-a5fe-4492345da8df" />





The infrastructure is designed to be fault-tolerant by spreading resources across three Availability Zones (AZ-A, AZ-B, and AZ-C) within a single VPC.
1. **Networking Layer**
   
**VPC:** 10.0.0.0/16 CIDR block.
**Public Subnets:** Hosts the external Application Load Balancer (ALB) to handle incoming internet traffic.
**Private Subnets:** Used for the application and database tiers to ensure they are not directly accessible from the public internet.

2. **Tier Breakdown**
**Web Tier (Frontend):**

frontend-alb: Public-facing Load Balancer in Availability Zone B.
frontend-server: EC2 instances (typically in an Auto Scaling Group) across all three AZs.
**App Tier (Backend):**

backend-alb: Internal Load Balancer to route traffic from the Web tier to the App tier.
backend-server: Private application servers processing business logic.

**Database Tier:**

**Multi-AZ RDS:** A primary database in AZ-A with a synchronous standby replica in AZ-B for automated failover and high availability.

📊 **Subnet Mapping**


| Subnet Type | AZ-A | AZ-B | AZ-C |
| :--- | :--- | :--- | :--- |
| **Public Web** | `10.0.0.0/20` | `10.0.16.0/20` | `10.0.32.0/20` |
| **Private Web** | `10.0.48.0/20` | `10.0.64.0/20` | `10.0.80.0/20` |
| **Private App** | `10.0.96.0/20` | `10.0.112.0/20` | `10.0.128.0/20` |
| **Private DB** | `10.0.144.0/20` | `10.0.160.0/20` | `10.0.176.0/20` |

# **Networking and Security - Part 1** #

Now we will be building out the VPC networking components as well as security groups that will add a layer of protection around our EC2 instances, Aurora databases, and Elastic Load Balancers.

Create an isolated network with the following components:

* VPC

* Subnets

* Route Tables

* Internet Gateway

* NAT gateway

* Security Groups

**VPC and Subnets**

**VPC Creation**

Navigate to the VPC dashboard in the AWS console and navigate to Your VPCs on the left hand side.

Make sure VPC only is selected, and fill out the VPC Settings with a Name tag and a CIDR range of your choice.

NOTE: Make sure you pay attention to the subnet mapping and the region you’re deploying all your resources in. 

<img width="1354" height="551" alt="Demo-VPC" src="https://github.com/user-attachments/assets/48225b63-fee4-48cf-bff6-688b34218966" />


<img width="1081" height="559" alt="12 Subnets" src="https://github.com/user-attachments/assets/8f22bd06-e53c-40c8-874d-793915c156b0" />


<img width="1111" height="552" alt="12 subnets- cont" src="https://github.com/user-attachments/assets/ac70c85f-cdac-4b6f-8378-7cfd43c61c55" />



<img width="1119" height="509" alt="Route Tables" src="https://github.com/user-attachments/assets/75f3a3e3-3456-40c2-b641-1dd379787b37" />

<img width="1375" height="586" alt="web-publicRT" src="https://github.com/user-attachments/assets/9916cbac-9992-4463-8568-5c97c84bb393" />

<img width="1365" height="540" alt="Web-public-RT-edit" src="https://github.com/user-attachments/assets/ac3e4fcd-9eea-40cc-a481-871e590c82f4" />

<img width="1383" height="549" alt="DB-private-RT" src="https://github.com/user-attachments/assets/16151e5d-25ff-414a-b396-e4ad28651ced" />

<img width="1360" height="414" alt="DB-Private-RT-edit" src="https://github.com/user-attachments/assets/99a0a762-b49d-451c-874c-8727e3d6f56d" />


<img width="1368" height="551" alt="App=private-RT" src="https://github.com/user-attachments/assets/f89413d9-bacb-4519-aba8-58bf273cd1e5" />


<img width="1365" height="485" alt="App=private-RT-edit" src="https://github.com/user-attachments/assets/76703bda-2b95-4341-81d3-af107c62cc83" />


**Internet Connectivity**

**Internet Gateway**

In order to give the public subnets in our VPC internet access we will have to create and attach an Internet Gateway. Click on Internet Gateway on the Dashboard.


Create your internet gateway by simply giving it a name and clicking Create internet gateway.


After creating the internet gateway, attach it to your VPC. You have a couple options on how to do this, either with the creation success message or the Actions drop down.


<img width="1318" height="489" alt="Internet Gateway" src="https://github.com/user-attachments/assets/204d20c6-3e7b-4cd3-ad27-6f8c67fca8e0" />


<img width="1381" height="543" alt="IG" src="https://github.com/user-attachments/assets/d8498257-b3cf-48a6-ad3d-aa85bd076072" />



