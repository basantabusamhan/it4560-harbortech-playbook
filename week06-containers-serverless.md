# Week 6: Containers and Serverless Computing

## HarborTech Ticket Summary

The Week 6 HarborTech ticket focused on selecting the right compute model for a workload that runs for about 20 seconds once per day to generate a report. The main problem was determining whether the workload needed a continuously running server or whether a serverless solution would be more efficient. Since the task runs briefly and only once a day, maintaining a virtual machine or container running all the time could create unnecessary costs and management work. The goal was to find a solution that could run the report when needed without requiring constant infrastructure maintenance.

## Client Impact

Using the wrong compute model can affect HarborTech in several ways. A virtual machine or container that remains running when it is not needed can increase costs. Reliability can also become an issue if the application depends on manually maintained servers, updates, or troubleshooting. Scalability is another concern because a system designed for constant operation may not handle changes in workload efficiently. Finally, managing servers, operating systems, and software updates creates additional work for IT staff. Choosing a compute model that matches the workload can help reduce costs, improve reliability, and simplify daily operations.

## AWS Services Involved

The investigation involved several AWS services and concepts.

- **Containers:** Containers package an application and its dependencies together so it can run consistently across different environments.
- **Serverless Computing:** Serverless computing allows applications to run without requiring the customer to manage the underlying servers directly.
- **AWS Lambda:** Lambda runs code in response to events or requests. It is a good option for HarborTech's short, daily reporting task.
- **Amazon API Gateway:** API Gateway allows applications to expose APIs that receive requests and connect them to backend services such as Lambda.
- **AWS Step Functions:** Step Functions coordinates multiple tasks into a workflow and can manage sequencing, retries, and failures.
- **IAM Execution Roles:** Execution roles provide AWS services with the permissions they need to access other resources securely.
- **Event-Driven Computing:** Event-driven computing starts a process when an event occurs instead of requiring a program to run continuously.
- **VPC-Connected Lambda:** A Lambda function can connect to resources inside a Virtual Private Cloud when the application requires access to private network resources.
- **Monitoring and Logs:** Amazon CloudWatch can collect logs, metrics, and alarms to help identify errors and monitor application performance.

## Virtualization Connection

Traditional virtual machines provide virtualized hardware on which an operating system and applications can run. They offer control over the operating system but also require management of resources, updates, and configurations. Containers provide another level of abstraction by packaging applications and their dependencies while sharing the host operating system kernel. This makes containers generally more lightweight than full virtual machines.

Serverless computing provides an even higher level of abstraction because AWS manages the underlying infrastructure needed to execute the code. With Lambda, the customer focuses more on the function, its permissions, and its triggers instead of managing a server. Containers and serverless computing can both reduce infrastructure management, but they serve different needs. Containers are useful when applications need a consistent environment or more control over their runtime, while Lambda is useful for short, event-driven tasks that do not need a continuously running server.

## Evidence Reviewed

The investigation considered the HarborTech workload requirements, the AWS environment, and the results of the Lambda function testing.

- **Workload duration and frequency:** The reporting task runs for approximately 20 seconds and is needed once per day.
- **Current server utilization:** The ticket's short runtime and low frequency suggest that keeping a dedicated server running continuously may be unnecessary. A complete utilization analysis of an existing production server was not available.
- **Trigger or event pattern:** The workload is a good candidate for a scheduled daily trigger or another event that starts the reporting process when needed.
- **Runtime and dependencies:** A Python Lambda function was created and tested. The application used a handler that checked whether a required department value was provided.
- **IAM permissions:** The function was deployed using the preconfigured LabRole in the AWS Learner Lab. Any production deployment would require confirmation that the execution role has only the permissions necessary for the task.
- **API request behavior:** API Gateway was reviewed as a possible way to trigger the function through an API. The Lambda tests confirmed application behavior, but a complete API Gateway integration test was not documented.
- **Lambda logs and errors:** The function was invoked with valid and invalid test events. The valid event returned an application status code of 200, while the missing department value returned an application status code of 400 with an error message explaining that the department was required. The AWS CLI invocation itself returned a StatusCode of 200 in both cases because the function executed successfully.
- **VPC connectivity requirements:** The workload requirements did not establish that the function needed access to private VPC resources. This would need to be confirmed before configuring VPC connectivity.
- **Workflow steps and failure points:** The tested function checked for a required input before returning a successful response. More complex workflows could require Step Functions to coordinate multiple tasks and handle failures.
- **Monitoring evidence:** Lambda invocation results and function output provided basic evidence of application behavior. Production monitoring should also include CloudWatch logs, error metrics, and alarms.

