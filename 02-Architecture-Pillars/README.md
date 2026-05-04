# AWS SAA-C03 Notes — Architecture Pillars

This section covers the core architecture ideas : elasticity, scalability, high availability, fault tolerance, reliability, resiliency, CloudWatch, CloudTrail, monolithic vs. microservices design, disaster recovery, and the AWS Shared Responsibility Model.[cite:1]

## Section scope

The section outline in this PDF includes Elasticity and Scalability, High Availability and Fault Tolerance, Elastic Load Balancing 101, Reliability and Resiliency, Amazon CloudWatch 101, APIs, Amazon CloudTrail, Monolithic vs. Microservices applications, Disaster Recovery approaches, and the AWS Shared Responsibility Model.[cite:1]

## Elasticity and scalability

Elasticity is the degree to which a system can adjust to workload changes by provisioning and deprovisioning resources so the available capacity matches the need.[cite:1] In AWS, this usually appears as horizontal scaling out when demand rises and scaling in when demand falls, commonly implemented with EC2 Auto Scaling Groups.[cite:1]

Scalability is the ability of a system to handle increasing workload by adding resources.[cite:1] It can happen through vertical scaling, where an instance is resized up or down, or through horizontal scaling, where instances are added or removed.[cite:1]

### Horizontal vs. vertical scaling

| Type | What changes | Strength | Limitation |
|---|---|---|---|
| Horizontal scaling | Adds or removes instances | Better for high availability and elasticity | Application may need stateless design or shared session storage |
| Vertical scaling | Increases CPU, RAM, or other capacity on one instance | Simpler for some legacy workloads | Has hardware limits and often requires downtime |

### DevOps and production angle

For CI/CD and production systems, horizontal scaling is usually preferred because it works better with immutable deployments, rolling updates, and Auto Scaling Groups. Stateful applications often need sticky sessions or an external session store such as ElastiCache so any instance can serve any request consistently.

## High availability and fault tolerance

The PDF introduces high availability, fault tolerance, and ELB together as linked architecture foundations.[cite:1] High availability means keeping the service accessible with minimal downtime, usually by distributing components across multiple Availability Zones and failing over when one component or zone fails.[cite:1]

Fault tolerance goes further by keeping the system operating with no meaningful interruption when a component fails. In practice, fault-tolerant designs cost more because they keep enough redundant capacity active to absorb failure immediately.

### Practical comparison

| Pattern | Behavior during failure | Cost profile | Common AWS example |
|---|---|---|---|
| High availability | Brief interruption or failover may occur | Moderate | Multi-AZ deployments, ALB across AZs |
| Fault tolerance | Service continues with no meaningful interruption | Higher | Active/Active multi-AZ or multi-site architectures |

### ELB basics in this section

Elastic Load Balancing distributes traffic across multiple targets and supports healthier, more available architectures.[cite:1] A core behavior is that unhealthy instances are removed from request routing after failing health checks, which protects users from being sent to broken targets.

## Reliability and resiliency

A reliable workload must be designed to automatically recover from failure and scale horizontally to increase aggregate availability.[cite:1] The PDF defines resiliency as the ability of a workload to recover from infrastructure or service disruptions, dynamically acquire computing resources to meet demand, and mitigate disruptions such as misconfigurations or transient network issues.[cite:1]

Reliability is the outcome the business wants, while resiliency is the engineering capability that helps achieve it. In AWS production design, ELB, Auto Scaling, Multi-AZ deployments, backups, and managed services all contribute to resilient and reliable systems.

## Monitoring and visibility — Amazon CloudWatch

CloudWatch is presented as the center of real-time monitoring and visibility in AWS.[cite:1] The PDF states that it is a metric data repository, provides statistics based on metric data, monitors alarms that can trigger actions, supports CloudWatch Logs, and works with both standard AWS metrics and custom metrics.[cite:1]

### Why it matters in DevOps

CloudWatch is central to operating CI/CD workloads after deployment. Teams use it to watch instance health, application metrics, logs, and alarms; then connect those alarms to notifications or Auto Scaling actions so systems respond automatically to operational issues.

