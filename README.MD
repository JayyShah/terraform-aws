#  Leverage Terraform as the Infrastructure as Code (IaC) tool to deploy a simple Wordpress application into AWS.

- Using Terraform as Infrastructure as code(IAC) tool to deploy a highly available WordPress Application in a single Region. It will be configured to leverage a Multi-AZ(Data Replication along multiple AZ) MYSQL data base and an autoscaler based on CPU utilization for web hosting.

![wordpress](terraform/media/wordpress-aws.png)

# Services Used:
    - AWS IAM - Securely manage Identities and access to AWS services and resources
    - AWS VPC - Define and launch AWS resources in logically isolated virtual environment
    - AWS EC2 - Provides Scalable Computing Capacity
    - AWS RDS - Fully managed, Open source cloud Database service that allows you to easily operate and scale your relational database.
    - AWS S3 - Object Storage service offering industry leading scalability, data availability, security and Performance.
    - AWS EFS - Serverless, Fully Elastic File storage
    - AWS Simple System Manager (SSM) - Operations hub for your aws applications and resources and a Secure end-to-end management solution.
    - AWS Secrets Manager - Manage, retrieve and rotate Database credentials, tokens, api keys and other Secrets.
    - AWS Certification Manager (ACM) - Provision and Manage SSL/TLS certificates with AWS services and connected resources.
    - AWS CloudFront - Content delievery Network(CDN) service built for high performance, security and developer convenience

# Terraform:
    - Terraform  is an infrastructure as code tool that lets you provision resources safely and efficiently. 
    - This is defined in human-readable configuration files that you can version, reuse and re-share.
    - It creates and manages resources on cloud platforms and other services through their APIs.
    - Terraform provide developers the ability to architect and to deploy a reliable, reproducible, and repeatable infrastructure.

![terraform](terraform/media/terraform.png)

# SetUp:

1. Open CloudFormation - Upload A template - Upload (cloud9-template.yaml) - Set a Unique name - Create Stack
2. Open Cloud9 - Open the Stack Created - Files - Upload Files - Upload terraform folder - Close the terminal - Open New Terminal - Settings(Top Right Corner) - AWS Settings - Turn Off Temporary credentials - Run `(aws sts get-caller-identity --no-cli-pager)`.

# Folder Layout:
    - main.tf - contains the core resources of the module
    - outputs.tf - contains the outputs of the module
    - variables.tf - contains the input variables of the module
    - providers.tf - contains the provider configuration.

# Variables and Outputs

1. Local Values:
    - Names assigned and can be used throughout your code. 
    - Local values can be constant or referenced values. 

2. Input variable:
    - Used to provide input parameters to customize terraform module without altering module's source code and prevent hard coding values and enables to re-use the code.

3. Output variables:
    - Allows you to information on the resources so that others Terraform configurations can use it.

# Providers, Resources and Data Sources

1. Providers:
    - provide interactions with cloud providers, Software as a Service (SaaS) providers and other API.
    - Each provider provides a set of resources and data sources that Terraform can manage.

2. Resources:
    - They are the core element in Terraform. Declaring a resource can define one or more infrastructure resource objects such as compute, networking, etc.

3. Data Sources:
    - Allows lookup of resources defined outside of Terraform and provide the attributes found within that resource.

# Terraform State
    - Terraform stores information about your infrastructure within a state file, typically in a terraform.tfstate file.
    - Terraform uses state to determine which changes to make to your infrastructure like to modify, add or remove.
    - Before any operation is applied, terraform performs a refresh to sync the current infrastructure for any resources configuration drifts.

# Configuration:

1. Creating input variables, provider, local values and data sourcesHeader anchor link
    # Input Variables
    - az_num - Set the number of availability zones to use
    - namespace - Set a prefix for the resource names
    - vpc_cidr_block - Set the IP address block for the virtual private cloud (VPC)

    # Provider
    - aws - Set the minimal provider version and default resource tags
    - Local Values - temporary localized variables of common configs
    - Data Sources
    - aws_region - Get the current region
    - aws_availability_zones - Get list of availability zones
    - aws_ami - Get a specific Linux Amazon machine image (AMI)
    - aws_iam_policy - Get a specific Identity and Access Management (IAM) policy
    - aws_iam_policy_document - Generate a IAM policy document to allow EC2 to assume a role

- `(terraform init)`
- `(terraform validate)`
- `(terraform plan)`
- `(terraform apply -auto-approve)`

2. Create IAM Resources
    - Create our necessary IAM resources, that will allow your EC2 instances to read from an S3 bucket, as well get full access permissions to the RDS resources.

![resources](terraform/media/resources.png)    

    - aws_iam_role - Create an IAM Role
    - aws_iam_instance_profile - Create an IAM EC2 Instance profile of a IAM role to attach to an EC2 instance.

- `(terraform validate)`
- `(terraform plan)`
- `(terraform apply -auto-approve)`

    - To quickly confirm the successful creation of the instance profiles, run the following command.
    `(App Profile)`

    `(aws iam get-instance-profile --instance-profile-name app-profile | grep InstanceProfile -q && echo "Successfully created app-profile" || echo "Creation of app-profile was unsuccessful")`

    `(Web Hosting)`

    `(aws iam get-instance-profile --instance-profile-name web-hosting-profile | grep InstanceProfile -q && echo "Successfully created web-hosting-profile" || echo "Creation of Web-hosting-profile was unsuccessful")`

