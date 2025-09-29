# Event-Driven Sales Data Processing Project

## Project Overview
This project demonstrates an **event-driven serverless data processing pipeline** on AWS. It automatically processes sales order data uploaded to an S3 bucket, validates it, stores it in DynamoDB, and handles failed orders via SQS dead-letter queue (DLQ).  

The solution leverages **AWS Step Functions**, **Lambda**, **S3**, **DynamoDB**, and **SQS** to create a scalable, reliable, and fault-tolerant pipeline.

## Architecture
      +-----------------+
      |  S3 Bucket      |
      | orders-data-p   |
      +--------+--------+
               |
               v
      +-----------------+
      | Step Functions  |
      | State Machine   |
      +--------+--------+
               |
       +-------+--------+
       | Map State       |
       +-------+--------+
               |
    +----------+----------+
 +---------------+ +-----------------+
| Lambda | | SQS DLQ |
| ValidateOrder | | order-data-dlq |
+-------+-------+ +-----------------+
             |
              v
        +---------------+
          | DynamoDB |
        | ecom-orders |
        +---------------+       
  
---

## Tech Stack
- **AWS Lambda** – Serverless compute to validate orders  
- **AWS Step Functions** – Orchestrates the workflow  
- **Amazon S3** – Stores input order files  
- **Amazon DynamoDB** – Stores processed orders  
- **Amazon SQS** – Dead-letter queue for failed orders  
- **Python 3.13** – Lambda runtime

---

## How It Works
1. **Upload CSV/JSON**: A sales order file is uploaded to `orders-data-p` S3 bucket.  
2. **Step Functions Triggered**: The Step Functions state machine is invoked with the uploaded file details.  
3. **GetObject**: S3 `getObject` retrieves the file content.  
4. **Map State**: Processes each order in the file individually:  
   - **ValidateOrder Lambda** checks for `contact-info` and `order-info`.  
   - If valid → store in **DynamoDB (`ecom-orders`)**.  
   - If invalid → send to **SQS DLQ (`order-data-dlq`)**.  
5. **Retry & Error Handling**: Failed DynamoDB writes are retried up to 3 times with exponential backoff.  

---

## Real-World Use Cases
- **E-commerce Platforms**: Automatically process incoming orders in real-time.  
- **Data Pipelines**: ETL workflows for order/transaction data.  
- **Error Handling & Reliability**: Ensure failed transactions are safely queued for review.  
- **Scalable Serverless Solutions**: Process thousands of orders without managing servers.

---

## Deployment / Run Instructions
1. **Upload Lambda Code**: Ensure `orders-data` Lambda is deployed with Python 3.13 runtime.  
2. **Configure Step Functions**: Replace resource ARNs with your AWS account-specific ARNs.  
3. **Create DynamoDB Table**: `ecom-orders` with `OrderId` as primary key.  
4. **Create S3 Bucket**: `orders-data-p` for uploading order files.  
5. **Create SQS Queue**: `order-data-dlq` for handling failed orders.  
6. **Upload Order Files**: Trigger the pipeline by uploading JSON/CSV files.

---

## Example Input
```json
{
  "orders": [
    {
      "order-info": {
        "OrderId": "1",
        "item-id": "101",
        "item-desc": "Item One",
        "qty": "1"
      },
      "contact-info": {
        "name": "Foo Bar",
        "email": "foobar@test.com"
      }
    }
  ]
}
Author
Prateek Shaw


