# System Design - Financing solution for insurance policy

## Introduction

In this document, we are going to create a system design for the hypothetical scenario described below. This is intended for educational purposes and can be used to prepare for interviews as well as applied in real-life scenarios with relevant modifications to fit the actual use case.

## Hypothetical Scenario

**Payments Team Responsibilities:**
In this scenario, the Payments team is tasked with enhancing the checkout experience, which includes customer data confirmation, e-signatures, financing, payment collection, and invoicing integrations using the MERN stack. Your role involves providing technical direction and guidance to the team, and collaborating with managers to develop the necessary skills for successful technical implementations.

**Building an In-House Financing Solution:**
Due to recent growth, the organization decided to develop an in-house financing solution for customers, and the Payments team was selected for this task. This system allows customers to pay for their insurance policy monthly over a 12-month term. The system receives the total cost of the product and does not calculate fees or taxes on the initial purchase. It needs to handle various payment and cancellation terms.

**Business Requirements:**
- **Scheduled Charging:** The system must charge customers scheduled amounts via Stripe (Credit or Direct Debit).
- **Non-Sufficient Funds:** If a charge fails, the system should (1) notify the customer, (2) retry the next day. If the second attempt fails, (3) create a record in Salesforce for manual handling by a broker.
- **Mid-Term Adjustments:** The system must handle adjustments to existing policy terms and calculate payment changes. For example, if a customer increases their coverage limit, the financing plan should reflect the new premium cost.
- **Cancellations:** The system should calculate refunds based on the cancellation terms when a customer cancels their insurance.
- **Communication:** The system must send receipts to customers for every charge or change in their financing terms.

**Team Composition:**
The Payments team is a cross-functional group skilled in the MERN stack, quality practices, and DevOps. However, the team members have limited experience with distributed services.

This hypothetical scenario illustrates a comprehensive technical and collaborative challenge, requiring a mix of software development, team leadership, and problem-solving skills.

# System Design Document

## Objective

Build a financing solution that allows paying monthly for an insurance policy over the length of its term, which is usually 12 months. The financing system should accept the total cost of the product, it won’t have to calculate fees or taxes on the initial purchase. The system should support various payment and cancellation terms.

## Payments Team Responsibility

1. Checkout experience
2. Customer data confirmation/validation
3. E-Sign
4. Financing
5. Payments collection
6. Invoicing integrations, backed by third-party solutions
7. Implementation of solutions using MERN (MongoDB, Express, React, Node) stack

## Payments Team Skills

1. Work cross-functionally
2. MERN (MongoDB, Express, React, Node) stack
3. DevOps
4. Some experience with distributed services

## Functional Requirements

1. **Scheduled Charging:**
   - Customers are charged for scheduled amounts via the **Payments Gateway**.

2. **Handling of Non-Sufficient Funds:**
   - Notify the customer.
   - Retry the next day.
   - Create a Salesforce record on the second failure for manual handling.

3. **Handling of Mid-Term Adjustments:**
   - Adjust insurance policy terms.
   - Calculate payment change delta.
   - Change the existing financing plan to charge a new amount.

4. **Handling of Cancellations:**
   - Calculate the refund based on the cancellation terms provided when the payment terms were initially started.

5. **Communication:**
   - Send receipts to the customers on every charge.
   - Send an email to the customers on every change of financing terms.
   - Send a notification to the customers on non-sufficient funds errors.

## Non-Functional Requirements

1. **Secure:**
   - Use Serverless that utilizes secure AWS infrastructure.
   - Use JWT Authorizers with API Gateway and Amazon Cognito.
   - CloudFront with Web Application Firewall.
   - SSL for web-facing endpoints.

2. **High availability:**
   - Redundancy by distributing services across multiple servers is automatically handled by serverless.
   - Automatic failover provided by serverless.
   - Monitoring, available on AWS.

3. **High Scalability:**
   - Leverage autoscaling offered by serverless architecture.

4. **Minimal Operational Costs:**
   - Serverless architecture allows the company to be charged only for what it uses.
   - Minimize the number of various technologies for more streamlined billing.

