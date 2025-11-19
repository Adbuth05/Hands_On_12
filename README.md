# **Hands-on-12 — Serverless Spark ETL Pipeline on AWS**

This project implements a **fully automated, serverless, event-driven Spark ETL pipeline** using AWS services.  
The pipeline ingests raw CSV review data, triggers downstream processing automatically, executes Spark SQL analytics, and stores the final processed results back into S3 — **no manual job execution required**.

---

## 📊 **Project Overview**

Most organizations struggle with manual data processing steps (upload → run job → export results).  
This project replaces those manual steps with an entirely automated pipeline.

### **Automated Flow**
1. A raw **`reviews.csv`** file is uploaded to an S3 **landing bucket**  
2. The upload event triggers an **AWS Lambda** function  
3. Lambda starts an **AWS Glue Spark ETL job**  
4. Glue reads & transforms the data, runs Spark SQL queries  
5. Results are written as Parquet files to a **processed S3 bucket**

---

## 🏗️ **Architecture**
S3 (Landing Bucket)
│
│ S3 Event Trigger
▼
AWS Lambda (start_glue_job_trigger)
│
│ glue:StartJobRun
▼
AWS Glue ETL Job (process_reviews_job)
│
▼
S3 (Processed Bucket)


**Data Path:**  
**S3 → Lambda → AWS Glue (Spark) → S3**

---

## 🛠️ **Technology Stack**

| Component | Service |
|----------|---------|
| Data Lake | Amazon S3 |
| ETL Engine | AWS Glue (Spark) |
| Serverless Trigger | AWS Lambda |
| Processing Language | PySpark (Spark SQL) |
| Security | AWS IAM |

---

# 🔧 **Setup & Deployment Instructions**

Follow these steps to build the pipeline end-to-end.

---

## **1. Prerequisites**
- AWS Account (Free Tier OK)
- Basic knowledge of S3, Lambda, Glue, IAM
- Template repository:  
  **https://github.com/ITCS6190-Fall2025/Hands-on-12-Spark-on-AWS.git**

---

## **2. Create S3 Buckets**

Create two buckets with globally unique names:

### **Landing Bucket**
handsonfinallanding

### **Processed Bucket**
handsonfinalprocessed

- Upload raw files into the landing bucket  
- Glue will store cleaned and aggregated Parquet files in the processed bucket  

---

## **3. Create IAM Role for AWS Glue**

1. Go to **IAM → Roles → Create role**
2. Select **AWS service**
3. Choose **Glue**
4. Attach:
   - `AWSGlueServiceRole`
   - `AmazonS3FullAccess` (demo)
5. Name the role: AWSGlueServiceRole-Reviews


---

## **4. Create the AWS Glue Spark ETL Job**

1. Open **AWS Glue → ETL Jobs**
2. Click **Create job**
3. Choose **Spark script editor**
4. Delete sample code and paste:
``` src/glue_job_script.py ```

6. In **Job details**:
   - **Name:** `process_reviews_job`
   - **IAM Role:** `AWSGlueServiceRole-Reviews`
7. Save the job

> The script is already configured to use `handsonfinallanding` and `handsonfinalprocessed`.

---

## **5. Create the Lambda Trigger Function**

This Lambda function starts the Glue job when a file is uploaded.

### **5a. Create Lambda Function**

1. Open **AWS Lambda → Create function**
2. Select **Author from scratch**
3. Set:
   - **Function name:** `start_glue_job_trigger`
   - **Runtime:** Python 3.10
4. Permissions → create new role with basic execution rights

---

### **5b. Add Lambda Code**

Paste the content of:
```
src/lambda_function.py
```

Ensure:
```python
GLUE_JOB_NAME = "process_reviews_job"
```
Click Deploy

## **5c. Add Permission for Lambda to Start Glue Jobs**

Your Lambda function must be allowed to trigger AWS Glue.  
To enable this, add an inline IAM policy to the Lambda execution role.