### Typical production uses

- Alert when CPU, memory, latency, or error rate crosses a threshold.
- Trigger Auto Scaling when load rises.
- Aggregate logs from EC2, Lambda, containers, or applications.
- Build dashboards for release validation and ongoing service health.

## APIs and Amazon CloudTrail

The PDF explains that an API is a set of rules that allows programs to talk to each other and notes that all access in AWS uses APIs, including actions taken through the AWS Management Console.[cite:1] This matters because CloudTrail records AWS API activity, which makes it a key service for auditing, troubleshooting, and security investigations.[cite:1]

CloudTrail enables governance, compliance, operational, and risk auditing of AWS accounts, and its event history records actions taken by IAM users, roles, or AWS services for 90 days.[cite:1] To retain logs longer or archive them, a CloudTrail trail must be created to deliver logs to an S3 bucket.[cite:1]

### CloudTrail event types

| Event type | What it captures | Default state |
|---|---|---|
| Management events | Management operations on resources such as creating a bucket or subnet | Enabled by default |
| Data events | Resource-level operations such as S3 object access or Lambda invocation | Disabled by default |
| Insights events | Unusual API write activity patterns | Disabled by default |

### Important operational notes

The PDF states that CloudTrail logs AWS service API calls, not API calls made to workloads hosted on AWS.[cite:1] It also notes that CloudTrail log file integrity validation can help determine whether delivered trail logs were modified, deleted, or left unchanged after delivery to S3, which is useful for security and forensics investigations.[cite:1]

## Monolithic vs. microservices applications

The course compares traditional monolithic applications with microservices-based applications.[cite:1] A monolithic application treats the whole process or tier as one unit, while microservices split functionality into smaller independent services.[cite:1]

### Architectural comparison

| Characteristic | Monolithic | Microservices |
|---|---|---|
| Deployment | Entire app often deployed together | Services can be deployed independently |
| Scaling | Scale the whole application | Scale only the busy service |
| Failure blast radius | One issue can affect the whole app | Problems can be isolated per service |
| AWS fit | Single EC2 app stack or simple platform setup | ECS, EKS, Lambda, API-driven architectures |

### CI/CD relevance

Microservices fit DevOps pipelines well because each service can have its own build, test, release, rollback, and scaling policy. That reduces deployment coupling and lets teams move faster, especially when different services change at different rates.

## Disaster recovery

The PDF defines a disaster as any event that can negatively affect business continuity or finances and defines Disaster Recovery as being ready for and able to recover from disaster situations.[cite:1] It also defines Recovery Time Objective, or RTO, as the time taken after disruption to restore a business process or infrastructure, and Recovery Point Objective, or RPO, as the acceptable data loss measured in time.[cite:1]

### DR strategies in AWS

The PDF compares four disaster recovery approaches: Backup and Restore, Pilot Light, Warm Standby, and Multi-Site.[cite:1] Cost and recovery speed increase as you move from Backup and Restore toward Multi-Site.[cite:1]

| Strategy | Core idea | Relative speed | Relative cost |
|---|---|---|---|
| Backup and Restore | Keep backups and rebuild after disaster | Slowest | Cheapest |
| Pilot Light | Keep core components running and replicate data | Faster than backup/restore | Low to medium |
| Warm Standby | Keep a scaled-down working environment running | Fast | Medium to high |
| Multi-Site | Keep full environments running in parallel | Fastest | Highest |

### Example decision logic

- Use Backup and Restore when budgets are tight and recovery can take hours.
- Use Pilot Light when core data and services must be ready, but full compute can wait until failover.
- Use Warm Standby when quick recovery matters and some steady-state DR cost is acceptable.
- Use Multi-Site for systems that require near-zero downtime and near-zero data loss.

## AWS Shared Responsibility Model

The PDF explains that customers are responsible for security in the cloud, including customer data, applications, identity and access management, operating systems, network and firewall configuration, encryption, data integrity, and traffic protection choices.[cite:1] AWS is responsible for security of the cloud, including software, compute, storage, database, networking, hardware, and the global infrastructure made up of Regions, Availability Zones, and edge locations.[cite:1]