5. **Minimize the efforts:**
   - Reuse as much of the current stack as possible.
   - Use cloud technologies to simplify the architecture.
   - No need to manage infrastructure and security with serverless architecture.
   - Minimize the number of moving parts for simplicity of the architecture.

## Assumptions

   1. The company is using Amazon AWS.

   2. Selection of the technologies has been approved.

   3. Security mechanism for communication between financing and pre-existing microservices already exists and we don’t have to design it.

   4. Financing system only needs to provide a **customer_id** to existing services to identify a customer they have to work against.

   5. A **Payments Gateway** that integrates with Stripe already exists. It returns errors and PDF receipts. It only needs **customer_id** and the **amount** to charge the customer.

   6. The only possible **Payments Gateway** error is Non-Sufficient Funds.

   7. Payment processing fees and taxes are calculated by a payment processor and don’t have to be stored anywhere.

   8. **Notifications Gateway** already exists and it can send emails, sms, and so on.
   It only needs **customer_id** to build an email template and send it.
   We don’t have to handle possible complications with sending notifications in our design.

   9. **Salesforce Gateway** already exists and it can create records in Salesforce for a given **customer_id**. We don’t have to handle possible complications with creating a record in Salesforce in our design.

   10. We have existing authentication and security mechanisms that can be reused. Design only includes a possible high-level solution for it, but without any extra details.

   11. The website (platform) has an existing mechanism to hook micro-frontends into it.

   12. Financing system expects the day of the month when customers should be billed at the time when payment terms were initially created.

   13. Payments scheduled for the 29th, 30th, and 31st days of the month will be billed on the last day of the month when a month has fewer days than the scheduled day.

   14. Monthly payment amounts remain the same for the duration of the term unless **total_amount** has been adjusted.

   15. Only the total cost of the payment terms can be adjusted.

   16. Cancellation terms are only provided to the system when payment terms are initially started.

   17. Payment terms should be canceled on the same day when the cancellation has been requested by the customer. The customer can’t choose a date when it should be canceled.

   18. We don’t have to define possible cancellation terms at this point. The design assumes flexible cancellation terms and that code can handle it.

## Proposed System Architecture

![System Design Diagram](images/system_design_diagram.png)

## Design Overview

High-level description of the proposed system architecture with a bit of detail to better explain the intended functionality.

1. **CDN (CloudFront):**
   Serves our static HTML, CSS, and JS resources from Amazon S3.

2. **Financing Micro-Frontends:**
   - Users interact with micro-frontends.
   - Micro-frontends are hooked into the website (platform).
   - They send REST API requests to individual endpoints.

   **Description of the components:**

   - **New terms micro-frontend:** communicates with the “Create Payment Terms API”.
   - **Terms adjustments micro-frontend:** communicates with the “Adjust Payment Terms API”.
   - **Terms cancellation micro-frontend:** communicates with the “Cancel Payment Terms API”.

3. **API Gateway with Authentication (Cognito):**
   Authenticates users and ensures secure communication with the public backend API.

4. **Financing Backend APIs:**
   - Backend APIs are implemented as serverless lambda functions.
   - Handle requests from micro-frontends.

    **Description of the components:**

   - **Create Payment Terms API:** adds new payment terms to the database.
   - **Adjust Payment Terms API:** updates payment terms recorded in the database. Calculates the delta and creates new transactions to compensate for the difference in a monthly amount.
   - **Cancel Payment Terms API:** updates payment terms record to pending cancellation.

5. **Database (MongoDB):**
   - Stores Payment Terms and Transactions.
   - Triggers **AWS EventBridge** to communicate the changes to the rest of the system.

   **Description of the collections:**

   - **Payment Terms:** stores **customer_id**, payment terms, and cancellation terms.
   - **Transactions:** stores payment transactions with their status.

6. **Payment Terms Update Trigger Handler:**
   - MongoDB calls it for updates in the **Payment Terms** collection.
   - Creates a refund transaction when the status is changed to pending cancellation.