3. Resources for Networking
    - The following resources will be needed to host our application
    - Create a New VPC(virtual Private cloud) with all the necessary resources to run our sample application

![networking](terraform/media/networking-resources.png)

    - aws_vpc - Create a default VPC
    - aws_subnet - Create a subnet within a VPC
    - aws_internet_gateway - Allow communication between VPC and the internet
    - aws_route_table - Create a routing table to control where the network traffic is directed
    - aws_main_route_table_association - Set the default network routing table for the default VPC
    - aws_route_table_association - Associate the route table to a VPC subnet
    - aws_nat_gateway - Create a Network Address Translation (NAT) gateway to allow private subnet to access services
    - aws_eip - Create an Elastic IP (EIP) for the NAT Gateways

- `(terraform validate)`
- `(terraform plan)`
- `(terraform apply -auto-approve)`

# Review Deployment
    - Navigate to VPC - Seach "aws-terraform" 
    - There will be in total of 1 VPC, 6 Subnets - 3 Route Tables - 1 Internet Gateway - 2 NAT Gateways 

4. Resources for Security
    - A VPC endpoint enables you to privately connect to supported AWS services and VPC endpoint services powered by AWS PrivateLink.
    - Amazon VPC instances do not require public IP addresses to communicate with resources of the service. Traffic between an Amazon VPC and a service does not leave the Amazon network.
    - aws_security_group - Create a security group that controls the traffic that is allowed to reach and leave the resources
    - aws_vpc_endpoint - Create a VPC endpoint (VPCE) enables customers to privately connect to supported AWS services

![security](terraform/media/security.png)

- `(terraform validate)`
- `(terraform plan)`
- `(terraform apply -auto-approve)`

# Review Deployment:
    - Navigate Back to VPC - Click Security - Security Groups - There will be in total 4 Security groups - 5  Endpoints 

5. Resources for application
    - Created an RDS MySQL database, a load balancer, an autoscaling group, an EFS file system, an S3 bucket, and many more resources.

![application](terraform/media/application.png)

    - aws_db_subnet_group - Create a subnet group for database across availability zones
    - random_password - Create a random password
    - aws_secretsmanager_secret - Create a Secrets Manager entry
    - aws_secretsmanager_secret_version - Create a value for the Secrets Manager entry
    - aws_db_instance - Create a MySQL database
    - aws_efs_file_system - Create an EFS volume
    - aws_efs_mount_target - Create a configuration where the EFS volume to a set of instances
    - aws_instance - Create an EC2 that bootstrap the Wordpress application
    - aws_ami_copy - Create a copy of the Amazon Machine Image (AMI) being used
    - aws_lb - Create a application load balancer (ALB) for the Wordpress web application
    - aws_launch_template - Create a launch template that hosts the Wordpress web application
    - aws_autoscaling_group - Create a auto-scaler group that scale the Wordpress web application instances
    - aws_autoscaling_policy - Create a auto scaling policy
    - aws_cloudwatch_metric_alarm - Create a CloudWatch Metric Alarm
    - aws_lb_target_group - Create a target group to attach the load balancer for the Wordpress web application
    - aws_lb_listener - Create a load balancer listener for the ALB
    - tls_private_key - Create a TLS private key
    - tls_self_signed_cert - Create a public TLS certificate signed by the private key
    - aws_acm_certificate - Create an ACM certificate entry with the TLS public certificate and private key
    - aws_s3_bucket - Create a private S3 Bucket
    - aws_s3_bucket_public_access_block - Create a public access block which disable public access for the private S3 Bucket
    - aws_s3_object - An object of S3, typically represent data similar file
    - aws_cloudfront_distribution - Create a CloudFront distribution for the Wordpress web application
    - aws_cloudfront_cache_policy - Create a CloudFront cache policy for the Wordpress web application

- `(terraform validate)`
- `(terraform plan)`
- `(terraform apply -auto-approve)`

# Review Deployment:
    - Navigate to AWS Console and Search for the following:
        - EC2
        - EFS
        - RDS
        - CloudWatch
        - S3
        - ACM
        - ELB

6. OutPuts for Application:
    - Output Variables make information about your infrastructure available on the command line, and can expose information for other Terraform configurations to use.
    - Output Variables are similar to return values in most programming languages.
    
    - alb_endpoint_uri - Application Load Balancer Uniform Resource Identifier
    - alb_endpoint_url - Application Load Balancer Uniform Resource Locator
    - cloudfront_endpoint_url - CloudFront Uniform Resource Identifier

- `(terraform validate)`
- `(terraform plan)`
- `(terraform apply -auto-approve)`

7. Validate application Deployment
    - Copy the value from **cloudfront_endpoint_url** output and enter it on a new tab in current browser.
    - Verify application WebPage

![copy](terraform/media/sample.jpg)

8. Destroy the Project
    - Run `(terraform destroy -auto-approve)`
    - Navigate to CloudFormation - Stacks - Select the name provided for the stack - Delete Stack.

# Key Outcomes:
    - Basic building blocks of Terraform.
    - Setting up Resources on AWS using Terraform.
    - Update and Deploy changes to Infrastructure Environment.

