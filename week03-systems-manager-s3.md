Week 3: Systems Manager and S3

HarborTech Ticket Summary

Bright Path Community Services submitted ticket TKT-2026-0003 to HarborTech Solutions. The client has five EC2 instances that require the same administrative maintenance every Monday. The current process requires staff to log into each server and repeat the same task, which takes about 90 minutes. Bright Path also needs a simple public information page that does not require a separate web server.

The investigation focused on determining which AWS Systems Manager capability could reduce the repeated manual administration and whether Amazon S3 could provide the simple static information page.

Client Impact

Repeated manual administration across five EC2 instances can take significant time and create opportunities for inconsistent results. When the same task is performed manually on multiple systems, an administrator could miss an instance, enter a command incorrectly, or complete the task differently on different servers.

This can increase support work because inconsistent maintenance can lead to additional troubleshooting. Centralizing approved maintenance tasks can help reduce repetitive work and provide a more consistent process across the five systems.

AWS Services Involved

The main AWS services and concepts reviewed during the investigation were:

AWS Systems Manager: Provides centralized management capabilities for AWS resources and managed nodes.

Run Command: Allows administrators to remotely run commands on multiple managed nodes.

Session Manager: Provides interactive shell access to managed nodes without requiring direct SSH access.

Inventory: Collects information about managed nodes, such as operating system and installed applications.

Parameter Store: Provides centralized storage for configuration values and parameters.

Amazon S3: Provides object storage that can store files such as the Bright Path website.

Static Website Hosting: Allows an S3 bucket to serve static website content without requiring a traditional web server.

Virtualization Connection

The five EC2 instances are virtual machines that require regular administration. Systems Manager provides a centralized management layer that allows administrators to manage multiple virtual machines without individually connecting to every instance for routine tasks.

Run Command is useful when the same approved command needs to be executed across multiple managed instances. Session Manager is more appropriate when an administrator needs interactive access to troubleshoot or perform a task manually.

Amazon S3 provides a different service model because the Bright Path information page consists of static content. The page does not require a traditional server to generate dynamic content. S3 can store and serve static files such as HTML, which can reduce the need to maintain another server for a simple information page.

Evidence Reviewed

The investigation showed that Bright Path has five EC2 instances that require the same maintenance task every Monday. The current process involves manually logging into each instance and repeating the administrative work.

The Systems Manager features reviewed were Run Command, Session Manager, Inventory, and Parameter Store. Run Command was considered for centralized execution of the repeated maintenance task. Session Manager was considered for interactive access when troubleshooting or when a task requires administrator interaction. Inventory can provide information about the managed nodes and their installed software. Parameter Store can be used when the maintenance process requires centralized configuration values or parameters.

Before Systems Manager can manage the five EC2 instances, they need to be available as managed nodes. The SSM Agent must be installed and running, the instances need an appropriate IAM instance profile with the required Systems Manager permissions, and the instances need network connectivity to the required Systems Manager endpoints.

For the S3 portion, I created the bucket brightpath-ba-05-7392 in the us-west-2 Region. The object key was index.html. Static website hosting was enabled with index.html configured as the index document.

The website endpoint was:

http://brightpath-ba-05-7392.s3-website-us-west-2.amazonaws.com

I tested the endpoint using CloudShell. The request returned HTTP/1.1 403 Forbidden with an AccessDenied error. This showed that public access was restricted by the Learner Lab environment. I documented the restriction instead of attempting to bypass it.

I also used the AWS CLI to synchronize the local website directory with S3:

aws s3 sync ./brightpath-site s3://brightpath-ba-05-7392/

The command successfully uploaded the file:

upload: brightpath-site/index.html to s3://brightpath-ba-05-7392/index.html

The object was then verified with:

aws s3 ls s3://brightpath-ba-05-7392/

The output showed:

2026-09-23 04:01:53        547 index.html

Operational Analysis

The repeated Monday maintenance task is better suited for centralized management because the same administrative work must be performed across five EC2 instances. Manually logging into each system creates unnecessary repetition and increases the chance of inconsistent results.

Run Command is the appropriate Systems Manager capability for this task because it allows an approved command to be executed on multiple managed nodes. Session Manager is better suited for interactive troubleshooting or situations where an administrator needs to work directly with an individual system.

Inventory is useful for collecting information about the managed nodes, while Parameter Store can support maintenance tasks that require centrally managed configuration values. These features can support the overall management process but do not replace Run Command for the main repeated task.

The Bright Path information page is better suited to static object hosting because it only requires HTML content and does not require server-side processing. Amazon S3 can store the static website files without requiring Bright Path to maintain an additional EC2 web server.

Recommendation

For the five EC2 instances, HarborTech should use AWS Systems Manager Run Command to centralize the repeated Monday maintenance task. This fits the workload because the same approved administrative task needs to be performed across multiple instances.

Before using Run Command, HarborTech should confirm that all five EC2 instances are registered as Systems Manager managed nodes. The SSM Agent must be installed and running, each instance must have the required IAM instance profile permissions, and the instances must have network connectivity to the necessary Systems Manager endpoints.

For the Bright Path information page, HarborTech should use Amazon S3 with static website hosting because the page consists of static HTML content and does not require a traditional application server.

Session Manager should be retained for interactive administrative or troubleshooting needs. Inventory can be used to collect system information, and Parameter Store can be used when centralized configuration values are required.

Escalation Notes

The S3 website endpoint returned HTTP/1.1 403 Forbidden with an AccessDenied message during testing. This indicates that the Learner Lab environment restricted the public-access configuration needed for public website access.

No attempt was made to bypass the Learner Lab restriction. In a normal AWS environment, additional approval or configuration would be required to allow the appropriate public-read access for the S3 website. The applicable S3 Block Public Access settings and bucket permissions would need to be reviewed.

For Systems Manager, HarborTech should also confirm the required IAM instance profile, SSM Agent status, network connectivity, and administrator permissions before attempting to manage the five EC2 instances.

Lessons Learned

Week 3 showed me how centralized management can reduce repetitive work across multiple systems. Instead of manually logging into five EC2 instances, Systems Manager can provide a central way to perform approved administrative tasks.

I also learned that different Systems Manager features are designed for different situations. Run Command is useful for centralized non-interactive commands, while Session Manager is useful when interactive access is needed. Inventory and Parameter Store can provide supporting information and configuration management.

The S3 portion showed me that not every website requires a traditional server. Static content can be stored as objects in S3 and served through static website hosting. I also learned the importance of working within environment permissions and documenting a restriction instead of attempting to bypass a security control.

Professional Vocabulary

Systems Manager: An AWS service that provides centralized tools for managing and operating systems and applications.

Managed Node: A machine that has been configured so AWS Systems Manager can manage and communicate with it.

Run Command: A Systems Manager feature used to remotely execute commands on one or more managed nodes.

Session Manager: A Systems Manager feature that provides interactive access to managed nodes without requiring traditional SSH or direct remote access.

Inventory: A Systems Manager capability that collects information about managed nodes, including system and application details.

Parameter Store: A Systems Manager capability for centrally storing configuration values and other parameters that applications or administrative tasks can use.

Automation: The use of tools or processes to perform tasks with less manual intervention and more consistent results.

Static Website Hosting: A method of hosting web pages made from static files such as HTML without requiring a server to generate the content dynamically.

Object Storage: A storage model that stores data as objects along with their associated metadata. Amazon S3 uses object storage.

Management Plane: The systems and services used to configure, control, monitor, and manage computing resources rather than directly providing the workload itself.