7. **Transactions Update Trigger Handler:**
   - MongoDB calls it for updates in the **Transactions** collection.
   - Changes **Payment Terms** record’s status to canceled when the refund has been processed.

8. **Event Bus (AWS EventBridge):**
   - Routes events triggered by MongoDB and other services to the relevant serverless lambda functions.
   - Archives all the messages for disaster recovery and audit purposes.
   - Adds undelivered messages into **Dead Letter Queue (SQS)** for further investigation.

9. **Dead Letter Queue (SQS):**
   Contains undelivered messages that require further processing.

10. **Dead Letter Queue Handler:**
    Monitors queue for messages and notifies the Team about them for further action.

11. **CRON (AWS EventBridge):**
    Executes **Payment Terms Loader** on a defined schedule.

12. **Payment Terms Loader:**
    Adds a new item to the **Transactions** collection for the payment terms that need to be charged.

13. **Transactions Creator:**
    Reads **Payment Terms Queue** and creates pending transactions.

14. **Payments Collector Queue (SQS):**
   - FIFO queue that prevents duplicate messages.
   - Holds transactions that have to be processed.

15. **Payment Collector:**
   - Reads **Payments Collector Queue** and collects payments via the **Payments Gateway**.
   - Notifies **Event Bus** about the status of the payment.

16. **Transactions Updater:**
   - Updates transactions with the new status in the database.

17. **Failed Payments Notifier:**
    Sends a payment error email template via **Email Gateway** for a current **customer_id**.

18. **PDF Receipt Sender:**
    Downloads and then sends a PDF Receipt email template via **Email Gateway** for a current **customer_id**.

19. **Payment Terms Notifier:**
    Sends an email template with the new or updated Payment Terms via **Email Gateway** for a current **customer_id**.

20. **Salesforce Record Creator:**
    Creates a new record in Salesforce for a given **customer_id** in **Salesforce Gateway**.

21. **Payments Gateway:**
    Creates an invoice, executes the payment, and returns the result including a link to a PDF of the invoice/receipt. It communicates billing errors back to the caller.

22. **Notifications Gateway:**
    Builds a message for a given **customer_id** and a message body template and then handles the delivery of it.

## Recommended Technologies

   1. **Atlas MongoDB** as a database. 
   *Reason:* The team knows how to work with MongoDB, it is scalable and reliable.

   2. **Atlas Triggers** to emit events on database changes. 
   *Reason:* Easy to use with MongoDB Atlas.

   3. **AWS Lambda** to implement individual actions of our services. 
   *Reason:* Easy to use, scalable, low cost.

   4. **AWS EventBridge** to reliably communicate changes between various services. 
   *Reason:* Allows to decouple microservices, adds resilience, and keeps an archive of all the events.

   5. **AWS SQS** as FIFO queue with deduplication. 
   *Reason:* Allows to decouple microservices, adds resilience and helps with debugging.

   6. **AWS S3** to store static front-end files.
   *Reason:* Low cost, scales automatically in static web hosting mode.

   7. **AWS CloudFront** to speed up the loading of the micro-frontends by delivering content through edge locations.
   *Reason:* CDN, Caching, All requests will be proxied, possible to add extra security with the Web Application Firewall.

   8. **AWS API Gateway with Amazon Cognito** to route, authenticate, and secure communication between front-end and backend APIs.
   *Reason:* Native for AWS infrastructure, has a lot of flexibility to choose different options.

   9. **REST API** for Frontend to Backend communication.
   *Reason:* public-facing application is fairly simple, but alternatively any API style will work.

## Skills Needed to Implement the System

System Architecture is designed to minimize the learning curve and fit into a MERN stack. Most of the new skills required are related to Amazon AWS and serverless.

**The list of additional skills required:**
   1. How to use MongoDB Streams / Triggers.
   2. How to connect MongoDB with AWS EventBridge.
   3. Basics of serverless architecture and development.
   4. How to use and secure AWS Lambda for implementing microservices.
   5. How to set up Amazon CloudFront with Amazon S3 to serve static content.
   6. How to set up AWS EventBridge rules.
   7. How to set up AWS API Gateway with Amazon Cognito for user authentication.

