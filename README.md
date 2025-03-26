# **CloudFormation Template: Highly Available VPC with ALB, NAT Gateway, and RDS PostgreSQL Database**

## **Overview**

This AWS CloudFormation template provisions a highly available infrastructure for a web application, including:

-   A **VPC** with public and private subnets across multiple availability zones (AZs).
-   An **Application Load Balancer (ALB)** for handling HTTP traffic.
-   **Elastic Load Balancer Security Group** to allow internet traffic.
-   **ECS Security Group** to allow traffic between ECS services and the ALB.
-   **NAT Gateway** to enable private subnets to access the internet.
-   **RDS PostgreSQL Database** with Multi-AZ failover enabled for high availability.
-   **Security Groups** for controlling access between ECS services, the ALB, and the RDS PostgreSQL instance.
-   **AWS Systems Manager Parameter Store** to store database configuration parameters such as username, password, and port.

The PostgreSQL database is configured to use Multi-AZ for high availability, ensuring automatic failover in case of instance failure. The database credentials are securely stored in **AWS Systems Manager Parameter Store**.

## **Architecture**

-   **VPC**: The template creates a custom VPC with a CIDR block of `10.0.0.0/16` and splits it into public and private subnets.
-   **Subnets**: Three public subnets and three private subnets are created, each located in a different Availability Zone (AZ) to provide high availability.
-   **NAT Gateway**: The NAT gateway is provisioned to allow resources in private subnets to access the internet.
-   **Security Groups**:
    -   **ECSSecurityGroup**: Allows ECS services to communicate with the ALB.
    -   **ALBSecurityGroup**: Allows HTTP traffic (port 80) from the internet to the load balancer.
    -   **RDSSecurityGroup**: Allows access to the PostgreSQL RDS instance from ECS services.
-   **RDS PostgreSQL**: The PostgreSQL instance is deployed in a Multi-AZ configuration with automatic failover for high availability.
-   **AWS Systems Manager Parameter Store**: Stores the database username, password, and port as parameters.

## **Parameters**

-   `DBUsername`: The username for the PostgreSQL database. This is a string parameter provided at the time of stack creation.
-   `DBPassword`: The password for the PostgreSQL database. This is a secure string parameter provided at the time of stack creation.
-   `DBName`: The name of the PostgreSQL database.
-   `DBPort`: The port for the PostgreSQL database (default: 5432).

## **Resources Created**

-   **VPC**: A custom VPC with CIDR block `10.0.0.0/16` and DNS support.
-   **Subnets**: Three public and three private subnets, each in a different AZ.
-   **Internet Gateway**: To provide public internet access.
-   **NAT Gateway**: For internet access from private subnets.
-   **Route Tables**: Public and private route tables to direct traffic accordingly.
-   **Security Groups**:
    -   **ECS Security Group**: Allows communication between ECS and ALB.
    -   **ALB Security Group**: Allows internet access to the ALB.
    -   **RDS Security Group**: Allows communication from ECS services to the PostgreSQL database.
-   **RDS PostgreSQL Database**: A PostgreSQL database with Multi-AZ failover and storage for the "photoalbum" database.
-   **AWS Systems Manager Parameter Store**: Stores the database username, password, and port.

## **How to Use This Template**

### **Prerequisites**

-   An AWS account with sufficient permissions to create resources such as VPCs, security groups, and RDS instances.
-   AWS CLI or AWS Management Console access to deploy the CloudFormation stack.

### **Deploying the Stack**

1. **Save the CloudFormation Template**:

    - Save the CloudFormation template as a `.yaml` file (e.g., `JuliusPhotoAlbumVPC.yaml`).

2. **Deploy via AWS Management Console**:

    - Go to the [AWS CloudFormation Console](https://console.aws.amazon.com/cloudformation).
    - Click **Create Stack**.
    - Upload the saved `.yaml` file or copy-paste its contents into the template editor.
    - Provide a **stack name** (e.g., `JuliusPhotoAlbumVPC`).
    - Enter the **DBUsername**, **DBPassword**, and **DBName** for the PostgreSQL database.
    - Click **Next** and configure stack options.
    - Review and click **Create**.

3. **Deploy via AWS CLI**:

    - Use the AWS CLI to create the stack.

    ```bash
    aws cloudformation create-stack \
      --stack-name JuliusPhotoAlbumVPC \
      --template-body file://JuliusPhotoAlbumVPC.yaml \
      --parameters ParameterKey=DBUsername,ParameterValue=<your-db-username> \
                   ParameterKey=DBPassword,ParameterValue=<your-db-password> \
                   ParameterKey=DBName,ParameterValue=<your-db-name> \
      --capabilities CAPABILITY_NAMED_IAM
    ```

### **Accessing the Resources**

-   **RDS PostgreSQL Database**:

    -   The database is deployed in a private subnet and is **not publicly accessible**. You can access it via an application hosted in ECS or through an EC2 instance in the private subnet.
    -   The **DBUsername**, **DBPassword**, and **DBPort** are stored in AWS Systems Manager Parameter Store.

-   **ECS Service**:

    -   ECS services can connect to the database using the security groups that allow access to the RDS instance.

-   **ALB (Application Load Balancer)**:
    -   The ALB will be accessible over HTTP on port 80, routing traffic to the ECS service.

## **Outputs**

Once the stack is successfully created, the following outputs will be available:

-   **Public Subnets**: The resource IDs of the three public subnets.
-   **Private Subnets**: The resource IDs of the three private subnets.
-   **Security Groups**: The resource IDs of the ECS, ALB, and RDS security groups.
-   **RDS PostgreSQL Database**: The identifier of the created PostgreSQL RDS instance.
-   **SSM Parameters**: The names of the SSM parameters storing the database credentials.

## **Template Overview**

### **Key Resources**

-   **VPC**:

    -   A custom VPC (`JuliusPhotoAlbumVPC2`) with a CIDR block of `10.0.0.0/16`.
    -   Multiple subnets across three availability zones for high availability.
    -   Route tables for routing traffic to the internet (for public subnets) and using the NAT Gateway (for private subnets).

-   **Security Groups**:

    -   **ECS Security Group**: Allows traffic from the ALB to ECS (port 3000).
    -   **ALB Security Group**: Allows inbound HTTP (port 80) from the internet.
    -   **RDS Security Group**: Allows inbound traffic from ECS to PostgreSQL on port 5432.

-   **NAT Gateway**: Provides internet access to the private subnets.
-   **RDS PostgreSQL Database**: A multi-AZ, fault-tolerant PostgreSQL database with automatic failover.
-   **AWS Systems Manager Parameter Store**: Stores the database username, password, and port.

## **Conclusion**

This CloudFormation template automates the deployment of a scalable, highly available infrastructure using AWS best practices. It integrates services like ECS, ALB, RDS, and Systems Manager Parameter Store to provide a secure, fault-tolerant environment for running a web application with PostgreSQL as the database backend.

For any troubleshooting or modifications, refer to the AWS CloudFormation console, review the logs, and adjust parameters as necessary.
