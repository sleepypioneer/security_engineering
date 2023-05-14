# Week 2 Project Answers

## Security Groups are an example of:

- Packet Filter
- Stateless Firewall
- **Stateful Firewall**
- Web Application Firewall
- Load Balancer

## Security groups:

- Protect network access to the AWS API.
- **Protect network access to AWS resources deployed by the tenant.**
- Both

## VPC ACLs are:

- Simpler as they are stateful.
- Easier to maintain as they are object-based and not tied to particular IP addresses or subnet ranges.
- Deployed on virtual firewall appliances that you need to configure.
- **Ideal for implementing high-level network policies.**

## Traffic between AWS subnets is controlled via:

- Load Balancers
- Routers
- **VPC ACLs**
- Security Groups
- Cannot be controlled because those are internal networks.

## Access to Polly and Thoughtpad is end to end encrypted because:
- They are accessed via TLS.
- Incorrect; it is not end-to-end because the TLS is terminated on the load-balancer.
- Their respective workloads are listening on TCP/443.
- The network traffic is secured via a VPC ACL.

## RDS Databases in AWS are encrypted by default.

- True
- **False**
- Not applicable; RDS does not support encryption.

## Control - List all places that logging should be added.

## Control - Do the roles assigned to the users apply the least privilege principles?

The readonly user (the one we are using) has a policy for ReadOnlyAccess, this seems like a good start, though it appears what can be read is not restricted. The other account has no policies attached, which suggests it has complete control and the full set of privileges, therefore, it does not apply the least privilege principle.

## Control - Is MFA (Multi-Factor Authentication) enabled on the accounts you encountered in IAM?

```sh
aws iam list-mfa-devices --user-name <username>
```

- Yes
- **No**

## Optional - Compute - Is ECR image scanning for runtime protection enabled? How can you tell.

```sh
aws ecr describe-repositories --query 'repositories[*].{RepositoryName:repositoryName,ImageScanningConfiguration:imageScanningConfiguration.scanOnPush}' --output table
```

| ImageScanningConfiguration  |    RepositoryName      |
|-----------------------------|------------------------|
|  False                      |  app-ecr-repo-709a364  |


## Optional - Compute - How are secrets managed specifically as it relates to ECS task definition configuration variables?

Secrets are within the task definition as environment variables as plain string values:

```sh
aws ecs describe-task-definition --task-definition arn:aws:ecs:us-east-1:849265715259:task-definition/django_database_task_definition-family:2  --query 'taskDefinition.{containerDefinitions: containerDefinitions[].{environment: environment[*].{name: name, value: value}}}' --output json
```

```
{
    "containerDefinitions": [
        {
            "environment": [
                {
                    "name": "DJANGO_NAME",
                    "value": "corise-django-admin"
                },
                {
                    "name": "DATABASE_ADDRESS",
                    "value": "mysql-server6e646b6.cq51gjhxucto.us-east-1.rds.amazonaws.com"
                },
                {
                    "name": "DATABASE_NAME",
                    "value": "votes"
                },
                {
                    "name": "USER_PASSWORD",
                    "value": "corise-admin-pass"
                },
                {
                    "name": "DATABASE_PORT",
                    "value": "3306"
                },
                {
                    "name": "SECRET_KEY",
                    "value": "corise-django-secret-key"
                },
                {
                    "name": "DJANGO_PASSWORD",
                    "value": "corise-django-admin-pass"
                },
                {
                    "name": "USER_NAME",
                    "value": "coriseadmin"
                }
            ]
        }
    ]
}
```

This is problematic as it means the secrets are stored in plain text and can be accessed by anyone with access to the task definition.

## Communication - Are the databases reachable from anywhere else apart from the relevant application?

Yes the Polly application shares only one security group which connects traffic to the loadbalancer (and therefore the outside world) and the database. This is problematic as it means the database is reachable from anywhere.


