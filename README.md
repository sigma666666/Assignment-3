# CSC4160 Assignment 3: Model Serving with AWS Lambda and Cold Start Performance Analysis (6 points)

### Deadline: October 24, 2024, 23:59

### Name: Boqi Wang

### Student ID: 122090511

---

## Overview

In this assignment, you will learn to deploy a machine learning model as a serverless application using AWS Lambda and API Gateway. You will create Docker images, push them to Amazon Elastic Container Registry (ECR), and conduct load testing on your deployed application. Additionally, you will analyze the cold start phenomenon associated with serverless functions.

We will use the well-known IRIS dataset to keep the machine learning model simple and focused on the serverless deployment process. The dataset includes four features: sepal length, sepal width, petal length, and petal width, and classifies samples into three categories: Iris Setosa, Iris Versicolour, and Iris Virginica.

![](./assets/architecture.png)

### Components

1. **Lambda Function Development**

   - Implement the `lambda_handler` function.

2. **Environment Setup**

   - Set up your local development environment.

3. **Docker Image Creation**

   - Make a Docker Image that will generate prediction using a trained model.

4. **ECR Repository Setup**

   - Create an AWS ECR repository and push your Docker image to AWS ECR.

5. **Lambda Function Creation in AWS Console**

   - Create a Lambda function using the container image.

6. **API Gateway Configuration**

   - Using the API gateway to access the prediction API

7. **Load Testing and Analysis**

   - Use Locust to perform load testing on your deployed API.
   - Plot the results to observe the cold start trend.
   - Analyze the differences between cold start and warm request response times.

## Instructions

### 1. Lambda Function Development

You will be provided with the `predict` function and the model file; your task is to implement the `lambda_handler` function.

The lambda_handler function performs the following tasks:

- Extracts the `values`: It retrieves the values input from the incoming event, which are the features used for making predictions.
- Calls the predict function: It invokes the predict function, passing the extracted values to generate predictions based on the machine learning model.
- Return the prediction result: Finally, it formats the prediction results as a JSON response and returns them to the caller.

<details>
  <summary>Steps to Implement <code>lambda_handler</code></summary>

#### Extract Input from Event:

- You will receive the input features inside the `body` of the event.
- Parse this `body` as JSON and retrieve the `values`.
- You could also handle any possible errors, like missing input or invalid JSON.

#### Call the `predict` Function:

- After extracting the `values`, pass them to the `predict` function, which will return a list of predictions.

#### Format and Return the Response:

- Return the predictions as a JSON response.
</details>

<details>
   <summary>Testing the function</code></summary>

#### Test with Mock Input:

You can simulate the input to the `lambda_handler` via the AWS Lambda console. For example, an event might look like this:

```bash
{
  "body": "{\"values\": [[5.1, 3.5, 1.4, 0.2]]}"
}
```

#### Simulate predict:

If you want to test without uploading the model, you can temporarily simulate the predict function to return a mock result.

#### Test in AWS Lambda:

Use the AWS Lambda Console to test your function with a sample event, or you can set up API Gateway and send a request from there.

</details>

### 2. Environment Setup

Set up your local development environment on your machine:

- Install Docker Desktop for your operating system: https://www.docker.com/
- Install the AWS CLI: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
- Ensure you have Python 3 and pip installed.
- (Optional but Recommended) Install Git: https://git-scm.com/downloads
- Configure your AWS credentials:

  <details>
  <summary>AWS credentials configuration</summary>

  #### To configure your AWS credentials, follow these steps:

  1. **Access your AWS credentials**: On the Vocareum main page, navigate to "Account details" then "AWS CLI." Copy the provided Access Key ID, Secret Access Key, and Session Token.

  2. **Create or open the credentials file**: Locate your AWS credentials file:

     - **macOS**: `~/.aws/credentials`
     - **Windows**: `C:\Users\%UserProfile%\.aws\credentials`

     If the file doesn't exist, create it using a plain text editor.

  3. **Add your credentials**: Paste the Access Key ID, Secret Access Key, and Session Token into the file, using the following format. Add the `region` line (you can use any region, e.g., `us-east-1`):

     ```ini
     [default]
     region=us-east-1  # Add this line.
     aws_access_key_id=YOUR_ACCESS_KEY_ID
     aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
     aws_session_token=YOUR_SESSION_TOKEN
     ```

     Replace `YOUR_ACCESS_KEY_ID`, `YOUR_SECRET_ACCESS_KEY`, and `YOUR_SESSION_TOKEN` with the values you copied from Vocareum.

  4. **Save the file**: Ensure the file is saved, and only you have access to it.

  5. **Important Security Note**: Never share your AWS credentials. Treat them like passwords. Do not commit this file to version control (e.g., Git). Add `.aws/credentials` to your `.gitignore` file. Consider using a more secure method of managing credentials in production environments.

  </details>

### 3. Docker Image Creation

Before building the Docker image, ensure the Docker daemon is running (start Docker Desktop on Windows/macOS or use `sudo systemctl start docker` on Linux).

In your local machine:

