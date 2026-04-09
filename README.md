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

1. In order to give the public subnets in our VPC internet access we will have to create and attach an Internet Gateway. Click on Internet Gateway on the Dashboard.


2. Create your internet gateway by simply giving it a name and clicking Create internet gateway.


3. After creating the internet gateway, attach it to your VPC. You have a couple options on how to do this, either with the creation success message or the Actions drop down.


<img width="1318" height="489" alt="Internet Gateway" src="https://github.com/user-attachments/assets/204d20c6-3e7b-4cd3-ad27-6f8c67fca8e0" />


<img width="1381" height="543" alt="IG" src="https://github.com/user-attachments/assets/d8498257-b3cf-48a6-ad3d-aa85bd076072" />


4. Then, select the correct VPC and click Attach internet gateway.

**NAT Gateway**

1. In order for our instances in the app layer private subnet to be able to access the internet they will need to go through a NAT Gateway. For high availability, you’ll deploy one NAT gateway in each of your public subnets. Navigate to NAT Gateways on the left side of the current dashboard and click Create NAT Gateway.

2. Fill in the Name, choose one of the public subnets you have created, and then allocate an Elastic IP. Click Create NAT gateway.



<img width="1390" height="544" alt="Nat Gateway" src="https://github.com/user-attachments/assets/3437ed4b-3f59-4129-b5a6-526b33101213" />


<img width="1352" height="632" alt="NAT" src="https://github.com/user-attachments/assets/c409aa30-46f1-4cae-ad6e-ce960c24d69d" />

**Routing Configuration**

Navigate to Route Tables on the left side of the VPC dashboard and click Create route table First, let’s create one route table for web layer public subnets, database, app,and web private database for all the regions and name it accordingly.


<img width="1119" height="509" alt="Route Tables" src="https://github.com/user-attachments/assets/1551ff73-be26-4269-8492-b285ec45834d" />

**Security Groups**

Security groups will tighten the rules around which traffic will be allowed to our application Load Balancers and EC2 instances. Navigate to Security Groups on the left side of the VPC dashboard, under Security.

The first security group you’ll create is for the frontend application load balancer, frontend server, backend application load balancer, backend server and the database. After typing a name and description, add an inbound rule to allow HTTP, SSH at both frontend server and backend server. The source of each security group adhere to the architectural diagram as seen below.

**Frontend Aplication load Balancer Security Group**

<img width="1377" height="556" alt="Frontend-alb-SG" src="https://github.com/user-attachments/assets/e11fa36e-e712-45a8-85d3-96b41dcd795f" />



**Frontend Server Security Group**


<img width="1359" height="579" alt="Frontend-server-SG" src="https://github.com/user-attachments/assets/1118a366-9d45-4b4d-a4df-6f23d9207a1e" />

**Backend Aplication load Balancer Security Group**



<img width="1379" height="596" alt="Backend-alb-SG" src="https://github.com/user-attachments/assets/d5d2345b-35e2-47b8-85d5-cbca36b5ffdd" />


**Backtend Server Security Group**


<img width="1354" height="576" alt="Backend sserver-SG" src="https://github.com/user-attachments/assets/df03310d-5c79-4b18-84dc-8e89442afd56" />

**Database Security Group**


<img width="1377" height="555" alt="Database-SG" src="https://github.com/user-attachments/assets/2ba6f9d3-ceb9-446a-88f3-edef70f8ae39" />


**Database Deployment - Part 2**

* **Deploy Database Layer**

  * Subnet Groups
  * Multi-AZ Database
  * 
**Subnet Groups**

Navigate to the RDS dashboard in the AWS console and click on Subnet groups on the left hand side. Click Create DB subnet group.
Give your subnet group a name, description, choose the Demo VPC and all the 3 subnets we created.


<img width="1341" height="356" alt="DB-SUB-GP" src="https://github.com/user-attachments/assets/5dd7c312-81f7-4fb3-ba0b-7ffb104488b4" />


