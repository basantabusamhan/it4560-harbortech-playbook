
# Week 7: Managed Database Services

## HarborTech Ticket Summary

HarborTech ticket TKT-2026-0007 concerns a café business running a transactional MySQL database on an Amazon EC2 instance. The database is working, but the client spends staff time on operating system maintenance, database patching, backups, storage management, and recovery planning.

The goal is to determine whether the client should keep its self-managed database on EC2 or evaluate a managed database service. No production migration has been approved.

## Client Impact

Database downtime, failed backups, or an unsuccessful migration could interrupt café operations and affect access to business information. Poor recovery planning could result in data loss or a longer service interruption. Choosing a database that does not fit the application's requirements could also create compatibility problems and unnecessary costs.

## AWS Services Involved

- **Amazon EC2:** Provides the virtual server where the current MySQL database runs.
- **Amazon RDS:** A managed relational database service and the first option to evaluate for the existing workload.
- **Amazon Aurora:** A managed relational database option to compare with standard RDS when compatibility, performance, availability, and cost justify it.
- **Amazon DynamoDB:** A managed NoSQL database that may fit applications using key-based access to independent records.
- **Multi-AZ:** A deployment option that can improve availability by maintaining database infrastructure across Availability Zones.
- **AWS Database Migration Service (AWS DMS):** A service that can support database migration and replication.
- **Schema conversion:** The process of adapting database schemas and database objects when moving between database engines.
- **Security Groups:** Virtual firewall rules that control network traffic to supported AWS resources.
- **Database connectivity:** The network path and permissions that allow an application to communicate with a database.

## Virtualization Connection

With MySQL on EC2, the physical hardware is abstracted through virtualization, but the client still manages the guest operating system and database software. The client is responsible for patching, maintenance, backups, and recovery planning.

Amazon RDS abstracts more of the infrastructure and automates many routine database management tasks. AWS manages the underlying infrastructure and the managed-service responsibilities defined by RDS, while the client remains responsible for data, application integration, access permissions, database design, and verifying recovery requirements.

## Evidence Reviewed

The client brief identifies the following workload characteristics:

- A transactional MySQL database.
- Structured tables and SQL queries.
- A need to preserve relational data and transaction behavior.
- Staff time spent on server and database maintenance.
- A need to review backups, recovery planning, and availability.
- Missing information about database size, traffic, recovery objectives, budget, and compatibility requirements.

AWS Academy observations from the earlier TKT-2026-0007 lab included an AWS session identity check and database inventory commands in us-east-1. These observations describe the lab environment, not the client's production database.

Additional evidence is needed about the current EC2 configuration, database version, storage, application connection settings, security group rules, backup history, and recovery process.

## AWS Academy Activity: Actions and Results

### Evidence Entry 1: Verify the AWS session

**Command:**

`aws sts get-caller-identity`

**Observed result:**

The command returned the AWS Academy assumed-role identity and account information.

**What it demonstrates:**

The AWS CLI session was connected to the Academy environment. The command verified the active session without displaying credentials.

### Evidence Entry 2: Check RDS inventory

**Command:**

`aws rds describe-db-instances --region us-east-1`

**Observed result:**

The command returned an empty `DBInstances` list.

**What it demonstrates:**

No RDS database instances were listed in us-east-1 during this check. This does not establish the configuration of the café's MySQL database on EC2 or the existence of resources in other regions.

### Evidence Entry 3: Check DynamoDB inventory

**Command:**

`aws dynamodb list-tables --region us-east-1`

**Observed result:**

The command returned an empty `TableNames` list.

**What it demonstrates:**

No DynamoDB tables were listed in us-east-1 during this check. This was an inventory check, not a migration or performance test.

### Final Verification and Limitations

The session identity check succeeded, and the RDS and DynamoDB inventory commands returned empty lists. The observations confirm what was visible during those checks but do not verify the café's EC2 database configuration, application compatibility, backup recovery, performance, or migration readiness.

No production migration was verified. The final target service must be selected using additional client requirements and test results.

## Operational Analysis

The café workload is relational because it uses MySQL, structured tables, SQL queries, and transactions. Amazon RDS for MySQL is the first service to evaluate because it could reduce the routine server maintenance handled by the client while preserving a relational database model.

Standard RDS should be compared with Amazon Aurora before making a final selection. Aurora should not be treated as the automatic choice because the client has not provided performance measurements, availability requirements, compatibility results, or a budget.

DynamoDB may fit a different workload in which independent records are retrieved by a known key and relational joins are unnecessary. The information provided does not justify replacing the café's existing relational database with NoSQL.

## Recommendation

Evaluate Amazon RDS for MySQL as the first modernization option. Compare it with the existing EC2 deployment and Amazon Aurora using application compatibility, total cost, availability, backup and recovery needs, performance, and operational responsibilities.

Before choosing a target, collect the database size, transaction volume, MySQL version, application dependencies, acceptable downtime, recovery time objective, recovery point objective, and budget. Test the proposed configuration and application connections in a nonproduction environment.

If a migration is justified, create and test a backup, plan the migration and rollback procedures, validate the application and data, and obtain approval before production cutover. AWS DMS and schema conversion tools should be evaluated only if the selected migration path requires them.

## Escalation Notes

The intern may collect evidence, compare services, document risks, and recommend next steps. The intern is not authorized to approve a production migration or independently make production architecture changes.

Escalate the final service selection, production connectivity changes, security-sensitive changes, availability configuration, migration scheduling, and cutover plan to the supervisor and client. The current evidence is insufficient to approve a migration.

## Lessons Learned

This assessment showed that database selection should depend on the workload rather than service popularity. Relational databases are appropriate when structured relationships, SQL queries, and transactions matter. NoSQL may be more suitable for specific access patterns that do not need relational features.

Managed databases can reduce routine infrastructure and database administration work, but they do not remove the client's responsibility for data security, application access, cost control, and recovery validation. Migration requires compatibility checks, testing, a rollback plan, and approval.

AWS inventory commands also have limits. An empty resource list in one region does not prove that a database does not exist elsewhere or that an EC2-hosted database is absent.

## Professional Vocabulary

- **Relational Database:** A database that organizes information into related tables and supports SQL queries.
- **NoSQL:** A category of databases that includes nonrelational data models designed for particular access and scaling needs.
- **Amazon RDS:** An AWS managed relational database service that handles many routine database administration tasks.
- **Amazon Aurora:** An AWS relational database engine compatible with MySQL and PostgreSQL.
- **Amazon DynamoDB:** An AWS managed NoSQL database service supporting key-value and document data models.
- **Multi-AZ:** A deployment across multiple Availability Zones intended to improve database availability and resilience.
- **Database Migration Service:** AWS DMS, a service that supports database migration and replication.
- **Schema Conversion:** Adapting database schemas and objects when moving between database engines or database platforms.
- **Managed Service:** A service in which the provider operates specified parts of the underlying technology.
- **Workload Fit:** How well a service matches an application's data model, access patterns, performance, availability, security, and cost requirements.
- **Recovery Time Objective (RTO):** The target time for restoring a service after an interruption.
- **Recovery Point Objective (RPO):** The acceptable amount of data loss measured in time.
