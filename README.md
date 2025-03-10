# Deploying an NPM Project on AWS EC2 (Not Recommended for Production)

This document explains the step-by-step process to deploy an NPM (Node.js) project on an AWS EC2 instance. It also details **why directly deploying on EC2 is not recommended** for production environments and suggests better alternatives.

---

## **1️⃣ Deployment Steps**

### **Step 1: Launch an EC2 Instance**
- Choose **Ubuntu 20.04 or Amazon Linux 2** as the OS.
- Select an **instance type** (e.g., t2.micro for free tier).
- Configure **security group** to allow SSH (port 22) and application traffic (port 5000).
- Launch the instance and connect via SSH:
  ```sh
  ssh -i your-key.pem ubuntu@your-ec2-ip
  ```

### **Step 2: Install Node.js and Git**
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install nodejs npm git -y
```
Check the installation:
```sh
node -v
npm -v
```

### **Step 3: Clone the Project from GitHub**
```sh
git clone https://github.com/your-repo/chat-app.git
cd chat-app
```

### **Step 4: Install Dependencies**
```sh
npm install
```

### **Step 5: Start the Application**
```sh
node index.js
```
If successful, your server will be running on **port 5000**.

### **Step 6: Allow Traffic on Port 5000 (Security Group Configuration)**
- Go to **AWS EC2 Dashboard > Security Groups**.
- Add a new inbound rule:
  - **Type:** Custom TCP
  - **Port Range:** 5000
  - **Source:** Anywhere (0.0.0.0/0) or your IP

Now, you can access the app via `http://your-ec2-ip:5000`.

### **Step 7: Keep the App Running in the Background (Using PM2)**
By default, the app **stops when you close SSH**. To fix this:
```sh
npm install -g pm2
pm2 start index.js
pm2 save
pm2 startup
```

---

## **2️⃣ Why Directly Deploying on EC2 is a Bad Practice?**

### ❌ **1. Application Stops When SSH is Closed**
- Running `node index.js` manually means the app **stops** when the SSH session ends or the EC2 instance restarts.
- **Solution:** Use **PM2** to keep the app running, but this is still not ideal.

### ❌ **2. No Auto-Scaling (Cannot Handle High Traffic)**
- A single EC2 instance **cannot handle high traffic**.
- If many users access the app, **the server may crash** due to high CPU/memory usage.
- **Better Approach:** Use **AWS Load Balancer + Auto Scaling** or **AWS Elastic Beanstalk**.

### ❌ **3. Security Risks (Direct Public Exposure)**
- The EC2 instance is directly exposed to the internet, making it vulnerable to attacks.
- **Better Approach:** Use **Nginx as a reverse proxy** and enable **HTTPS (SSL encryption)**.

### ❌ **4. Manual Deployment (No CI/CD)**
- Every time you update the project, you must:
  - SSH into EC2
  - Pull changes from GitHub
  - Restart the application
- **Better Approach:** Use **AWS CodeDeploy or GitHub Actions** for automatic deployments.

### ❌ **5. EC2 Doesn't Restart the App on Crash**
- If the application **crashes**, EC2 **does not restart it automatically**.
- **Better Approach:** Use **Docker + AWS ECS (Elastic Container Service)** for auto-recovery.

---

## **3️⃣ Better Alternatives for Deployment**
### ✅ **1. AWS Elastic Beanstalk** (Best for Small to Medium Apps)
- Automatically manages EC2, load balancing, scaling, and deployment.
- Deploy using:
  ```sh
  eb init
  eb create my-app-env
  ```

### ✅ **2. AWS ECS (Elastic Container Service) with Docker** (Best for Scaling)
- Package your app in a **Docker container** and run it in AWS ECS.
- More reliable and easier to scale than a single EC2 instance.

### ✅ **3. AWS Lambda + API Gateway** (For Serverless Apps)
- No need for EC2 at all! Use **AWS Lambda** to run backend functions only when needed.

---

## **4️⃣ Summary**
| Approach | Pros | Cons |
|----------|------|------|
| **Directly Deploying on EC2** | Simple, works for small projects | App stops when SSH closes, no scaling, security issues |
| **Using PM2 on EC2** | Keeps app running after SSH closes | Still needs manual scaling & deployment |
| **AWS Elastic Beanstalk** | Auto-manages EC2, scaling, and deployment | Slightly complex setup |
| **AWS ECS (Docker)** | Best for production, auto-recovery | Requires Docker knowledge |
| **AWS Lambda + API Gateway** | No need to manage servers | Only works for serverless apps |

---

## **5️⃣ Next Steps**
🔹 If you **just want to keep your EC2 deployment running**, use **PM2 + Nginx**.  
🔹 If you **want a better deployment**, try **AWS Elastic Beanstalk** or **Docker on ECS**.  
🔹 If you **want full automation**, set up **CI/CD with AWS CodeDeploy or GitHub Actions**.

🚀 **For long-term scalability, DO NOT use EC2 directly for deployment.** Use AWS services designed for this purpose instead.