When adding subnets, make sure to add the subnets we created in each availability zone specificaly for our database layer. You may have to navigate back to the VPC dashboard and check to make sure you're selecting the correct subnet IDs.


<img width="1359" height="569" alt="DB-subnet-group" src="https://github.com/user-attachments/assets/9b147eac-d475-45e2-9fc0-
 6dbcce65f2c4" />

 **Multi-AZ Database Deployment**
 
1. Navigate to Databases on the left hand side of the RDS dashboard and click Create database.

2. We'll now go through several configuration steps. Start with a Standard create for this MySQL-Compatible Amazon Aurora database. Leave the rest of the defaults in the Engine options as default.


<img width="1313" height="553" alt="image" src="https://github.com/user-attachments/assets/16074589-d166-42e8-8e01-dd6d1b9b6bf8" />


Under the Templates section choose Dev/Test since this isn't being used for production at the moment. Under Settings set a username and password of your choice and note them down since we'll be using password authentication to access our database.

<img width="1361" height="547" alt="image" src="https://github.com/user-attachments/assets/5c5d0f15-323c-49d0-9a80-de84dec0b7f2" />


Next, under instance configuration, chode burstable class and instance typr db.t3.micro. Under Connectivity, set the VPC, choose the subnet group we created earlier, and select no for public access.

<img width="1327" height="592" alt="image" src="https://github.com/user-attachments/assets/5d8aa804-7a22-4f89-9e89-57c0ed3e6f5c" />


Next, under instance configuration, chode burstable class and instance typr db.t3.micro. Under Connectivity, set the VPC, choose the subnet group we created earlier, and select no for public access.



<img width="1335" height="554" alt="image" src="https://github.com/user-attachments/assets/28bc1bf9-a30c-4776-9dc1-7e939b1663a2" />


When your database is provisioned, you should see a reader and writer instance in the database subnets of each availability zone.

<img width="1347" height="550" alt="image" src="https://github.com/user-attachments/assets/bb67edeb-19a2-4409-8c25-ef807d4430e9" />

<img width="1382" height="334" alt="Database" src="https://github.com/user-attachments/assets/ec9b42d8-5528-4613-b68a-22196b1af5f8" />


**Instance Deployment - Part 3**

Navigate to the EC2 service dashboard and click on Instances on the left hand side. Then, click Launch Instances.

Select the first Amazon Linux 2 AMI

We'll be using the free tier eligible T.3 micro instance type. Select that and click Next: Configure Instance Details.

When configuring the instance details, make sure to select to correct VPC, subnet, and security group we created. we are creating both frontend server and backend server, so use one of the public subnets we created for this layer.

<img width="924" height="543" alt="image" src="https://github.com/user-attachments/assets/ab0e57e1-0f48-424a-be2a-61789f614adf" />

Create new key and enable public IP

<img width="1359" height="540" alt="image" src="https://github.com/user-attachments/assets/91669c96-1d41-4293-a957-c79fabadb268" />


Configure the volume and launch the instance


<img width="1319" height="508" alt="image" src="https://github.com/user-attachments/assets/6ad44fbf-3829-415d-af77-77c93c021eb7" />

Follow the same procedure to configure backend server


<img width="1343" height="373" alt="image" src="https://github.com/user-attachments/assets/4ca88aa9-a127-4d17-9992-26f065bc1ea4" />

**Connect to Instance**

Navigate to your list of running EC2 Instances by clicking on Instances on the left hand side of the EC2 dashboard. When the instance state is running, connect to your instance by clicking the checkmark box to the left of the instance, and click the connect button on the top right corner of the dashboard.Select the Session Manager tab, and click connect. This will open a new browser tab for you.

<img width="1327" height="506" alt="image" src="https://github.com/user-attachments/assets/f8e927d1-6e6c-40b2-a1be-412158a25e4c" />

After connecting to the Backend server........


<img width="1024" height="659" alt="image" src="https://github.com/user-attachments/assets/8783dc98-e0dd-424b-9c90-46aa889ac837" />