### **Steps:**
1. Open **AWS Lambda → Your Function → Configuration → Permissions**
2. Under *Execution Role*, click the role name (this opens IAM)
3. In IAM, click **Add permissions → Create inline policy**
4. Choose the **JSON** editor and paste the following:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "glue:StartJobRun",
      "Resource": "*"
    }
  ]
}
```

Click Next

Name the policy: Allow-Glue-StartJobRun
This gives Lambda the ability to start your Glue job automatically.

## **5d. Add S3 Trigger**

To make the pipeline fully automated, you must connect your S3 landing bucket to the Lambda function.  
This ensures that whenever a new file (such as `reviews.csv`) is uploaded, the Lambda function is triggered automatically.

### **Steps to Add the S3 Trigger:**

1. Open **AWS Lambda → Your Lambda Function**
2. Click **Add trigger**
3. Select **S3** as the trigger type
4. Configure the trigger:
   - **Bucket:** `handsonfinallanding`
   - **Event type:** `s3:ObjectCreated:*`
5. (Optional) Add a suffix filter to only trigger on CSV files: .csv
6. Click **Add** to save the trigger

### **Result**
After adding this trigger, **any new file uploaded** to the `handsonfinallanding` bucket will automatically start the Lambda function, which will then trigger the Glue ETL job.

## **6. Next Steps**

With the S3 trigger successfully added, your serverless ETL pipeline is now fully automated.  
The next steps involve testing the pipeline, monitoring the workflow, verifying outputs, adding required Spark queries, and preparing your submission.

---

## **6a. Test the Pipeline**

1. Download or locate the provided `reviews.csv` file  
2. Upload it to the landing bucket: s3://handsonfinallanding/
3. This upload will automatically:
- Trigger the Lambda function  
- Start the AWS Glue Spark ETL job  
- Begin processing the CSV file  

You should see a new Glue job run appear within a few seconds.

---

## **6b. Monitor the Workflow**

### **Monitor Lambda Execution**
- Go to **AWS Lambda → Monitor → CloudWatch Logs**
- Check for:
- Successful invocation messages
- Any `StartJobRun` failures or errors

### **Monitor Glue Job**
- Open **AWS Glue → Jobs → process_reviews_job**
- View:
- Job run status  
- Logs  
- Execution times  

A successful run will show a status of **Succeeded**.

---

## **6c. Verify Processed Outputs in S3**

After Glue completes, navigate to your processed bucket: s3://handsonfinalprocessed/

You should see:

processed-data/
Athena Results/daily_review_counts/
Athena Results/top_5_customers/
Athena Results/rating_distribution/


Each folder will contain **Parquet files** generated by Spark.

---

## **6d. Add Your Three Additional Spark Queries (Required)**

Update `glue_job_script.py` by adding **three new analytical queries**.

Examples include:
- Most reviewed products  
- Average rating by product category  
- Reviews grouped by weekday  
- Top 10 highest-rated items  
- Customers with highest number of low ratings  

Each new query must:
- Use Spark SQL  
- Create a DataFrame  
- Be written to a **separate S3 output folder** under `Athena Results/`

---

## **6e. Prepare Final Submission**

Your final GitHub repository must include:

### ✔ Code Files
/src
glue_job_script.py
lambda_function.py

/data
reviews.csv
README.md
![1](https://github.com/user-attachments/assets/22fb82cd-ed1e-4016-bd02-6a9fdeb2bce5)


![2](https://github.com/user-attachments/assets/30376072-266a-42e8-989a-a7d81c32546a)


![3](https://github.com/user-attachments/assets/4c4671cd-5a58-4556-ad4d-f1a20a35f330)

![4](https://github.com/user-attachments/assets/bc4bd565-e089-4fa5-8592-56bddc1a65ab)


![5](https://github.com/user-attachments/assets/64821004-d8ad-45b8-9e6f-c40cefb25eee)


![6](https://github.com/user-attachments/assets/09ccc672-dfcd-4da5-bea8-22537d16d62a)


![7](https://github.com/user-attachments/assets/bc6f852e-aca0-4a94-a9c7-f67549374d5c)







