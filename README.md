# blog-generation-lambda-bedrock


# 📝 Generative AI Blog Generator with AWS Bedrock, Lambda, and API Gateway  

This project demonstrates how to build an **end-to-end Generative AI application** on AWS.  
It uses **Amazon Bedrock** (foundation models), **AWS Lambda**, **Amazon API Gateway**, and **Amazon S3** to generate blogs from prompts and save them as text files.  

---

## 📌 Architecture

1. A client (Postman / frontend) sends a request with a **blog topic** → API Gateway.  
2. API Gateway triggers a **Lambda function** (`app.py`).  
3. Lambda calls **Amazon Bedrock** with the prompt.  
4. Bedrock generates a blog (200–300 words).  
5. Lambda saves the generated blog to an **S3 bucket** (`blog-output/` folder with timestamp).  
6. Logs are stored in **CloudWatch Logs**.  


Client → API Gateway → Lambda → Bedrock → S3


---

## 📂 Repository Structure  
```

generative-ai-bedrock-lambda/
│── app.py                     # Lambda function
│── requirements.txt           # Local dependencies
│── README.md                  # Documentation (this file)
│── .gitignore                 # Ignore venv, zip, creds etc.
│
├── infrastructure/
│   ├── lambda-policy.json     # IAM policy for Bedrock + S3
│   ├── boto3-layer/           # Instructions to build boto3 Lambda layer
│   └── deploy\_instructions.md # Setup guide
│
├── postman/
│   ├── BlogAPI.postman\_collection.json   # Postman collection
│   └── example\_payload.json              # Example request body
│
└── outputs/
└── sample\_response.json   # Example response from Bedrock

```

---

## 🚀 Setup Instructions  

### Clone Repo  
bash
git clone <repo-url>
cd generative-ai-bedrock-lambda


### Optional) Create Virtual Environment

bash
python3 -m venv .venv
source .venv/bin/activate   # Mac/Linux
.venv\Scripts\activate      # Windows

pip install -r requirements.txt


### Create S3 Bucket

Example:

```bash
aws s3 mb s3://aws_bedrock_course1
```

### Deploy Lambda Function

* Create a new Lambda function in **us-east-1**.
* Upload `app.py` as code.
* Attach the IAM role from `infrastructure/lambda-policy.json`.

IAM Policy (least privilege example):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["bedrock:InvokeModel"],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": "arn:aws:s3:::aws_bedrock_course1/*"
    }
  ]
}
```

### Create API Gateway

* Create a REST API.
* Add a POST method → integrate with the Lambda function.
* Deploy the API and note the endpoint URL.

### Test with Postman

* Import `postman/BlogAPI.postman_collection.json`.
* Example request body:

```json
{
  "blog_topic": "Applications of Generative AI in Healthcare"
}
```

* Response:

```json
"Blog Generation is completed"
```

---

## Updating boto3 in Lambda (via Layer)

Lambda often has an older boto3 version (which may not support Bedrock).
To update:

```bash
mkdir -p boto3-layer/python
pip install boto3 botocore -t boto3-layer/python/
cd boto3-layer
zip -r boto3-layer.zip python
```

* Go to **Lambda Console → Layers → Create Layer**.
* Upload `boto3-layer.zip`.
* Attach the layer to your Lambda function.

---

## Logs & Monitoring

* All function logs → **CloudWatch Logs**.
* Blog files → saved in S3 under `blog-output/<timestamp>.txt`.