## Operational Analysis

The evidence suggests that AWS Lambda is the best fit for HarborTech's daily reporting workload. The task runs for a short period and does not need a continuously running process. A dedicated virtual machine would require additional administration, while a container-based solution could introduce more infrastructure and configuration than this simple workload requires.

The Lambda test also demonstrated how input validation can prevent a function from processing an incomplete request. When the department value was provided, the function returned a successful application response. When the value was missing, it returned an application-level error. This showed that testing should include both successful and unsuccessful scenarios.

Amazon API Gateway would be appropriate if HarborTech needed other applications or users to request reports through an API. A scheduled event could trigger the function for daily reporting. AWS Step Functions would be more useful if the reporting process later expanded into multiple coordinated tasks. Based on the available evidence, a single Lambda function is sufficient for the current requirement, provided that its execution time, dependencies, and permissions meet the final production requirements.

## Recommendation

I recommend using AWS Lambda for HarborTech's daily reporting task. Since the workload runs for approximately 20 seconds once per day, Lambda can execute the code when needed without requiring HarborTech to maintain a dedicated server that stays running between executions.

A scheduled trigger could start the function once per day. The function should validate its inputs, use a properly configured IAM execution role, and send logs and monitoring data to CloudWatch. If HarborTech later needs users or other applications to request reports directly, API Gateway could provide the API endpoint. If the process grows to include several dependent tasks, Step Functions could coordinate the workflow.

This recommendation reduces direct infrastructure management while allowing HarborTech to retain responsibility for the application code, permissions, configuration, testing, and monitoring. Before production deployment, the team should verify the required permissions, application dependencies, error-handling behavior, and any private network access requirements.

## Escalation Notes

No immediate escalation was identified for the basic Lambda practice function tested in the AWS Learner Lab. The function was successfully invoked with both valid and invalid inputs, and the results showed that the required input validation worked as expected.

However, additional approval or support would be needed before making production changes. The production team should review the IAM execution role and approve any new permissions. Network support would be needed if the function must connect to private resources inside a VPC. Application support may be required if the actual reporting process needs additional libraries, external connections, or more complex error handling. Any API Gateway integration, scheduled production trigger, or workflow involving Step Functions should also be tested and approved before deployment.

## Lessons Learned

Week 6 helped me understand that choosing a compute model depends on how an application runs and what it needs. Virtual machines provide more control over the operating system, while containers package applications and their dependencies in a consistent environment. Serverless computing reduces the need to manage infrastructure directly and works well for tasks that run only when needed.

I also learned that Lambda can respond to events instead of running continuously. API Gateway can connect API requests to backend services, while Step Functions can coordinate more complicated workflows. IAM execution roles are important because they control what a function can access, and CloudWatch helps identify errors and monitor performance.

The Lambda testing exercise also showed me why it is important to test both valid and invalid inputs. A function can execute successfully at the AWS level while still returning an application-level error. Overall, the main lesson is that the best compute model is not always the most complex one. It is the one that meets the workload's requirements while balancing cost, reliability, scalability, security, and operational effort.

## Professional Vocabulary

**Container:** A package that includes an application and the dependencies it needs to run in a consistent environment.

**Serverless Computing:** A computing model in which the cloud provider manages the underlying infrastructure so developers can focus on their code and application logic.

**AWS Lambda:** An AWS service that runs code in response to events or requests without requiring the customer to manage a server.

**API Gateway:** An AWS service that creates and manages APIs so applications can communicate with backend services.

**Execution Role:** An IAM role that gives an AWS service, such as Lambda, permission to perform specific actions on other AWS resources.

**Event Trigger:** An event or condition that starts a function or process, such as a scheduled time, an API request, or a change to stored data.

**Step Functions:** An AWS service that coordinates multiple tasks into a workflow and helps manage their order, retries, and failures.

**VPC-Connected Lambda:** A Lambda function configured to access resources in a Virtual Private Cloud when private network connectivity is required.

**Runtime:** The environment that executes an application or function, including the language support needed to run its code.

**Abstraction:** The process of hiding lower-level technical details so users can focus on the functions and services they need instead of managing every underlying component.