**Configure Database**

 1. Start by downloading the MYSQL client for linux 2023:

   ```bash
   sudo dnf install mariadb105 -y                
```

2. Initiate your DB connection with your MYSQL RDS endpoint url. In the following command, replace the RDS writer endpoint and the username, and then execute it in the browser terminal:

 ```bash
 mysql -h <db-endpoint> -P 3306 -u <username> -p     
```

<img width="1289" height="676" alt="image" src="https://github.com/user-attachments/assets/71f9bb1d-4d74-4c62-8d47-539eb7c11a90" />



NOTE: If you cannot reach your database, check your credentials and security groups.


You will then be prompted to type in your password. Once you input the password and hit enter, you should now be connected to your database.

3. Create a database called webappdb with the following command using the MySQL CLI:

```bash
CREATE DATABASE webappdb;
``` 


You can verify that it was created correctly with the following command:

```bash
SHOW DATABASES;
``` 

<img width="1329" height="680" alt="image" src="https://github.com/user-attachments/assets/2adc3d46-98b9-4517-9fd0-4c98a408e3d1" />


4. Create a data table by first navigating to the database we just created:
   
```bash
USE webappdb;
```


Then, create the following transactions table by executing this create table command:

```bash
CREATE TABLE IF NOT EXISTS transactions(id INT NOT NULL AUTO_INCREMENT, amount DECIMAL(10,2), description VARCHAR(100), PRIMARY KEY(id));
```


Verify the table was created:

```bash
SHOW TABLES;
```

<img width="1345" height="671" alt="image" src="https://github.com/user-attachments/assets/e77ade6c-5ba0-435b-909c-8a400d2991aa" />


Insert data into table for use/testing later:

```bash
INSERT INTO transactions (amount,description) VALUES ('400','groceries');
```


Verify that your data was added by executing the following command:

```bash
SELECT * FROM trasanctions;
```


When finished, just type exit and hit enter to exit the MySQL client.

<img width="1252" height="688" alt="image" src="https://github.com/user-attachments/assets/b96b8509-f062-4b42-a63a-b8e603e09812" />


Create AMI from both frontend server and backend server

<img width="1363" height="515" alt="image" src="https://github.com/user-attachments/assets/cd273f32-19c9-4798-bad8-bfc82e61e6f1" />

After creating both images, go to ec2, then AMI

<img width="1365" height="470" alt="image" src="https://github.com/user-attachments/assets/d5e3ca94-fc13-4a27-a386-d3fb173edb61" />


Now, we need to create launch template from both AMI'S


<img width="1342" height="537" alt="image" src="https://github.com/user-attachments/assets/3729424c-e992-4732-84ab-c0b45c41ca55" />


Now, select My AMI'S and the associated image for frontend server AMI'S 

<img width="1354" height="544" alt="image" src="https://github.com/user-attachments/assets/fc8dd99e-19f6-457b-ad9b-be92d7f861a9" />

Select the corresponding existing security group

<img width="1383" height="535" alt="image" src="https://github.com/user-attachments/assets/63eb3114-7449-45c3-b578-fe5b672e97a0" />


Add the user scrip in the documentation and replace the url with dns of backend application load balancer

<img width="1351" height="547" alt="image" src="https://github.com/user-attachments/assets/79a36d8b-4524-4198-9c12-2b0593d8d063" />


Copy the DNS of backend-alb, replace and launch template

<img width="1360" height="343" alt="image" src="https://github.com/user-attachments/assets/9ff4a426-440d-4d5f-9925-c61c092f96c0" />

Follow the same stpe to create launch template backend server AMI'S and replace the host with your RDS endpoint url. similarly, insert your username and password as seen blow


<img width="1347" height="500" alt="image" src="https://github.com/user-attachments/assets/5e6f031b-f025-4784-9200-91e90834981b" />


<img width="1340" height="501" alt="image" src="https://github.com/user-attachments/assets/a3df9d57-03a4-423a-9045-4390a283c946" />


