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
   AWS Lambda integrates with ECR by enabling the creation of Lambda functions directly from the container images stored in    ECR. This setup allows for a serverless deployment where the Lambda function runs within a container, providing a consistent environment and simplifying the management of applications with specific runtime needs.


### Analysis of Cold Start Phenomenon

4. **Cold Start Versus Warm Requests** (1 Point):

   Provide your analysis comparing the performance of requests during cold starts versus warm requests (based on the line graph and histograms you obtained in `performance_analysis.ipynb`). Discuss the differences in response times and any notable patterns observed during your load testing.

   **Answer**:
<img width="670" alt="0b2dfeb15c8d9a71153b8063ba806b4" src="https://github.com/user-attachments/assets/447a4ff3-2cb2-466a-8ffd-d938e0feab3e">
<img width="669" alt="85ae61bd913657a921d5508e8c880f5" src="https://github.com/user-attachments/assets/eaecd5e8-a7a7-42c0-972d-9ebf9466da03">


- **Cold Start Spike in Response Times:**
  - The line graph indicates a significant increase in response times for the p50, p95, and max values during cold starts.
  - This spike suggests that the system requires more time to initialize and process requests when it is starting up or has been idle.
- **Stable Response Times During Warm Requests:**
  - In contrast to cold starts, response times during warm requests are relatively stable with no notable spikes.
  - The system exhibits consistent performance once it has been running and is actively handling requests.
- **Histogram Analysis:**
  - The histograms provide further evidence of the performance difference between cold starts and warm requests.
  - A higher number of requests have lower response times during warm requests, indicating better performance.
  - During cold starts, there is a clear cluster of requests with higher response times, reflecting the initial performance penalty.
- **Performance Consistency:**
  - The data overall suggests that the system performs more consistently during warm requests.
  - After the initial cold start, the system's response times become predictable and maintain a steady state.
- **Cold Start Performance Penalty:**
  - Cold starts incur a significant performance penalty, with response times much higher compared to warm requests.
  - This penalty is a critical factor to consider in systems where rapid response times are crucial.
- **Warm Request Efficiency:**
  - Once the system is in a warm state, it handles requests more efficiently, with reduced response times and less variability.
  - The efficiency during warm requests is a positive indicator of the system's operational readiness after the initial warm-up phase.




6. **Implications and Strategies** (0.5 Point):

   Discuss the implications of cold starts on serverless applications and how they affect performance. What strategies can be employed to mitigate these effects?

   **Answer**:

Cold starts in serverless applications occur when a serverless function (like an AWS Lambda function) is invoked after being idle for a period. The cloud provider must allocate resources and initialize the runtime environment, which introduces latency that can negatively affect performance. Here are some implications and strategies to mitigate cold starts:

### Implications of Cold Starts

(1) **Increased Latency**: Cold starts can lead to noticeable delays in response times, especially for user-facing applications where speed is critical.

(2) **User Experience**: Applications that are sensitive to latency may frustrate users, potentially leading to higher abandonment rates.

(3) **Performance Variability**: Cold starts can introduce variability in performance metrics, complicating the monitoring and optimization of applications.

(4) **Cost Considerations**: While serverless architectures can be cost-effective, the latency from cold starts may require developers to architect solutions that can lead to increased costs (e.g., more frequent invocations).

### Mitigation Strategies

(1) **Provisioned Concurrency**: AWS Lambda offers provisioned concurrency, which keeps a specified number of function instances warm and ready to handle requests. This reduces the chance of cold starts.

(2) **Regular Invocations**: Implement a scheduling mechanism (e.g., using AWS CloudWatch Events) to invoke functions periodically to keep them warm. This can help minimize cold start occurrences.

(3) **Minimize Initialization Code**: Reduce the amount of code executed during the initialization phase of your functions. This can involve optimizing dependencies, lazy loading, or minimizing resource allocations.

(4) **Use Smaller Functions**: Break down larger functions into smaller, more focused functions. Smaller functions typically have shorter initialization times.

(5) **Runtime Selection**: Choose runtimes that have lower cold start times. Some runtimes, like Node.js, typically exhibit faster cold start performance compared to others, such as Java or .NET.

(6) **Optimize Dependencies**: Use only the necessary libraries and dependencies to minimize initialization overhead. Consider using lighter alternatives or compiling dependencies directly into your function.

(7) **Custom Runtimes**: If applicable, consider using custom runtimes optimized for cold start performance, which can reduce initialization time further.





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