- Use the provided Dockerfile to create a Docker image:

  ```bash
  docker build -t iris_image .
  ```

- Run the Docker container locally:

  ```bash
  docker run -it --rm -p 8080:8080 iris_image:latest
  ```

  Here, we are mapping port 8080.

- Verify if the image is functioning properly by executing `test.py`.

### 4. ECR Repository Setup

Begin by launching your AWS Academy Learner Lab and ensuring your AWS credentials are correctly configured. Then, on your local computer, proceed with the following steps.

- Create an ECR repository:
  ```bash
  aws ecr create-repository --repository-name iris-registry
  ```
- Authenticate your Docker client with ECR:
  ```bash
  aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com
  ```
- Get image id:
  ```bash
  docker image ls
  ```
- Tag and push your Docker image:

  ```bash
  docker tag <image_id> <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/iris-registry:latest

  docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/iris-registry:latest
  ```

### 5. Lambda Function Creation

- In AWS console, create the Lambda function using the existing container image you have built and select `LabRole` as the execution role.

### 6. API Gateway Configuration

- Create an REST API for your Lambda function using API Gateway via AWS console.
- Test your API in your local machine using `curl` (Linux):

  ```bash
  curl --header "Content-Type: application/json" --request POST --data "{\"values\": [[<value1>, <value2>, <value3>, <value4>]]}" https://<your_api_id>.execute-api.<region>.amazonaws.com/default/<your_lambda_function>

  ```

  or using `Invoke-WebRequest` (Windows):

  ```bash
  Invoke-WebRequest -Method Post -Uri "https://<your_api_id>.execute-api.<region>.amazonaws.com/default/<your_lambda_function>" `
   -Headers @{ "Content-Type" = "application/json" } `
   -Body '{"values": [[<value1>, <value2>, <value3>, <value4>]]}'
  ```

### 7. Load Testing and Analysis

#### Load Testing

In your local machine, use the provided Locust load test script to evaluate the performance of your deployed API.

- Install Locust

```bash
pip install locust
```

- Navigate to the directory containing `locustfile.py`.
- Run the Locust test using:

```bash
locust -f locustfile.py --host https://<your_api_gateway_id>.execute-api.us-east-1.amazonaws.com --users 10 --spawn-rate 5 --run-time 60s --csv "locust_logs/test" --csv-full-history --html "locust_logs/test_locust_report.html" --logfile "locust_logs/test_locust_logs.txt" --headless
```

For Windows users, set the PATH for `locust`, or directly use the `locust.exe`, specifying its path, e.g.:

```bash
c:\users\user\appdata\roaming\python\python39\scripts\locust.exe -f locustfile.py --host https://<your_api_gateway_id>.execute-api.us-east-1.amazonaws.com --users 10 --spawn-rate 5 --run-time 60s --csv "locust_logs/test" --csv-full-history --html "locust_logs/test_locust_report.html" --logfile "locust_logs/test_locust_logs.txt" --headless
```

#### Analysis

Analyze the results using the performance analysis notebook on Google Colab. Upload your logs and run the notebook `performance_analysis.ipynb`. Fill in the estimated cold start time (in `<FILL IN>`) before graphing the histogram to compare response times during cold start and warm requests.

You will receive 1 point for including the required figures in your `.ipynb`: a line graph, a histogram of cold starts, and a histogram of warm requests. Additionally, 0.5 points will be awarded for providing a clear explanation of your analysis.

## Questions

### Understanding AWS Lambda, API Gateway, and ECR

1. **AWS Lambda Function** (0.5 point):

   What is the role of a Lambda function in serverless deployment? How does the `lambda_handler` function work to process requests?

   **Answer**:

   （1）A Lambda function in serverless deployment is a piece of code that runs in response to events, such as HTTP requests, database actions, or file uploads. It is executed without the need for provisioning or managing servers. The cloud provider runs the code in a highly available compute environment and scales automatically.

   （2）The `lambda_handler` function is the core of a serverless application that processes requests using AWS Lambda. It begins by loading a machine learning model with `pickle.load`. When invoked, it receives an `event` object containing the request data and a `context` object providing runtime information.
The function extracts input values from the event's JSON body and passes them to the `predict` function, which utilizes the loaded model to generate predictions. These predictions are then packaged into a JSON response with a status code of 200.
Error handling is robust, with specific responses for missing fields (400 status code), invalid JSON (400 status code), and other exceptions (500 status code). The function returns a response object, which AWS Lambda then converts into an HTTP response or another format as required by the invocation event.
In summary, `lambda_handler` serves as the interface between incoming requests and the machine learning model, managing data flow, error handling, and response formatting for a serverless prediction service.


3. **API Gateway and Lambda Integration** (0.5 point):

   Explain the purpose of API Gateway in this deployment process. How does it route requests to the Lambda function?

   **Answer**:

   API Gateway in the deployment process acts as the entry point for all client requests, providing a secure, scalable, and efficient way to manage and route these requests to the appropriate backend service, such as a Lambda function. It receives HTTP requests from clients, directs them to the correct Lambda function based on the configured API endpoints and HTTP methods, and handles aspects like security, monitoring, and throttling to ensure the backend services remain available and responsive under varying loads. This integration with Lambda allows for a seamless flow of data between the client and serverless compute resources, facilitating a robust serverless architecture.


