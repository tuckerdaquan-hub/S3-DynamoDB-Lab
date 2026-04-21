# 🗄️ AWS S3 + DynamoDB — Configuring Storage for a Web Application
  
**Lab:** Lab 3 — Configuring a Web Application with Amazon S3 and DynamoDB  
**Console:** AWS Management Console 

---

## 📋 Objective

Configure the storage layer for a real, running web application — an Employee Directory App hosted on an EC2 instance. This lab involves two AWS storage services working together: Amazon S3 to store employee photos (object storage), and Amazon DynamoDB to store employee records like names, locations, and emails (NoSQL database). The goal is to understand how cloud applications separate compute, object storage, and database storage — and how IAM policies tie them all together securely.

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **AWS Management Console** | Browser-based interface for managing all AWS services |
| **Amazon S3** | Creating a bucket, uploading employee photo objects, and writing a bucket policy |
| **Amazon DynamoDB** | Creating an `Employees` table, defining a partition key, and inserting records |
| **IAM Bucket Policy (JSON)** | Granting an IAM Role (EC2 app) read access to the S3 bucket |
| **Employee Directory App** | Live web application running on EC2 that reads from both S3 and DynamoDB |
| **Coursera / AWS Labs** | Guided lab environment providing the pre-built AWS account and EC2 app |

---

## ✅ Skills Learned

- Understanding the difference between **object storage (S3)** and **NoSQL database storage (DynamoDB)**
- Creating an S3 bucket with a globally unique name using the correct bucket type (**General Purpose**)
- Understanding S3 bucket namespace types: **Global namespace** vs. **Account Regional namespace**
- Writing and applying an **S3 Bucket Policy** in JSON to grant an IAM Role access to a specific bucket
- Understanding the IAM policy elements: `Sid`, `Effect`, `Principal`, `Action`, and `Resource`
- Scoping an S3 policy correctly using two ARN entries — one for the bucket itself, one for all objects inside it (`bucket/*`)
- Uploading multiple objects (10 employee photo `.png` files, ~802 KB total) to an S3 bucket
- Creating a **DynamoDB table** (`Employees`) with a **partition key** (`id`, type: String)
- Understanding DynamoDB's **schemaless** design — only a table name and partition key are required at creation
- Adding items to a DynamoDB table with custom attributes: `id`, `name`, `location`, `email`, `photo`
- Understanding how a web application reads from **both S3 and DynamoDB simultaneously** to render a complete page
- Verifying end-to-end functionality by seeing employee records appear in the live **Employee Directory App**

---

## 📸 Step-by-Step Walkthrough

---

### Step 1 — Writing the S3 Bucket Policy

**Service:** Amazon S3 → `employee-photo-bucket-dtt-3546` → Permissions → Edit Bucket Policy

Wrote and applied a JSON bucket policy to grant the EC2 application's IAM Role (`EmployeeDirectoryAppRole`) full S3 access to the employee photo bucket. The policy structure breaks down as:

- **`Sid`**: `"AllowS3ReadAccess"` — a human-readable label for the statement
- **`Effect`**: `"Allow"` — explicitly permits the action
- **`Principal`**: the IAM Role ARN (`arn:aws:iam::253397837122:role/EmployeeDirectoryAppRole`) — *who* gets access
- **`Action`**: `"s3:*"` — all S3 actions
- **`Resource`**: two entries — the bucket itself AND all objects inside it (`bucket-name/*`) — required to cover both bucket-level and object-level operations

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::253397837122:role/EmployeeDirectoryAppRole"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::employee-photo-bucket-dtt-3546",
        "arn:aws:s3:::employee-photo-bucket-dtt-3546/*"
      ]
    }
  ]
}
```

This is a real-world S3 security pattern: applications never use hardcoded credentials — they assume an IAM Role, and the bucket policy defines exactly what that role can do.

<img width="1366" height="768" alt="S3 + DynamoDB (1)" src="https://github.com/user-attachments/assets/0562425e-26b6-42b9-968c-3889ecc4105e" />

---

### Step 2 — Creating the S3 Bucket

**Service:** Amazon S3 → Create Bucket

Created a new General Purpose S3 bucket named `employee-photo-bucket-dtt-3546` in the Global namespace. Key decisions made during creation:

- **Bucket type**: General Purpose — the standard type that supports all storage classes and redundancy across multiple Availability Zones (vs. Directory buckets, which are single-AZ optimized for speed)
- **Bucket namespace**: Global — meaning the bucket name must be unique across *all* AWS accounts worldwide
- **Naming**: `employee-photo-bucket-dtt-3546` — the suffix (`dtt-3546`) ensures global uniqueness, a real-world requirement for every S3 bucket

<img width="1366" height="768" alt="S3 + DynamoDB (2)" src="https://github.com/user-attachments/assets/520c180c-87d8-4c4b-8e42-c5103f4f7069" />

---

### Step 3 — S3 Bucket Successfully Created

**Service:** Amazon S3 → `employee-photo-bucket-dtt-3546`

The green confirmation banner confirms: *"Successfully created bucket 'employee-photo-bucket-dtt-3546'."* The bucket's Objects tab shows 0 objects — it's empty and ready to receive uploads. The tab structure visible (Objects, Metadata, Properties, Permissions, Metrics, Management, Access Points) represents the full lifecycle management surface for an S3 bucket.

<img width="1366" height="768" alt="S3 + DynamoDB (3)" src="https://github.com/user-attachments/assets/319da468-d21e-4a7a-bb28-90d64f5019a7" />

---

### Step 4 — Creating a DynamoDB Item (Employee Record)

**Service:** Amazon DynamoDB → Tables → `Employees` → Create Item

Added a new employee record to the `Employees` DynamoDB table by manually defining attributes:

| Attribute | Value | Type |
|-----------|-------|------|
| `id` *(Partition Key)* | `id` | String |
| `name` | `Kay` | String |
| `location` | `Maine` | String |
| `email` | `employee4@hotmail.com` | String |
| `photo` | *(empty)* | String |

This demonstrates DynamoDB's schemaless nature — you define only the partition key at table creation, then each item can have any additional attributes you choose. The `photo` attribute links to the corresponding object stored in S3 (e.g., `employee-4.png`), showing how S3 and DynamoDB work together: DynamoDB holds the metadata, S3 holds the actual file.

<img width="1366" height="768" alt="S3 + DynamoDB (4)" src="https://github.com/user-attachments/assets/979c5ca5-8869-48a2-90a8-c85733ca94d4" />

---

### Step 5 — Uploading Employee Photos to S3

**Service:** Amazon S3 → `employee-photo-bucket-dtt-3546` → Upload

Uploaded 10 employee photo files to the S3 bucket — `employee-1.png` through `employee-10.png`, each ranging from ~67 KB to ~103 KB. These files are the objects that the Employee Directory App will fetch and display alongside each employee's DynamoDB record. This step demonstrates the core S3 use case: storing and serving unstructured binary data (images, files, media) that a relational or NoSQL database is not designed to hold.

<img width="1366" height="768" alt="S3 + DynamoDB (5)" src="https://github.com/user-attachments/assets/17ae8da5-e8d9-47ce-845f-807437c1ce33" />

---

### Step 6 — Creating the DynamoDB Table

**Service:** Amazon DynamoDB → Create Table

Created the `Employees` DynamoDB table with the following configuration:

- **Table name**: `Employees`
- **Partition key**: `id` (type: String) — the unique identifier used to retrieve and distribute items across DynamoDB's infrastructure
- **Sort key**: none (optional, not needed for this use case)

The console description confirms DynamoDB's core design philosophy: *"DynamoDB is a schemaless database that requires only a table name and a primary key when you create the table."* This is fundamentally different from relational databases (like MySQL or PostgreSQL), which require you to define every column and its data type upfront. DynamoDB's flexibility makes it a popular choice for cloud-native applications with evolving data structures.

<img width="1366" height="768" alt="S3 + DynamoDB (6)" src="https://github.com/user-attachments/assets/f5259cbe-f7fe-4a0c-b4d7-b2ed60ad6f1d" />

---

### Step 7 — Employee Directory App Live and Working

**Service:** Employee Directory App (running on EC2 at `ec2-54-166-161-198.compute-1.amazonaws.com`)

With both S3 (photos) and DynamoDB (employee records) configured, the Employee Directory App is fully functional. The app confirms *"Employee 'Brad' added successfully!"* and displays a live table of employees — Sen (California), Shea (New York), and Brad (Texas) — each with a green ✅ Photo Available status, confirming the app is successfully fetching records from DynamoDB *and* photos from S3 simultaneously.

The URL `ec2-54-166-161-198.compute-1.amazonaws.com` confirms this is a real application running on a public-facing EC2 instance, not a simulation. The right panel shows a live employee photo (`employee-3.png`) being loaded directly from the S3 bucket via the IAM Role policy set up in Step 1.

<img width="1366" height="768" alt="S3 + DynamoDB (7)" src="https://github.com/user-attachments/assets/fcf2da5f-5636-4ba4-8577-5ad8b3c3a40a" />

---
