# Quiet Riot 
### :notes: *C'mon, Feel The Noise* :notes:
  
_An enumeration tool for scalable, unauthenticated validation of AWS principals; including AWS Acccount IDs, root e-mail addresses, users, and roles._

__Credit:__ Daniel Grzelak [@dagrz](https://twitter.com/dagrz) for identifying the technique and Will Bengston [@__muscles](https://twitter.com/__muscles) for inspiring me to scale it.

See the blog post [here]()

### Featureploitation Limits
#### Throttling
After performing extensive analysis of scaling methods using the AWS Python (Boto3) SDK, I was able to determine that the bottleneck for scanning (at least for Python and awscli -based tools) is I/O capacity of a single-threaded Python application. After modifying the program to run with multiple threads, I was able to trigger exceptions in individual threads due to throttling by the various AWS APIs. You can see the results from running a few benchmarking test scans [here](./results/scan-run-statistics.txt). APIs that I tested had wildly different throttling limits and notably, s3 bucket policy attempts took ~10x as long as similar attempts against other services.

With further testing, I settled on a combination of SNS, ECR-Public, and ECR-Private services running in US-East-1 in ~40%/50%/10% configuration split with ~700 threads. The machine I used was a 2020 Macbook Air (M1 and 16 GB RAM). This configuration yielded on average ~1100 calls/sec, though the actual number of calls can fluctuate significantly depending on a variety of factors including network connectivity. Under these configurations, I did occasionally throw an exception on a thread from throttling...but I have subsequently configured additional (4 -> 7) re-try attempts via botocore that would eliminate this issue with some performance trade-off.

#### Computational Difficulty
To attempt every possible Account ID in AWS (1,000,000,000,000) would require an infeasible amount of time given only one account. Even assuming absolute efficiency*, over the course of a day an attacker will only be able to make 95,040,000 validation checks. Over 30 days, this is 2,851,200,000 validation checks and we are still over 28 years away from enumerating every valid AWS Account ID. Fortunately, there is nothing stopping us from registering many AWS accounts and automating this scan. While there is an initial limit of 20 accounts per AWS organization, I was able to get this limit increased for my Organization via console self-service and approval from an AWS representative. The approval occured without any further questions and now I'm off to automating this writ large. Again, assuming absolute efficiency, the 28 years scanning could potentially be reduced down to ~100 days.

*~1100 API calls/check per second in perpetuity per account and never repeating a guessed Account ID.

## Potential Supported Services

| # | AWS Service | Description | API Limits | Resource Pricing | Enumeration Capability |
| --- | ----------- | ----------- | --------------- |--------------- | ---------- |
| 1 | __SNS__ | Managed Serverless Notification Service | Unknown | Unknown | Yes |
| 2 | __KMS__ | Encryption Key Management Service | Unknown | Unknown | Yes |
| 3 | __SecretsManager__ | Managed Secret Store | Unknown | Unknown | Yes |
| 4 | __CodeArtifact__ | Managed Source Code Repository | Unknown | Unknown | Yes |
| 5 | __ECR Public__ | Managed Container Registry | Unknown | Unknown | Yes |
| 6 | __ECR Private__ | Managed Container Registry | Unknown | Unknown | Yes |
| 7 | __Lambda__ | Managed Serverless Function | Unknown | Unknown | Yes |
| 8 | __s3__ | Managed Serverless Object Store | Unknown | Unknown | Yes |
| 9 | __SES__ | SMTP Automation Service | Unknown | Unknown | Unknown |
| 10 | __ACM__ | Private Certificate Authority | Unknown | Unknown | Unknown |
| 11 | __CodeBuild__ | Software Build Agent | Unknown | Unknown | Unknown |
| 12 | __AWS Backup__ | Managed Backup Service | Unknown | Unknown | Unknown |
| 13 | __Cloud9__ | Managed IDE | Unknown | Unknown | Unknown |
| 14 | __Glue__ | Managed ETL Job Service | Unknown | Unknown | Unknown |
| 15 | __EKS__ | Managed K8s Service | Unknown | Unknown | Unknown |
| 16 | __Lex V2__ | Managed NLP Service | Unknown | Unknown | Unknown |
| 17 | __CloudWatch Logs__ | Managed Log Pipeline/Monitoring | Unknown | Unknown | Unknown |
| 18 | __VPC Endpoints__ | Managed Virtual Network | Unknown | Unknown | Unknown |
| 19 | __Elemental MediaStore__ | Unknown | Unknown | Unknown | Unknown |
| 20 | __OpenSearch__ | Managed ElasticSearch | Unknown | Unknown | Unknown |
| 21 | __EventBridge__ | Managed Serverless Event Hub | Unknown | Unknown | Unknown |
| 22 | __EventBridge Schemas__ | Managed Serverless Event Hub | Unknown | Unknown | Unknown |
| 23 | __IoT__ | Internet-of-Things Management | Unknown | Unknown | Unknown |
| 24 | __s3 Glacier__ | Cold Object Storage | Unknown | Unknown | Unknown |
| 25 | __ECS__ | Managed Container Orchestration | Unknown | Unknown | Unknown |
| 26 | __Serverless Application Repository__ | Managed Source Code Repository | Unknown | Unknown | No |
| 27 | __SQS__ | Managed Serverless Queueing Service | Unknown | Unknown | No |
| 28 | __EFS__ | Managed Serverless Elastic File System | Unknown | Unknown | No |

## Getting Started With Quiet Riot
To get started with Quiet Riot, clone the repository to your local directory. You'll need boto3 and AWS cli tools installed. You'll need credentials configured with sufficient privileges in an AWS account to deploy the resources (SNS topic, ECR-Public repository, and ECR-Private repository). Then you just run ./main.py and follow the prompts. If you don't bring your own wordlists, feel free to use one from the wordlists/ directory and I further recommend [SecLists Usernames](https://github.com/danielmiessler/SecLists/tree/master/Usernames).

### Prerequisites
awscli
boto3
botocore
Sufficient AWS credentials configured via CLI