5. **ECR Role** (0.5 point):

   What is the role of ECR in this deployment? How does it integrate with Lambda for managing containerized applications?

   **Answer**:
   
   (1)Role of ECR in Deployment:
   Amazon ECR serves as a repository to store Docker container images for the Lambda function. It allows you to upload,        store, and manage these images, which include the application code and any necessary dependencies, on AWS.

   (2)Integration with Lambda:
   AWS Lambda integrates with ECR by enabling the creation of Lambda functions directly from the container images stored in    ECR. This setup allows for a serverless deployment where the Lambda function runs within a container, providing a           consistent environment and simplifying the management of applications with specific runtime needs.


### Analysis of Cold Start Phenomenon

4. **Cold Start Versus Warm Requests** (1 Point):

   Provide your analysis comparing the performance of requests during cold starts versus warm requests (based on the line graph and histograms you obtained in `performance_analysis.ipynb`). Discuss the differences in response times and any notable patterns observed during your load testing.

   **Answer**:![image](https://github.com/user-attachments/assets/4b0265c8-7c8a-4bdd-be5d-0a4c536b4eb1)
   ![image](https://github.com/user-attachments/assets/554c2cb2-b59b-4f17-ae57-8ecbfd38f251)



(1)Overall Performance Pattern (Line Graph):
- There's a clear distinction between the initial cold start period and subsequent warm requests
- Initial cold starts show significantly higher response times:
  * p50 (median): ~2800ms
  * p95: ~3000ms
  * max: ~3100ms
- After warming up, the system stabilizes dramatically:
  * Response times drop to ~250-300ms
  * The three metrics (p50, p95, max) converge and remain stable
  * Very little variance in performance after warmup

(2)Cold Start Distribution (Upper Histogram):
- Highly bimodal distribution
- Main cluster of requests around 250-500ms
- Smaller cluster of cold starts between 2500-3000ms
- Clear separation between cold and warm performance
- Majority of requests (~115) fall in the lower response time range
- Small number of requests (~2-3) in the cold start range
- 5th percentile and median are close together, indicating consistency in warm performance
- 95th percentile is significantly higher due to cold starts

(3)Warm Requests Distribution (Lower Histogram):
- Much tighter distribution
- Response times clustered in two main ranges:
  * Group around 240-250ms
  * Group around 300-310ms
- Very small variance within each cluster
- Mean response time ~275ms
- 5th percentile: ~245ms
- 95th percentile: ~305ms
- Difference between p95 and p5 is only about 60ms, showing consistent performance

Key Findings:
(1)Cold starts incur a significant penalty, with response times roughly 10x higher than warm requests
(2)Once warmed up, the system maintains very stable performance
(3)The transition from cold to warm state is sharp rather than gradual
(4)Warm requests show predictable bi-modal behavior, possibly indicating different processing paths or resource allocation patterns
(5)The system demonstrates good reliability after warmup, with minimal performance variability



6. **Implications and Strategies** (0.5 Point):

   Discuss the implications of cold starts on serverless applications and how they affect performance. What strategies can be employed to mitigate these effects?

   **Answer**:

(1) Performance Impact
Cold starts show response times of ~3000ms compared to ~250ms for warm requests, as seen in our data. This 10x delay significantly impacts user experience, particularly for interactive applications and those with strict SLA requirements.

(2) Pre-warming Strategy
Implement scheduled function invocations to keep instances warm. While this increases costs, it provides more predictable performance for critical applications. The warming frequency should be based on traffic patterns and performance requirements.

(3) Code Optimization
Move initialization code and dependency loading to the module level instead of request handling. Use lightweight frameworks and implement lazy loading for resources. This reduces the initialization overhead during cold starts.

(4) Resource Management
Implement connection pooling for databases and maintain cached resources where possible. This prevents the need to re-establish connections or recreate resources during each cold start.

(5) Hybrid Architecture
For latency-sensitive operations, consider a hybrid approach using both serverless and traditional servers. Critical paths can run on dedicated instances while variable workloads use serverless functions.




## Submission Requirements

Please submit your assignment via BlackBoard in one `.zip` file, including the following files:

- `README.md` (with all questions answered) (3 points)
- `lambda_function.py` (your implementation of the Lambda function) (1 point)
  - Extracts the `values`
  - Calls the predict function
  - Return the prediction result
- Provide the CloudWatch log event for one of your Lambda function invocations. (0.5 point)
- A screenshot of a successful request and response using `curl` or `Invoke-WebRequest`, showing the correct prediction output. (0.5 point)
- `performance_analysis.ipynb`, including:
  - Figure of the line graph, cold start histogram, and warm request histogram. (0.5 point)
- `test_locust_logs.txt` (Locust load test logs) (0.5 point)

## Author

This assignment was created by Juan Albert Wibowo on October 8, 2024, for CSC4160: _Cloud Computing_, Fall 2024 at CUHKSZ.