## References for Acquiring Required Skills

   - **[How to set up a CloudFront distribution for Amazon S3](https://aws.amazon.com/cloudfront/getting-started/S3/)**
   - **[How to use CloudFront to serve a static website hosted on Amazon S3](https://youtu.be/_JMf1mMjbMQ)**
   - **[Atlas Triggers](https://www.mongodb.com/docs/atlas/app-services/triggers/)**
   - **[Send Trigger Events to AWS EventBridge](https://www.mongodb.com/docs/atlas/app-services/triggers/aws-eventbridge/)**
   - **[MongoDB Streams to AWS EventBridge](https://youtu.be/4qHwgKeL5hE)**
   - **[AWS EventBridge](https://aws.amazon.com/eventbridge/)**
   - **[Ingesting MongoDB Atlas data using Amazon EventBridge](https://aws.amazon.com/blogs/compute/ingesting-mongodb-atlas-data-using-amazon-eventbridge/)**
   - **[Use Amazon EventBridge to Build Decoupled, Event-Driven Architectures](https://pages.awscloud.com/AWS-Learning-Path-How-to-Use-Amazon-EventBridge-to-Build-Decoupled-Event-Driven-Architectures_2020_LP_0001-SRV.html?pg=ln&cp=bn)**
   - **[Using Lambda with Amazon SQS](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)**
   - **[Serverless architecture with Mongo Atlas](https://docs.aws.amazon.com/prescriptive-guidance/latest/migration-mongodb-atlas/serverless.html)**
   - **[Schedule AWS Lambda Functions Using CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/RunLambdaSchedule.html)**
   - **[Serverless Scheduling with Amazon EventBridge and AWS Lambda](https://aws.amazon.com/blogs/architecture/serverless-scheduling-with-amazon-eventbridge-aws-lambda-and-amazon-dynamodb/)**
   - **[Control access to a REST API using Amazon Cognito](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)**
   - **[Integrating Amazon Cognito User Polls with API Gateway](https://aws.amazon.com/blogs/mobile/integrating-amazon-cognito-user-pools-with-api-gateway/)**
   - **[Use JWT Authorizers with Amazon Cognito and API Gateway](https://www.youtube.com/watch?v=o7OHogUcRmI)**

## Design of the Backend Endpoints

Endpoints use standard REST API HTTP codes to communicate success and errors: 
<https://learn.microsoft.com/en-us/rest/api/searchservice/http-status-codes>

Endpoints output the selected **Payment Terms** fields along with the **payment_terms_id**. 

**Create Payment Terms**

**HTTP Path:** /payment_terms

**HTTP Method:** POST

|**Property Name**|**Type**|**Required**|**Description**|
| - | - | - | - |
|**customer_id**|**String**|**Yes**|ID of the customer that can be accepted by **Payments Gateway**, **Notifications Gateway,** and **Salesforce Gateway**.|
|**total_amount**|**Int**|**Yes**|Total amount in cents owed for the whole duration of the term.|
|**start_date**|**Timestamp**|**Yes**|Timestamp of when payment term should start.|
|**term_duration**|**Int**|**Yes**|Duration of the payment term in months.|
|**billing_day**|**Int**|**Yes**|Day of the month when payment method should be charged.|
|**cancellation_terms**|**Object**|**No**|Defines cancellation terms.|
|**payment_terms**|**Object**|**No**|Defines payment terms, for the possibility to extend the system and handle more complex scenarios.|

**Adjust Payment Terms**

**HTTP Path:** /payment_terms/{payment_terms_id}

**HTTP Method:** PATCH

|**Property Name**|**Type**|**Required**|**Description**|
| - | - | - | - |
|**total_amount**|**Int**|**Yes**|Total amount in cents owed for the whole duration of the term.|

**Cancel Payment Terms**

**HTTP Path:** /payment_terms/{payment_terms}

**HTTP Method:** DELETE

## Possible Implementation Risks

Cloud Security Alliance lists the following most critical risks for serverless applications: <https://cloudsecurityalliance.org/blog/2019/02/11/critical-risks-serverless-applications/>

   - SAS-1: Function Event Data Injection
   - SAS-2: Broken Authentication
   - SAS-3: Insecure Serverless Deployment Configuration
   - SAS-4: Over-Privileged Function Permissions & Roles
   - SAS-5: Inadequate Function Monitoring and Logging
   - SAS-6: Insecure Third-Party Dependencies
   - SAS-7: Insecure Application Secrets Storage
   - SAS-8: Denial of Service & Financial Resource Exhaustion
   - SAS-9: Serverless Business Logic Manipulation
   - SAS-10: Improper Exception Handling and Verbose Error Messages
   - SAS-11: Obsolete Functions, Cloud Resources and Event Triggers
   - SAS-12: Cross-Execution Data Persistency

**Other implementation risks include:**

   - Complications discovered while implementing the solution.
   - Change of the requirements.
   - Extra time to learn new technologies.
   - Low engagement of stakeholders which might prevent the team from getting feedback earlier and taking the wrong direction.
   - Insufficient staff for the project deadline.

## Recommended Implementation Phases

**Implementation of the micro-frontends:**

   1. Collect requirements for the micro-frontends and create prototypes that will work against the mocked backend.
   2. Demo micro-frontends to Stakeholders for early feedback.
   3. Implement adjustments requested by stakeholders.
   4. Set up CloudFront + WAF with S3 to serve micro-frontends.
   5. Create a DevOps pipeline (scripts) that will build micro-frontends and publish them to S3, so they can be accessed.

**Implementation of the database with the event bus:**

   1. Set up **MongoDB** with the collections.
   2. Implement trigger handlers with the unit tests.
   3. Create a DevOps pipeline (scripts) for trigger handlers.
   4. Set up **Event Bus** (EventBridge) and hook it to **MongoDB**.
   5. Create an integration test that will verify that database changes are communicated to the Event Bus.

**Implementation of the transactions creator:**

   1. Set up **Payment Terms Queue** (SQS).
   2. Implement **Payment Terms Loader** with the unit tests.
   3. Implement **Transactions Creator** with the unit tests.
   4. Set up **CRON (EventBridge)** to call **Payment Terms Loader** on a schedule.
   5. Create a DevOps pipeline for the **Payment Terms Loader** and **Transactions Creator**.

**Implementation of the public backend APIs:**

   1. Set up API Gateway and Amazon Cognito.
   2. Create publicly facing backend API handlers with the unit tests.
   3. Connect micro-frontends with the public backend API and create end-to-end tests.

**Implementation of the payment collection:**

   1. Implement **Payments Collector** with the unit tests.
   2. Implement **Transactions Updater** with the unit tests.
   3. Set up **Payments Collector Queue** (SQS)
   4. Hook all the components of this subsystem.
   5. Create a DevOps pipeline for the **Payments Collector** and **Transactions Updater**.

**Implementation of the notifications subsystem:**

   1. Implement **Failed Payments Notifier** with the unit tests.
   2. Implement **PDF Receipt Sender** with the unit tests.
   3. Implement **Payment Terms Notifier** with the unit tests.
   4. Implement **Salesforce Record Creator** with the unit tests.
   5. Hook all the components of this subsystem to **Event Bus**.
   6. Create a DevOps pipeline for the **Payments Collector** and **Transactions Updater**.

**Implementation of the monitoring:**

   1. Set up **Dead Letter Queue.**
   2. Decide on monitoring the system and hook it.

**Testing of the system:**

   1. Verify the functionality of the entire system.
   3. Develop any extra end-to-end and integration tests.
   4. Instruct the QA team on the best testing strategies for that system.

**Launch:**

   1. Demo the system to stakeholders.
   2. Collect feedback and do necessary adjustments.
