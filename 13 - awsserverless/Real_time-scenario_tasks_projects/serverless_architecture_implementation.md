# Real Time Task on Step By Step Implementation of Serverless Architecture with DynamoDB, Api Gateway and Lambda.

## Step 1: Create a DynamoDB Table

1.  Log in to the AWS Management Console and navigate to the DynamoDB service.
2.  In the left-hand menu, click on Tables.
3.  Click the Create table button.
4.  Under Table settings:
    *   Table name: Enter `bookstore`.
    *   Partition key:
        *   Name: `id`
        *   Type: Select `Number` from the dropdown.
5.  Keep all other settings as default (for example, on-demand capacity, encryption, etc.).
6.  Click Create table.
7.  Wait until the table’s status becomes `Active`.

## Step 2: Create a Lambda Function

### 2.1 Create a Lambda Function Using a Blueprint

1.  In the AWS Management Console, navigate to Lambda.
2.  Click on Create function.
3.  Under Create function, select Use a blueprint.
4.  In the blueprint search field, type and select "Create a mobile backend that interacts with a DDB table".
    *   This blueprint is preconfigured to interact with DynamoDB.
5.  Under Basic information:
    *   Function name: Enter `testing1`.
    *   Runtime: (The blueprint will auto-select an appropriate runtime, for example, Python or Node.js.)
6.  Under Permissions, choose Create a new role with basic Lambda permissions.
7.  Click Create Function.

### 2.2 Configure and Test the Lambda Function

1.  After creation, click on the Code tab in your Lambda function’s details page.
2.  Configure a Test Event:
    *   Click on Test.
    *   In the pop-up, click on Configure test event.
    *   Select Create new event.
    *   Event name: Enter `testing001`.
    *   In the Event JSON section, paste sample JSON data for a book record. For example:

    ```json
    {
    "operation": "create",
    "data": {
    "id": 101,
    "title": "Serverless Essentials",
    "author": "AWS User",
    "price": 29.99
    }
    }
    ```
    *   Click Create.
3.  Attach a Policy for DynamoDB Access:
    *   Switch to the Configuration tab of the Lambda function.
    *   Under Permissions, click on the Role name link. This will open the IAM role associated with your function.
    *   In the IAM role page, click Attach policies.
    *   Search for `AmazonDynamoDBFullAccess`.
    *   Check the box next to it and click Attach policy.
        *   This gives the Lambda function full access to DynamoDB.
4.  Return to the Lambda function’s Code tab.
5.  Click Test (using the `testing001` event) to invoke the function.
6.  Open the DynamoDB Console, select your `bookstore` table, and click on Explore items.
    *   Verify that the test event data has been inserted into the table.
    *   (The blueprint’s code typically performs an operation like inserting the JSON data into the table.)

## Step 3: Test with API Gateway

### 3.1 Create a REST API in API Gateway

1.  In the AWS Management Console, navigate to API Gateway.
2.  Click on REST API.
3.  Choose New API.
4.  API Name: Enter `bookstore`.
5.  Click Create API.

### 3.2 Create a Resource in the API

1.  In the newly created API, click on Actions and then Create Resource.
2.  For Resource Name, enter `bookid`.
3.  For Resource Path, enter `/` (if you want the root resource, you can simply name it `bookid` for identification).
    *   Alternatively, you can create a subresource `/bookid`.
4.  Click Create Resource.

### 3.3 Create a PUT Method for the `/bookid` Resource

1.  With the `/bookid` resource selected, click on Actions and then Create Method.
2.  Choose the `PUT` method from the dropdown.
3.  Click the checkmark to confirm.
4.  In the PUT setup:
    *   Integration type: Select `Lambda Function`.
    *   Use Lambda Proxy integration: (Typically enabled; if not, you may need to configure mapping templates.)
    *   Lambda Region: Ensure it is set to the same region as your Lambda function.
    *   Lambda Function: Enter or select `testing1` (the Lambda function you created earlier).
5.  Click Save, and then acknowledge any prompts regarding Lambda permissions.
6.  After the method is created, click on Test within the PUT method configuration.
    *   In the test window, paste any random JSON data for a book record similar to:

    ```json
    {
    "operation": "create",
    "data": {
    "id": 102,
    "title": "AWS Serverless Cookbook",
    "author": "DevOps Guru",
    "price": 39.99
    }
    }
    ```
7.  Click Test and verify that the test invocation is successful.
8.  Switch to the DynamoDB Console, select your `bookstore` table, and confirm that the new data is visible in Explore items.

### 3.4 Deploy the API

1.  In API Gateway, click on Actions → Deploy API.
2.  In the Deployment stage dropdown, choose `[New Stage]`.
3.  Enter a Stage Name (e.g., `prod`).
4.  Click Deploy.
5.  Note the Invoke URL provided after deployment. This URL is now accessible publicly.

## Step 4: Final Verification

### 4.1 Using MySQL Workbench/Query Tools (Optional)

*   Although this step is more focused on API testing, you can also manually verify data by querying the DynamoDB table.

### 4.2 Verify the End-to-End Flow