The material also states that responsibilities vary by service model and by whether a service is managed.[cite:1] For managed services such as Amazon S3 and Amazon RDS, AWS is responsible for the infrastructure and major platform layers, while the customer still controls data handling, access control, and encryption decisions.[cite:1]

### Responsibility examples

| Item | AWS | Customer |
|---|---|---|
| Physical data center security | Yes | No |
| Guest OS patching on EC2 | No | Yes |
| Security group configuration on EC2 | No | Yes |
| Data encryption choices | Shared tooling provided | Customer configures and manages usage |
| S3 bucket access policy | No | Yes |
| RDS OS and platform maintenance | Yes | Customer still manages data access and security controls |

## Real-world DevOps examples

### Example 1: Web application release

A production web app can combine an Application Load Balancer, an EC2 Auto Scaling Group, CloudWatch alarms, and Multi-AZ deployment to improve elasticity, reliability, and high availability. A CI/CD pipeline can deploy a new version gradually while CloudWatch dashboards and alarms validate health after release.

### Example 2: Audit and compliance pipeline

A security-conscious AWS account can enable CloudTrail trails to S3, turn on integrity validation, and route selected logs into CloudWatch Logs for alerting. This supports incident response, change tracking, and compliance evidence collection.

### Example 3: Stateful application modernization

A legacy monolith that stores user sessions in instance memory becomes hard to scale horizontally. Moving session state to a shared store such as ElastiCache allows multiple instances behind a load balancer to serve the same users more safely.

## Exam traps quick reference

| Topic | Trap | Correct way to think |
|---|---|---|
| Elasticity vs scalability | Treating them as the same term | Scalability is the ability to grow; elasticity is automatic scale out and scale in based on demand |
| Vertical scaling | Assuming it scales forever | It has hardware limits and often requires downtime |
| Horizontal scaling | Assuming all apps can scale out easily | Stateful apps need sticky sessions or external session storage |
| High availability vs fault tolerance | Assuming both mean zero downtime | High availability allows brief failover; fault tolerance aims for continuous operation |
| ELB | Assuming ELB always sends traffic to every instance | ELB stops routing traffic to unhealthy targets after health checks fail |
| CloudWatch | Assuming metrics and alarms are the same | Metrics are measurements; alarms evaluate metrics and trigger actions |
| CloudTrail | Assuming it logs application API calls hosted on EC2 | It logs API calls to AWS services, not your app's own API traffic |
| CloudTrail retention | Assuming 90-day event history is enough for long-term compliance | Create a trail and deliver logs to S3 for long-term retention |
| CloudTrail event types | Assuming all event types are enabled by default | Management events are enabled; Data and Insights events are disabled by default |
| Monolith vs microservices | Assuming microservices are only about code style | They mainly change deployment, scaling, isolation, and operational boundaries |
| DR metrics | Mixing up RPO and RTO | RPO is acceptable data loss; RTO is acceptable recovery time |
| DR strategies | Confusing Pilot Light with Warm Standby | Pilot Light keeps minimal core components; Warm Standby keeps a scaled-down working environment |
| Shared responsibility | Assuming AWS secures everything automatically | AWS secures the cloud; the customer secures configurations, identities, data, and access inside it |
| Security groups | Assuming AWS is responsible for open inbound rules | Security groups are customer-managed controls |
| RDS responsibility | Assuming AWS handles all database security | AWS manages underlying platform and patching, but customer manages data access, encryption settings, and policies |

## Key rules to remember

- Prefer horizontal scaling for modern highly available workloads.
- Use elasticity when demand changes dynamically and cost efficiency matters.
- Combine ELB and Auto Scaling for resilient stateless web tiers.
- Use CloudWatch for monitoring and actioning; use CloudTrail for auditing and investigation.
- Choose DR strategy based on business RPO, RTO, and cost tolerance.
- Never confuse AWS-managed infrastructure responsibility with customer-managed configuration responsibility.
