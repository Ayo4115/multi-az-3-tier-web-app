# ***multi-az-3-tier-web-app***
Scalable 3-tier cloud infrastructure on AWS. Features isolated networking layers, auto-scaling web/app tiers, and high-availability database clustering.



**Architecture Overview**

<img width="3519" height="2684" alt="aws-three-tier-architecture" src="https://github.com/user-attachments/assets/c3a4399b-c564-4e74-a5fe-4492345da8df" />




The infrastructure is designed to be fault-tolerant by spreading resources across three Availability Zones (AZ-A, AZ-B, and AZ-C) within a single VPC.
1. Networking Layer
   
VPC: 10.0.0.0/16 CIDR block.
Public Subnets: Hosts the external Application Load Balancer (ALB) to handle incoming internet traffic.
Private Subnets: Used for the application and database tiers to ensure they are not directly accessible from the public internet.

2. Tier Breakdown
Web Tier (Frontend):

frontend-alb: Public-facing Load Balancer in Availability Zone B.
frontend-server: EC2 instances (typically in an Auto Scaling Group) across all three AZs.
App Tier (Backend):

backend-alb: Internal Load Balancer to route traffic from the Web tier to the App tier.
backend-server: Private application servers processing business logic.

Database Tier:

Multi-AZ RDS: A primary database in AZ-A with a synchronous standby replica in AZ-B for automated failover and high availability.
