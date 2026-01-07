# **Standard Operating Procedure (SOP)**

## **AWS Instance Scheduler – Containerized Backend with Cognito Authentication**
---

## **1. Purpose**

The purpose of this SOP is to define the standardized process for deploying the **AWS Instance Scheduler Backend** using **Docker**, storing the image in Amazon ECR, configuring **Amazon Cognito** for authentication, and deploying infrastructure using **AWS CloudFormation**.

This SOP ensures:

* Secure authentication using Amazon Cognito
* Standardized container deployment
* Infrastructure-as-Code using CloudFormation
* Compliance with enterprise cloud deployment practices

---

## **3. Prerequisites**

Ensure the following are available before starting:

### **3.1 AWS Requirements**

* Active AWS account
* IAM user with permissions for:

  * Amazon ECR
  * Amazon Cognito
  * CloudFormation
  * IAM (Role creation)
  * EC2 / VPC
  
### **3.2 Local System Requirements**

* Docker installed
* AWS CLI installed and configured
* Git installed

### **3.3 Repository**

* Git repository named:
  **`Instance-Scheduler`**
* Repository contains a valid **Dockerfile**

---

## **4. Architecture Overview**

<img width="720" height="382" alt="image" src="https://github.com/user-attachments/assets/a2e96008-3e15-4f6c-9d89-63be92c90f5a" />


The solution consists of:

* **Dockerized Instance Scheduler Backend**
* **Amazon ECR (Private Registry)** to store Docker images
* **Amazon Cognito User Pool** for authentication
* **CloudFormation Template (`with_network.yml`)** for infrastructure deployment
* **IAM Role** for service permissions

---

## **5. Procedure**

---

## **Step 1: Clone the Instance Scheduler Repository**

1. Clone the repository:

   ```
   git clone https://github.com/OT-COE/instance-scheduler.git
   ```
2. Navigate to the project directory:

   ```
   cd Instance-Scheduler
   ```

---

## **Step 2: Build Docker Image**

1. Verify that the `Dockerfile` exists in the root directory.
2. Build the Docker image:

   ```
   sudo docker build --no-cache -t instance-scheduler-backend ./backend
   ```

<img width="933" height="272" alt="image" src="https://github.com/user-attachments/assets/b0399c23-a0e9-4bbb-bea9-8002858cc94b" />

---

## **Step 3: Create Amazon ECR Repository**

1. Go to **AWS Console → Amazon ECR**
2. Select **Private Registry**
3. Click **Create Repository**
4. Repository name:

   ```
   instance-scheduler-backend
   ```
5. Enable:

   * Image scanning (Recommended)
6. Create the repository

<img width="1898" height="542" alt="image" src="https://github.com/user-attachments/assets/e3600ee8-d36a-4dd5-b159-bc2054589b8b" />

---

## **Step 4: Push Docker Image to ECR**

1. Push the image:

   ```
   docker push <ecr-uri>:latest
   ```
<img width="1908" height="593" alt="image" src="https://github.com/user-attachments/assets/bb240cc9-9a17-44e9-8fd4-956887e76114" />

---

## **Step 5: Configure Amazon Cognito**

### **5.1 Create User Pool**

1. Go to **AWS Console → Amazon Cognito**
2. Click **Create User Pool**
3. User Pool Name:

   ```
   User pool - tnjpe
   ```
---

### **5.2 Application Configuration**

* Application Type: **Single Page Application (SPA)**
* Application Name:

  ```
  MY SPA app
  ```

---

### **5.3 Sign-In & Sign-Up Settings**

* Sign-in option:

  *  Email
* Self-registration:

  *  Disabled
* Required attribute for sign-up:

  * Email

---

### **5.4 Security & Authentication Settings**

* Password policy:

  * Minimum 8 characters
  * Uppercase, lowercase, number, special character (Recommended)
* MFA:

  * Optional (As per organization policy)

---

### **5.5 Hosted UI & URLs**

* Callback URL:

  * Not required (as specified)
* Logout URL:

  * Not required

---

### **5.6 Save the Following Values (Mandatory)**

After creation, note down:

* **User Pool ID**
* **App Client ID**
* **Issuer URL**


<img width="1910" height="628" alt="image" src="https://github.com/user-attachments/assets/91f3fa71-15c3-4e55-882a-5a421f7fb775" />
<img width="1911" height="676" alt="image" src="https://github.com/user-attachments/assets/13107ffd-ecce-4ba0-9507-526c3c88ed7b" />
<img width="1900" height="700" alt="image" src="https://github.com/user-attachments/assets/c1229bc1-48c6-46cb-8328-a94a77fba449" />

---

## **Step 6: Update CloudFormation Template**

File:

```
cloudformation/with_network.yml
```

Update the **CognitoAuthorizer** section:

### **Mandatory Attributes to Update**

* **Audience**

  * Value: App Client ID
* **Issuer**

  * Value: Cognito Issuer UR(authority)

 This ensures:

* Only authenticated users can access the backend
* JWT token validation is enforced

---

## **Step 7: Create IAM Role**

1. Go to **AWS Console → IAM → Roles**
2. Create a new role
3. Trusted entity:

   * AWS Service (CloudFormation / Lambda / ECS – based on architecture)
4. Attach required policies:

   * AmazonECRReadOnly
   * CloudWatchLogsFullAccess
   * Custom policies (if backend accesses EC2 / Scheduler APIs)
5. Name the role clearly:

   ```
   InstanceSchedulerExecutionRole
   ```

---

## **Step 8: Deploy CloudFormation Stack**

1. Go to **AWS Console → CloudFormation**
2. Click **Create Stack**
3. Select:

   * **Use existing template**
4. Template source:

   * Upload a template file
   * Upload: `with_network.yml`

---

### **Stack Configuration**

1. Stack Name:

   ```
   instance-scheduler-backend-stack
   ```
2. Parameters:

   * Review and keep defaults or update if required

---

### **Permissions**

* Attach the IAM Role created earlier
* Acknowledge IAM role creation checkbox

---

### **Final Deployment**

1. Click **Next**
2. Review configuration
3. Click **Create Stack**

<img width="1907" height="802" alt="image" src="https://github.com/user-attachments/assets/c8bd5c9d-9513-4c70-8088-91586985cc00" />
---

## **Step 9: Validation & Verification**

* Verify CloudFormation stack status: **CREATE_COMPLETE**
* Confirm:

  * Backend service is running
  * Cognito authentication is enforced
  * ECR image is successfully pulled
* Check CloudWatch logs for errors

---

## **6. Security & Best Practices**

* Use **private ECR repositories only**
* Disable self-sign-up unless business requires
* Follow least privilege for IAM roles
* Enable CloudWatch logging
* Use versioned Docker tags for production

---

## **7. Conclusion**

This SOP defines a secure, scalable, and enterprise-ready approach for deploying the **Instance Scheduler Backend** using AWS native services.
The process ensures:

* Standardized deployments
* Strong authentication
* Infrastructure reproducibility
* Compliance with DevOps best practices

---