1.  Invoke the API:
    *   Use a tool like Postman or curl to send a PUT request to your API's `/bookid` resource.
    *   Example using curl:

    ```bash
    curl -X PUT \
    -H "Content-Type: application/json" \
    -d '{
    "operation": "create",
    "data": {
    "id": 103,
    "title": "Serverless in Action",
    "author": "AWS Guru",
    "price": 49.99
    }
    }' \
    https://<your-api-id>.execute-api.<region>.amazonaws.com/prod/bookid
    ```

2.  Check DynamoDB:
    *   Go to the DynamoDB Console and open the `bookstore` table.
    *   Confirm that the new data (id 103) has been inserted.
3.  Test via Lambda Console:
    *   You may also run additional test events from the Lambda console to confirm the function’s behavior.

## What to Expect After Implementing This Project Task?

By completing this hands-on AWS Serverless Architecture project, you will have successfully built a fully functional, scalable, and cost-efficient backend solution. Here's what you can expect after implementation:

### Key Achievements:

*   **A Fully Serverless Backend**
    *   You will have built a completely serverless application using AWS services—Lambda, API Gateway, and DynamoDB—without provisioning or managing any servers.
    *   This architecture auto-scales based on traffic and runs only when needed, helping optimize cost and performance.

*   **DynamoDB as a Highly Available NoSQL Database**
    *   Your DynamoDB table (`bookstore`) will dynamically store, retrieve, and manage book records with low latency and high availability.
    *   It offers on-demand scaling and eliminates the need for database administration.

*   **AWS Lambda for Event-Driven Computing**
    *   Your Lambda function (`testing1`) will process API requests, execute backend logic, and interact with DynamoDB—all without running continuously.
    *   This results in cost efficiency, as you only pay for execution time.

*   **API Gateway for RESTful Endpoints**
    *   Your API Gateway instance (`bookstore` API) will act as the entry point for external applications, allowing them to interact with your DynamoDB table using simple HTTP methods like PUT, GET, DELETE, and POST.
    *   API Gateway ensures secure and managed API access with built-in logging and monitoring.

*   **End-to-End Integration & Testing**
    *   You will have tested the entire pipeline using Postman, cURL, and AWS Console to validate database interactions.
    *   You will have successfully inserted and retrieved book records in DynamoDB via API requests.

### Why This Project is Valuable?

*   **Real-World AWS Experience**
    *   This is a practical, industry-relevant AWS project that mirrors real-world scenarios used by companies for scalable, event-driven applications.
    *   Ideal for DevOps, cloud computing, and backend development roles.

*   **Cost Optimization**
    *   Unlike traditional server-based architectures, this setup ensures that you only pay for actual usage, making it highly cost-efficient.
    *   No need to manage infrastructure, reducing operational overhead.

*   **Highly Scalable & Secure**
    *   AWS auto-scales Lambda and DynamoDB based on demand.
    *   API Gateway provides authentication, rate limiting, and monitoring for enhanced security.

### What’s Next?

Now that you have successfully implemented the AWS Serverless Architecture, you can further extend and optimize the project:

### Enhancements & Advanced Features:

*   **Add More API Methods** – Implement GET, DELETE, and UPDATE endpoints for a complete CRUD (Create, Read, Update, Delete) API.
*   **Implement Authentication** – Secure your API with AWS Cognito for user authentication.
*   **Optimize Performance** – Use DynamoDB Streams to trigger Lambda functions on data changes.
*   **Monitor with AWS CloudWatch** – Set up logging, metrics, and alerts for better observability.
*   **Deploy with Infrastructure as Code** – Automate deployments using Terraform or AWS CloudFormation.

## Summary

*   **DynamoDB Table Creation:**
    *   A table named `bookstore` is created with a partition key `id` (Number).
*   **Lambda Function Setup:**
    *   A Lambda function (`testing1`) is created using a blueprint that interacts with DynamoDB.
    *   A test event is configured, and the function is granted full access to DynamoDB via an attached policy.
*   **API Gateway Configuration:**
    *   A new REST API (`bookstore`) is created.
    *   A resource `/bookid` is set up with a PUT method integrated with the Lambda function.
    *   The API is deployed to a stage (e.g., `prod`).
*   **End-to-End Testing:**
    *   Test data is sent via API Gateway, which triggers the Lambda function to insert data into DynamoDB.
    *   The inserted data is verified in the DynamoDB console.

## Wrapping Up: Mastering Serverless Architecture with AWS! 🚀

Congratulations on reaching the end of this comprehensive guide on AWS Serverless Architecture!

We’ve covered everything from theory to real-world implementation, ensuring that you now have a solid understanding of AWS Lambda, API Gateway, and DynamoDB in action.

*   You’ve learned how serverless architecture eliminates infrastructure management and enables scalability, cost-efficiency, and event-driven workflows.
*   You’ve built a fully functional API that seamlessly interacts with DynamoDB via AWS Lambda.
*   You’ve explored best practices for designing, deploying, and securing serverless applications.

### What’s Next?

AWS offers an entire ecosystem of serverless services, and we’ve only scratched the surface. As we move forward, I’ll be covering more advanced AWS services with real-time implementation projects.

Daily AWS Practical Tasks Incoming!
I’m committed to sharing AWS real-world projects daily, covering different services with hands-on implementation. Follow me to stay updated and keep learning!

More AWS practical implementations coming soon—stay tuned! 🔥 Let’s keep building and automating! ⚡

