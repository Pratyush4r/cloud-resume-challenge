Cloud Resume Challenge - AWS Infrastructure
A serverless web application demonstrating cloud architecture, infrastructure as code, and full-stack integration on AWS.
Live Site: http://pratyush4r-cloud-resume.s3-website.eu-north-1.amazonaws.com/
What This Project Does
This is a resume website with a visitor counter that increments with each page load. Sounds simple, but it required building and connecting five different AWS services - demonstrating I understand how cloud infrastructure actually works, not just frontend design.
Architecture
Browser → S3 (static site) → API Gateway → Lambda (Python) → DynamoDB
Why this architecture?
This is "serverless" - I don't manage any servers. AWS scales automatically. If 1 person visits or 10,000 visit, it works the same. I only pay for actual usage (basically free for a resume site). This is how modern cloud apps are built.
Components & What They Do
1. S3 Bucket (Storage + Hosting)

What: Amazon S3 stores the HTML/CSS files and serves them as a website
Why S3: It's designed for static sites - cheap, fast, reliable. No server to maintain.
Configuration: Had to disable "Block Public Access" and add a bucket policy so anyone can view it. Also enabled "Static Website Hosting" to make it act like a web server.

2. DynamoDB (Database)

What: NoSQL database storing the visitor count
Table: cloud-resume-visit-counter
Schema: One item with id: "visitor-count" and count: 0 (increments with each visit)
Why DynamoDB: Serverless database - no server to manage, scales automatically, pay-per-request pricing. Perfect for simple data like a counter.

3. Lambda Function (Backend Logic)

What: Python function that runs when the API is called
What it does:

Reads current count from DynamoDB
Increments it by 1
Saves new count back to DynamoDB
Returns count as JSON


Why Lambda: "Function as a Service" - I write code, AWS runs it. No servers. Only charged when someone actually visits the site (milliseconds of compute time = fractions of a penny).
Key learning: DynamoDB returns numbers as Decimal type (Python library quirk), which JSON can't serialize. Had to convert to int first. This is the kind of real-world issue you don't learn from tutorials.

4. API Gateway (HTTP Endpoint)

What: Creates a public HTTPS URL that triggers the Lambda function
Endpoint: https://hdkleyqqj7.execute-api.eu-north-1.amazonaws.com/count
Why: Lambda functions aren't directly accessible from browsers. API Gateway sits in front, handles HTTPS, CORS headers, request routing.
Type: HTTP API (simpler and cheaper than REST API for this use case)

5. JavaScript (Frontend Integration)

What: Fetches the count from API Gateway when page loads
How: fetch() API call to the endpoint, parses JSON response, updates the DOM
CORS: API Gateway had to send Access-Control-Allow-Origin: * header so browsers allow the cross-origin request

Data Flow (What Happens When You Visit)

Browser requests http://pratyush4r-cloud-resume.s3-website.eu-north-1.amazonaws.com/
S3 returns index.html and style.css
Browser executes JavaScript in the HTML
JavaScript makes fetch() call to API Gateway endpoint
API Gateway triggers Lambda function
Lambda reads DynamoDB → increments count → writes back to DynamoDB
Lambda returns count as JSON: {"count": 42}
JavaScript receives response and updates <span id="visitor-count"> with the number
User sees the visitor count on the page

Problems I Solved
Problem 1: Table Name Mismatch

Error: Lambda couldn't find DynamoDB table
Cause: Created table as cloud-resume-visit-counter but code referenced cloud-resume-counter
Fix: Updated Lambda code to match actual table name
Learning: AWS error messages often say "resource not found" - could be wrong name, wrong region, wrong permissions. Always verify the basics first.

Problem 2: Decimal Serialization

Error: Object of type Decimal is not JSON serializable
Cause: DynamoDB Python SDK returns numbers as Decimal objects (for precision), but Python's json.dumps() doesn't handle them
Fix: Convert to int before serializing: count = int(count) + 1
Learning: Every AWS SDK has quirks. boto3 (Python) uses Decimal for DynamoDB numbers. You learn this by breaking things, not reading docs.

Problem 3: Public Access

Error: Bucket policy wouldn't save
Cause: Created bucket with "Block all public access" enabled
Fix: Disabled Block Public Access, then added bucket policy
Learning: AWS is secure by default - you have to explicitly allow public access. Good for security, annoying when you actually want something public.

What This Proves (For Interviews)

I can build on AWS - Not just theory. I provisioned real infrastructure and made it work.
I understand serverless architecture - The components, how they connect, why you'd use this pattern.
I can debug cloud issues - Region mismatches, permission errors, SDK quirks - these are real problems DevOps engineers solve daily.
I can integrate frontend and backend - CORS, API calls, error handling, async JavaScript.
I can ship - This isn't a half-finished tutorial. It's live, working, on the internet.

Tech Stack

Frontend: HTML, CSS, JavaScript (vanilla, no frameworks - keeping it simple)
Backend: Python 3.14 (Lambda)
Database: DynamoDB (NoSQL)
Infrastructure: AWS S3, Lambda, API Gateway, DynamoDB
Version Control: Git/GitHub

What's Next

 CI/CD pipeline (GitHub Actions auto-deploy to S3)
 Infrastructure as Code (Terraform or CloudFormation)
 Custom domain with Route 53
 HTTPS with CloudFront + ACM certificate
 Monitoring with CloudWatch
