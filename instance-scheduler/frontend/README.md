# **Standard Operating Procedure (SOP)**

## **Frontend Deployment – Static Website Hosting for Instance Scheduler**

---

## **1. Purpose**

This SOP defines the standardized procedure to **build and deploy the frontend application** of the **Instance Scheduler** as a **static website** using **Amazon S3**, integrated with **Amazon Cognito** for authentication and **Amazon API Gateway** for backend communication.

The objective is to ensure:

* Secure frontend authentication using Amazon Cognito
* Proper environment configuration (`env.json`)
* Reliable static website hosting
* Enterprise-grade deployment practices

---

## **2. Prerequisites**

### **2.1 AWS Requirements**

* AWS account with access to:

  * Amazon Cognito
  * Amazon S3
  * Amazon API Gateway
  * IAM
  * Node.js (LTS version recommended)
  * npm installed
  * Frontend source code available
  * Access to backend API Gateway endpoint

---

## **3. Architecture Overview**

<img width="720" height="382" alt="image" src="https://github.com/user-attachments/assets/a2e96008-3e15-4f6c-9d89-63be92c90f5a" />

The frontend architecture consists of:

* **Static frontend build (HTML, CSS, JS)**
* **Amazon S3** for static website hosting
* **Amazon Cognito User Pool** for authentication
* **Amazon API Gateway** for backend API access

---

## **4. Procedure**

---

## **Step 1: Update Frontend Environment Configuration**

### **4.1 Fetch Cognito Client ID**

1. Go to **AWS Console → Amazon Cognito**
2. Select **User Pool**
3. Navigate to:

   ```
   App clients
   ```
4. Copy the **Client ID**

<img width="1421" height="511" alt="image" src="https://github.com/user-attachments/assets/e79bf994-1fdd-4802-88ce-9f673fe88330" />

---

### **4.2 Fetch API Base URL**

1. Go to **AWS Console → Amazon API Gateway**
2. Select the required API
3. Navigate to:

   ```
   Stages → Default
   ```
4. Copy the **Invoke URL (Default Endpoint)**

 Example:

```
https://fjo2r5l6tc.execute-api.ap-south-1.amazonaws.com
```
<img width="1851" height="802" alt="image" src="https://github.com/user-attachments/assets/19dc84e9-f2b3-4ada-b8e6-ccaed893c415" />

---

### **4.3 Update `env.json` File**

1. Open `env.json`
2. Update the following mandatory attributes:

```json
{
  "CLIENT_ID": "<Cognito-App-Client-ID>",
  "API_BASE_URL": "<API-Gateway-Endpoint>",
}
```
3. Save and validate the file

---

## **Step 2: Build Frontend Application**

1. Verify Node and npm versions:

   ```
   node -v
   npm -v
   ```

2. Install dependencies:

   ```
   npm install
   ```

3. Build the frontend:

   ```
   npm run build
   ```

   <img width="1807" height="904" alt="image" src="https://github.com/user-attachments/assets/7b3e3a46-2647-44d2-a573-88c1a6c066ff" />

### Build Error: Missing `env.json` File

###  Error Message
During the frontend production build, the following error may occur:

### solution 

 1. Create the required directory
  
```
sudo mkdir -p /var/www/env
sudo chown -R ubuntu:ubuntu /var/www
```
2.  Create the env.json file

```
sudo vim /var/www/env/env.json
```
3.  Add environment configuration

```
{
  "API_BASE_URL": "https://fjo2r5l6tc.execute-api.ap-south-1.amazonaws.com/",
  "VITE_COGNITO_CLIENT_ID": "43hdb82hl30qssbgh4r47eah5f"
}
```

4. Delete dist folder 

```
sudo rm -rf dist
```
5. Re-run the build command

```
npm run build
```

<img width="1600" height="618" alt="image" src="https://github.com/user-attachments/assets/78ad930c-9231-457b-8d42-ea33ad0a415d" />

---

## **Step 4: Create Amazon S3 Bucket**

1. Go to **AWS Console → Amazon S3**
2. Click **Create Bucket**

### **Bucket Configuration**

* Bucket Name:

  ```
  instance-shaduler-website01
  ```
* Region:

  * Same as backend (Recommended)

---

### **3.1 Public Access Configuration**

* **Uncheck**:

  * Block all public access
* Acknowledge the warning

 Required for static website hosting

---

### **3.2 Create Bucket**

* Click **Create Bucket**

---

## **Step 4: Enable Static Website Hosting**

1. Open the bucket:

   ```
   instance-shaduler-website01
   ```
2. Go to **Properties**
3. Scroll to **Static website hosting**
4. Click **Edit**
5. Enable:

   * Hosting type: **Host a static website**
   * Index document:

     ```
     index.html
     ```
6. Save changes

---

## **Step 5: Attach Bucket Policy**

Attach the following bucket policy to allow public read access:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::instance-shaduler-website01/*"
    }
  ]
}
```

---

## **Step 6: Upload Frontend Build to S3**

1. Navigate to the build directory:

   ```
   cd dist
   ```

2. Upload all files to the S3 bucket
  
4. Verify objects are uploaded successfully

 <img width="1866" height="816" alt="image" src="https://github.com/user-attachments/assets/ce30e22a-4d2d-409b-9af3-c5314f965800" />

---

## **Step 7: Access Static Website**

1. Go to **S3 → Properties → Static Website Hosting**
2. Copy the **Bucket Website Endpoint**
3. Open the URL in a browser

<img width="1806" height="961" alt="image" src="https://github.com/user-attachments/assets/4354740c-b1cf-4e85-8809-6df8ed911287" />

---

## **Step 8: Configure Cognito Users**

### **8.1 Create User**

1. Go to **AWS Console → Amazon Cognito**
2. Select the user pool
3. Go to **Users**
4. Click **Create User**
5. Provide:

   * Email
   * Temporary password
6. Create user

<img width="1896" height="684" alt="image" src="https://github.com/user-attachments/assets/26c9eafb-76c3-4407-aa2f-50f495fafeee" />

---

### **8.2 Update App Client Authentication Flow**

1. Go to:

   ```
   App clients → instance-scheduler
   ```
2. Click **Edit App Client Information**
3. Enable authentication flow:

   *  USER_PASSWORD_AUTH
4. Save changes

---

### **8.3 Confirm User**

1. Navigate to:

   ```
   Amazon Cognito → Users
   ```
2. Select the user
3. Confirm user status

<img width="1873" height="949" alt="image" src="https://github.com/user-attachments/assets/82ae68e4-c6d7-4589-b4a7-b4ac8b398947" />
<img width="1913" height="706" alt="image" src="https://github.com/user-attachments/assets/1d33097f-4825-4f0b-9b3e-db58a4a167c2" />
<img width="1512" height="123" alt="image" src="https://github.com/user-attachments/assets/45379a55-3662-4e6a-889c-77c701604be7" />

---

## **Step 9: Validation & Testing**

* Open frontend URL
* Login using Cognito user credentials
* Verify:

  * Successful authentication
  * API calls working via API Gateway
  * No CORS or authorization errors

---

## **6. Security & Best Practices**

* Do not hardcode secrets in frontend
* Restrict API Gateway using Cognito Authorizer
* Enable versioning on S3 bucket
* Use separate environments (dev / prod)

---

## **7. Conclusion**

This SOP establishes a **secure, scalable, and standardized approach** for deploying the **Instance Scheduler Frontend** as a static website using AWS services.
It ensures:

* Proper environment configuration
* Secure authentication
* Reliable static hosting
* Enterprise-ready deployment standards
---