## Communication - Is there encryption in place between the browser and the applications? How can you tell?

Encryption between a browser and an application is typically managed by SSL/TLS. Both of the application's URL starts with https, meaning it is using SSL/TLS encryption. The loadbalancers also tell us that port 443 is exposed with TLS/HTTPS protocol.

```sh
aws elbv2 describe-listeners --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:849265715259:loadbalancer/app/wp-example-fe-alb-1112877/d2c57dbccfa6eea6 --query 'Listeners[].{Port:Port, Protocol:Protocol}' --output table
```

| Port  | Protocol   |
|-------|------------|
|  443  |  HTTPS     |
|  80   |  HTTP      |

```sh
aws elbv2 describe-listeners --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:849265715259:loadbalancer/net/django-balancer-ddf0f5f/74d469718e97d074  --query 'Listeners[].{Port:Port, Protocol:Protocol}' --output table
```

| Port  | Protocol   |
|-------|------------|
|  443  |  TLS       |
|  80   |  TCP       |


## Optional - Communication - Is there encryption in place between the applications and the databases?

```sh
aws rds describe-db-instances --query 'DBInstances[*].{DBInstanceIdentifier: DBInstanceIdentifier, StorageEncrypted: StorageEncrypted}' --output table
```


## Communication - Are there any firewalls or some form of Network Access Control Lists (NACLs) in place?

Yes there is a NACL in place for both VPCs.

```sh
aws ec2 describe-network-acls --query 'NetworkAcls[].{NetworkAclId: NetworkAclId, VpcId: VpcId}' --output table
```

|      NetworkAclId      |          VpcId          |
|------------------------|-------------------------|
|  acl-006a77c2a16553f57 |  vpc-06eb467fa397406d9  |
|  acl-068a7a63df7af5c01 |  vpc-07ddda53af641648f  |


## Communication - Are the containers spread across multiple subnets and/or AWS Regions for redundancy and availability?

Yes they are spread across two subnets in two different availability zones within both VPCs.


## Storage - Are the RDS DBs encrypted? Should they or should they not be if they don't store any user related content Please Explain.

No they are not encrypted. Encryption is not enabled by default. Event though they do not store any user related content, it is still a good idea to encrypt them incase more sensitive data is added in the future.

```sh
aws rds describe-db-instances --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,StorageEncrypted:StorageEncrypted}' --output table
```

|   DBInstanceIdentifier    | StorageEncrypted   |
|---------------------------|--------------------|
|  mysql-server6e646b6      |  False             |
|  wp-example-be-rdsbce40b7 |  False             |



## Storage - Any concerns whether storage infrastructure is protected from physical threats?

As an AWS user, you don't have direct control over the physical security of AWS data centers. 
> AWS operates under a shared security responsibility model, where AWS is responsible for the security of the underlying cloud infrastructure and you are responsible for securing workloads you deploy in AWS.

However, AWS's data centers have robust physical security controls in place. You can take a look in their virtual tour of an AWS data center, [here](https://aws.amazon.com/compliance/data-center/data-centers/). Starting with a perimeter layer of security guards, fencing and intrusion detection technology, they strictly limit access to the data center. At the data layer they separate privileges by layer and deploy threat detection devices. They also consider environmental factors such as flooding, extreme weather, and seismic activity.


## Optional: Communication - Should network segmentation be used? If yes, please explain how that would help secure the overall environment.
## Optional: Storage - Is data being backed up and if yes, how?

Currently there is no automated backups enabled for the RDS instances.

```sh
aws rds describe-db-instances  --query 'DBInstances[*].{DBInstanceIdentifier:DBInstanceIdentifier,BackupRetentionPeriod:BackupRetentionPeriod}' --output table
```

|  BackupRetentionPeriod |   DBInstanceIdentifier     |
|------------------------|-----------------------------|
|  0                     |  mysql-server6e646b6       |
|  0                     |  wp-example-be-rdsbce40b7  |