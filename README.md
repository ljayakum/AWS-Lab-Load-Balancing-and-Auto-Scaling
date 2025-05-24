# AWS-Lab-Load-Balancing-and-Auto-Scaling
I’m working on a lab that walks through how to use **Elastic Load Balancing (ELB)** and **Auto Scaling** on AWS. I’m also taking notes on the side as I go.

---

## What this lab covers

This lab is about making your AWS infrastructure more reliable and scalable using two main services:

- **Elastic Load Balancing (ELB):** Distributes incoming traffic across multiple EC2 instances automatically. It helps make your application fault-tolerant and can handle changes in traffic smoothly.  
- **Auto Scaling:** Adjusts the number of EC2 instances you have running based on conditions you set. It ensures you're running the right number of instances—scaling up when demand increases, and scaling down when demand drops to save costs.

### The final state of the infrastructure is:

![load balance final](https://github.com/user-attachments/assets/07806ff2-ac22-4f76-b5cb-e95ce75597a3)

### I will start with the following infrastructure:

![load balancer 1st](https://github.com/user-attachments/assets/d72eb508-5c93-404e-a836-f24941cd3bf6)

## 🧱 Task 1: Create an AMI for Auto Scaling

We'll create an AMI from the existing **Web Server 1** instance. This ensures every new instance launched by Auto Scaling is pre-configured with the same OS, web server, and app files.

### 🔹 Steps:
1. Go to **EC2** from the AWS Console.
2. In the left sidebar, click **Instances**.
3. Wait for **Web Server 1** to show `2/2 checks passed`.
4. Select **Web Server 1** → Click **Actions > Image and templates > Create image**.
5. Enter:
   - **Image name**: `WebServerAMI`  
   - **Description**: `Lab AMI for Web Server`
   - ![ami image](https://github.com/user-attachments/assets/b06ef124-b12d-4023-8e4c-b921e92012a8)

6. Click **Create image**.
7. Note the **AMI ID** from the confirmation banner.

You'll use this AMI to launch instances in your Auto Scaling group later.

## 🌐 Task 2: Create a Load Balancer

In this task, I set up a **Target Group** and an **Application Load Balancer (ALB)** to balance traffic across EC2 instances in multiple Availability Zones.

### 🔹 Create Target Group

- Go to **EC2 → Target Groups** → Click **Create target group**
- Type: `Instances` | Name: `LabGroup` | VPC: `Lab VPC`
- Skip registering targets → Click **Create**

![target groups 1](https://github.com/user-attachments/assets/d0072a36-3836-4e3d-8274-a544b36def7d)

### 🔹 Create Application Load Balancer

- Go to **Load Balancers** → Click **Create**
- Choose **Application Load Balancer**
- Name: `LabELB` | Scheme: `Internet-facing` | VPC: `Lab VPC`
- Select **Public Subnet 1** and **Public Subnet 2** (in different AZs)

![load balancer subnet](https://github.com/user-attachments/assets/87437232-975c-4bea-976b-8c5fe3fbbf49)

### 🔹 Configure Settings

- **Security Group**: Select `Web Security Group` and remove the default
- **Listener (HTTP:80)**: Forward to `LabGroup`
- Click **Create load balancer**

➡️ Load balancer will say **Provisioning** — moving to the next

![created lb](https://github.com/user-attachments/assets/33ceb466-c9ec-4f57-aa3e-09033b144b63)

## ⚙️ Task 3: Create Launch Template & Auto Scaling Group

### 🔸 Launch Template (`LabConfig`)

A launch template is a blueprint for EC2s created by Auto Scaling.

- Name: `LabConfig`
- AMI: `Web Server AMI`
- Instance Type: `t2.micro`
- Key Pair: `vockey`
- Security Group: `Web Security Group`
- Monitoring: Enabled (CloudWatch - 1 min)

➡️ Click **Create launch template**

![created launch templates](https://github.com/user-attachments/assets/553415d3-3bf7-4754-b746-9d2262e6ba26)
---

### 🔸 Auto Scaling Group (`Lab Auto Scaling Group`)

An Auto Scaling Group launches EC2s based on the template and manages scaling.

#### Step 1: Basic Config
- Name: `Lab Auto Scaling Group`
- Template: `LabConfig`

#### Step 2: Network
- VPC: `Lab VPC`
- Subnets: `Private Subnet 1` & `Private Subnet 2`

#### Step 3: Load Balancer & Monitoring
- Attach to existing target group: `LabGroup`
- Enable CloudWatch group metrics (1-min)

#### Step 4: Scaling Policy
- Desired: `2` | Min: `2` | Max: `6`
- Policy: Target tracking (Average CPU Utilization = 60%)
![configuring the scaling poilicy](https://github.com/user-attachments/assets/d53eaa17-3514-428c-ac6d-e61d56fd57cb)

#### Step 5: Notifications
- Leave as default

#### Step 6: Tags
- Add tag:  
  - Key: `Name`  
  - Value: `Lab Instance`
    ![follow 6 steps](https://github.com/user-attachments/assets/bff6ef94-6834-4c47-8edb-943ac495b8ea)

➡️ Review & Click **Create Auto Scaling Group**

ℹ️ Group will scale up to 2 instances to match desired capacity.

📝 `LabGroup` is a named Target Group — a boundary where EC2 instances will be registered later for load balancing.

![Success auto scale](https://github.com/user-attachments/assets/b9eb5bd9-29b8-40da-8ec3-afac1f2d0f84)

## 🌐 Task 4: Verify Load Balancing is Working

### ✅ Check EC2 Instances

1. Go to **EC2 → Instances**
2. Look for two instances named `Lab Instance` (launched by Auto Scaling)
3. If not visible, wait 30 seconds and click **Refresh**
![LAB INSTANCES](https://github.com/user-attachments/assets/b925fe29-d4b0-4b89-9ad0-53b847f159cc)

---

### ✅ Check Target Group Health

1. Go to **EC2 → Target Groups**
2. Select `LabGroup` → Go to **Targets** tab
3. Wait for both instances to show **Status: healthy**
   - Click **Refresh** if needed
   - ✅ Healthy = passed load balancer health checks

---

### ✅ Test the Load Balancer

1. Go to **EC2 → Load Balancers**
2. Select `LabELB`
3. Copy the **DNS name** (omit the "(A Record)" part)
4. Open a browser and paste the DNS link
5. ![FINAL LOAD](https://github.com/user-attachments/assets/434a4016-ad73-4659-8fd6-c5f758616186)

➡️ If your application loads, **load balancing is working!**

## ✅ Final Check: Running EC2 Instances

At the end of the lab, I verified that the EC2 instances launched by the Auto Scaling Group were active and healthy.

- I saw **4 EC2 instances** named `Lab Instance` and `Web Server 1` in the EC2 console.
- All instances were in the **"running"** state with **2/2 status checks passed**.
- Instances were distributed across **two Availability Zones** (`us-east-1a` and `us-east-1b`), ensuring high availability.
- The original `Web Server 1` was still running and used earlier to create the AMI.

![4 instances](https://github.com/user-attachments/assets/fcc4c2b7-c04f-49b0-89af-d50458e0573f)

📌 This confirms:
- Auto Scaling launched multiple EC2 instances based on the defined launch template.
- Load Balancing and health checks were successful.
- The setup meets the goals of **scalability** and **resilience** using AWS Auto Scaling and ELB.



