# epam_AWS-specialization_final-Project
TXT file with my own personal approach to implement the final project.


Final Project
--------------------------------------------------------------------------------------------------------------------------------

WHAT?
----------

'User Story' Model:
As a library user, I want to submit book requests to the digital catalog so that librarians can review them and
potentially add them to the collection.

What we want to implement is a solution for that User Story with the following flow:

	- A user submits a request with book data.
	- The system receives that request.
	- The request is processed.
	- More information about the book is obtained from an external API.
	- Everything is stored in a database.
	- Librarians review the requests.
	- Monitoring from the user

Input → processing → fetch data → storage

We are going to implement it using a decoupled serverless architecture.

```
User → API Gateway → Producer Lambda → SQS Queue → Consumer Lambda → Book API → DynamoDB
                                         │
                                         └── DLQ (Dead Letter Queue)
```
-------------------------------------------------------------------------------------------------------------------------------

WHY?
-------

1. Why Agile? It fits in any approach.
   Whether using EC2 instances, containers, or serverless, we need agile practices:
	- Organization
	- Planning
	- Teamwork
	- Workflow
 	  Issue → Pull Request → Merge → Deploy

2. Why Serverless and decoupled?
 	- Occasional requests
 	- Each request is an event that triggers a small processing task
 	- There is no continuous processing
 	- Traffic can be very variable and unpredictable
	- SQS is asynchronous and improves fault tolerance (resilience).
 
 	Conclusion: this is an intermittent, event-driven workload. It fits with Lambda

 	- Serverless benefit: Time-To-Market and costs.
 		With a traditional infrastructure we would need to configure:
 		- VPC
 		- subnets
 		- load balancer
 		- EC2
 		- autoscaling
 		- deployment
 		- monitoring

 		With serverless, this is reduced to:
 		- API Gateway
 		- Lambda
 		- SQS
 		- DynamoDB

 		- Cost efficiency due to small workloads, compared to an EC2 instance running
		  all the time or container compute capacity. AWS Pricing calculator.

3. Why DynamoDB?
	- Serverless environment. Greater compatibility with Lambda
	- Pay-per-use consumption
	- Event-driven architecture
	- Autoscaling

4. Why IaC?
	- Reproducibility
	- Portability
	- Agility (Agile practices)
	- Versioning. CI/CD
	- Manual infrastructure is not always a best practice in the real world. Great for education purposes.

-------------------------------------------------------------------------------------------------------------------------------


HOW?
-------

Architecture:
```
User (POST /book-request) → API Gateway Triggers → Producer Lambda → SQS → Consumer Lambda → Book API → DynamoDB
   ↑                           		↓  ↑		           ↓
   └--------------<---------<------└-----<-----<-----└---- HTTP response (request accepted + request_id)
```

FLOW:

User's request (POST /book-request)
	↓
API Gateway triggers Producer Lambda
	↓
Producer Lambda validates input and sends SQS message*
	↓
Consumer Lambda reads SQS message, calls API, gets data.
	↓
Data gets stored en DynamoDB
	
Maybe for later??:
GET /book-request/{request_id}
To allow user to monitor request progress.


1. Agile
	- Create repository "book-request-system"
	- Create Issues for each phase of the project. For example:
	  * Create API endpoint
	  * Implement Producer Lambda
	  * Create SQS queue
	  * Implement Consumer Lambda
	  * Integrate external book API
	  * Create DynamoDB table
	  * Write Terraform IaC
	  * Configure IAM roles
	  * Add logging
	
	- GitHub Projects:
	  * Backlog - User Stories
	  * To Do
	  * In Progress
	  * Done

	- GitHub Actions/Gitlab CI. Create Pipeline:
	  push code
	     ↓
	  run tests
	     ↓
	  terraform plan (-out plan.out) - Keep a record of the plan.
	     ↓
	  terraform apply
	     ↓
	  deploy lambdas


2. IaC - Terraform. Define the entire AWS environment as code.
	- API Gateway
	- Producer Lambda
	- Consumer Lambda
	- SQS queue
		{
		  "event_type": "BOOK_REQUEST_CREATED",
		  "request_id": "b1d2-9933",
		  "title": "Clean Code",
		  "author": "Robert C. Martin",
		  "requested_at": "2026-03-10T18:00:00Z"
		}


	- Dead Letter Queue: We need to prevent infinite retry cycles.
		Consumer Lambda fails
		       ↓
		SQS retries
		       ↓
		max retry attempts
		       ↓
		message → DLQ

	- DynamoDB table:
		Table: book_requests
		PK: request_id
		- Fields:
		  request_id
		  title
		  author
		  isbn
		  publisher
		  year
		  description
		  requested_at
		  status

	- IAM roles: Lambda, DynamoDB, TF
		lambda-producer-role
		lambda-consumer-role
		terraform-deployment-role
	- CloudWatch logs

3. AWS Lambda code.
	- Producer: receive request, validate data, generate request_id, create message, send message to SQS.
	- Consumer: read message from SQS, call external book API, obtain data, store result in DynamoDB.

4. Integrate external API for data enrichment.
	- Search: author and title
	- Get: isbn, publisher, year, description

5. Observability. Add logs for monitoring and auditing. Each Lambda should log:
	- request_id
	- event
	- result
	- errors


By the way, Cloud Atlas by David Mitchell seems to be awesome!!
