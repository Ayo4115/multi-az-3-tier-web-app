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


