# Week 1 Project Answers

Control:
- [What is the user ID associated with the credentials?](#control---what-is-the-user-id-associated-with-the-credentials)
- [What permissions do your current credentials grant? Can you assume any roles?](#control---what-permissions-do-your-current-credentials-grant-can-you-assume-any-roles)
- [What are the roles on this account?](#control---what-are-the-roles-on-this-account)
- [Is there any logging setup? If so, what kind?](#control---is-there-any-logging-setup-if-so-what-kind)
- [Are there any alarms or thresholds setup? If so, what kind?](#control---are-there-any-alarms-or-thresholds-setup-if-so-what-kind)
- [What can you learn about how this environment was deployed?](#control---what-can-you-learn-about-how-this-environment-was-deployed)

Compute:
- [What sorts of services are used to run the application? Virtual instances? Containers?](#compute---what-sorts-of-services-are-used-to-run-the-application-virtual-instances-containers)
- [Provide details of the compute entities that you discover](#compute---provide-details-of-the-compute-entities-that-you-discover)

Communication:
- [What service ports are in use by the application?](#communication---what-service-ports-are-in-use-by-the-application)
- [Are there any load balancers in use? If so, what type(s)?](#communication---are-there-any-load-balancers-in-use-if-so-what-types)
- [What service ports and protocols are exposed by this application?](#communication---what-service-ports-and-protocols-are-exposed-by-this-application)
- [What does the network configuration look like? List all subnets that you can find.](#communication---what-does-the-network-configuration-look-like-list-all-subnets-that-you-can-find)
- [Can you determine what network segmentation, if any, is in use?](#communication---can-you-determine-what-network-segmentation-if-any-is-in-use)
- [What are the configured security groups?](#communication---what-are-the-configured-security-groups)
- [What are the VPCs associated with the applications?](#communication---what-are-the-vpcs-associated-with-the-applications)
- [can you describe the end-to-end traffic flow from user to the application servers? Make sure to list all intermediate entities](#communication---can-you-describe-the-end-to-end-traffic-flow-from-user-to-the-application-servers-make-sure-to-list-all-intermediate-entities)

Storage:
- [What types of storage are used by the applications? Break this down per service](#storage---what-types-of-storage-are-used-by-the-applications-break-this-down-per-service)
- [Provide the configuration details of each storage service (service ports, network configuration, e.t.c.)](#storage---provide-the-configuration-details-of-each-storage-service-service-ports-network-configuration-etc)

Optional:
- [How would you update the documentation for Thoughtpad or Polly to better reflect the infrastructure you explored? Are there any inaccuracies you would address?](#how-would-you-update-the-documentation-for-thoughtpad-or-polly-to-better-reflect-the-infrastructure-you-explored-are-there-any-inaccuracies-you-would-address)
- [How would you change the design of either project? Share why you would make the specific changes that you are proposing.](#how-would-you-change-the-design-of-either-project-share-why-you-would-make-the-specific-changes-that-you-are-proposing)
- [Do these deployments share any commonalities with projects you have deployed or managed in real life?](#do-these-deployments-share-any-commonalities-with-projects-you-have-deployed-or-managed-in-real-life)

---

## Control

### Control - What is the user ID associated with the credentials?

```sh
aws sts get-caller-identity --query 'UserId' --output text
```

```
AIDA4LPBONA56ROR4RWMQ
```

```sh
aws iam list-users
```

```json
{
  "Users": [
    {
      "Path": "/",
      "UserName": "corise",
      "UserId": "AIDA4LPBONA537PM4AEV2",
      "Arn": "arn:aws:iam::849265715259:user/corise",
      "CreateDate": "2023-03-12T22:41:57+00:00"
    },
    {
      "Path": "/",
      "UserName": "corise-readonly",
      "UserId": "AIDA4LPBONA56ROR4RWMQ",
      "Arn": "arn:aws:iam::849265715259:user/corise-readonly",
      "CreateDate": "2023-03-27T00:12:45+00:00"
    }
  ]
}
```


### Control - What permissions do your current credentials grant? Can you assume any roles?

```sh
aws iam list-attached-user-policies --user-name corise-readonly
```

```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "ReadOnlyAccess",
      "PolicyArn": "arn:aws:iam::aws:policy/ReadOnlyAccess"
    }
  ]
}
```

```sh
aws iam list-roles --query 'Roles[].{RoleName:RoleName, AWSUserRoleAccess: join(`,`, AssumeRolePolicyDocument.Statement[].Principal.AWS)}' --output table
```


|  AWSUserRoleAccess |                RoleName                 |
|--------------------|-----------------------------------------|
|                    |  app-exec-role-1d49f8e                  |
|                    |  app-task-role-abc9304                  |
|                    |  AWSServiceRoleForAPIGateway            |
|                    |  AWSServiceRoleForECS                   |
|                    |  AWSServiceRoleForElasticLoadBalancing  |
|                    |  AWSServiceRoleForGlobalAccelerator     |
|                    |  AWSServiceRoleForOrganizations         |
|                    |  AWSServiceRoleForRDS                   |
|                    |  AWSServiceRoleForSupport               |
|                    |  AWSServiceRoleForTrustedAdvisor        |
|                    |  ecsTaskExecutionRole                   |
|                    |  wp-example-fe-task-role-fb52a41        |



### Control - What are the roles on this account?

```sh
aws iam list-roles --query 'join(`\n`, Roles[].RoleName)' --output text 
````

app-exec-role-1d49f8e
app-task-role-abc9304
AWSServiceRoleForAPIGateway
AWSServiceRoleForECS
AWSServiceRoleForElasticLoadBalancing
AWSServiceRoleForGlobalAccelerator
AWSServiceRoleForOrganizations
AWSServiceRoleForRDS
AWSServiceRoleForSupport
AWSServiceRoleForTrustedAdvisor
ecsTaskExecutionRole
wp-example-fe-task-role-fb52a41

```sh
aws iam list-roles --query 'Roles[].{RoleName:RoleName, RoleArn:Arn, AWSUserRoleAccess: join(`,`, AssumeRolePolicyDocument.Statement[].Principal.AWS), ServiceRoleAccess: join(`,`, AssumeRolePolicyDocument.Statement[].Principal.Service)}' --output table
```

| AWSUserRoleAccess |                                                          RoleArn                                                           |               RoleName                 |          ServiceRoleAccess           |
|-------------------|----------------------------------------------------------------------------------------------------------------------------|----------------------------------------|--------------------------------------|
|                   |  arn:aws:iam::849265715259:role/app-exec-role-1d49f8e                                                                      |  app-exec-role-1d49f8e                 |  ecs-tasks.amazonaws.com             |
|                   |  arn:aws:iam::849265715259:role/app-task-role-abc9304                                                                      |  app-task-role-abc9304                 |  ecs-tasks.amazonaws.com             |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/ops.apigateway.amazonaws.com/AWSServiceRoleForAPIGateway                  |  AWSServiceRoleForAPIGateway           |  ops.apigateway.amazonaws.com        |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/ecs.amazonaws.com/AWSServiceRoleForECS                                    |  AWSServiceRoleForECS                  |  ecs.amazonaws.com                   |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/elasticloadbalancing.amazonaws.com/AWSServiceRoleForElasticLoadBalancing  |  AWSServiceRoleForElasticLoadBalancing |  elasticloadbalancing.amazonaws.com  |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/globalaccelerator.amazonaws.com/AWSServiceRoleForGlobalAccelerator        |  AWSServiceRoleForGlobalAccelerator    |  globalaccelerator.amazonaws.com     |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/organizations.amazonaws.com/AWSServiceRoleForOrganizations                |  AWSServiceRoleForOrganizations        |  organizations.amazonaws.com         |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/rds.amazonaws.com/AWSServiceRoleForRDS                                    |  AWSServiceRoleForRDS                  |  rds.amazonaws.com                   |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/support.amazonaws.com/AWSServiceRoleForSupport                            |  AWSServiceRoleForSupport              |  support.amazonaws.com               |
|                   |  arn:aws:iam::849265715259:role/aws-service-role/trustedadvisor.amazonaws.com/AWSServiceRoleForTrustedAdvisor              |  AWSServiceRoleForTrustedAdvisor       |  trustedadvisor.amazonaws.com        |
|                   |  arn:aws:iam::849265715259:role/ecsTaskExecutionRole                                                                       |  ecsTaskExecutionRole                  |  ecs-tasks.amazonaws.com             |
|                   |  arn:aws:iam::849265715259:role/wp-example-fe-task-role-fb52a41                                                            |  wp-example-fe-task-role-fb52a41       |  ecs-tasks.amazonaws.com             |

### Control - Is there any logging setup? If so, what kind?

#### VPC Flow Logs

VPC Flow Logs capture information about the IP traffic going to and from network interfaces in your VPC.

```sh
aws ec2 describe-flow-logs

```

```json
{
    "FlowLogs": [
        {
            "CreationTime": "2023-04-04T23:43:34.035000+00:00",
            "DeliverLogsErrorMessage": "Access error",
            "DeliverLogsPermissionArn": "arn:aws:iam::849265715259:role/wp-example-fe-task-role-fb52a41",
            "DeliverLogsStatus": "FAILED",
            "FlowLogId": "fl-02f797b63de522d8e",
            "FlowLogStatus": "ACTIVE",
            "LogGroupName": "sh-test",
            "ResourceId": "vpc-06eb467fa397406d9",
            "TrafficType": "ALL",
            "LogDestinationType": "cloud-watch-logs",
            "LogFormat": "${version} ${account-id} ${interface-id} ${srcaddr} ${dstaddr} ${srcport} ${dstport} ${protocol} ${packets} ${bytes} ${start} ${end} ${action} ${log-status}",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "sh-test"
                }
            ],
            "MaxAggregationInterval": 600
        }
    ]
}
```

#### Amazon CloudWatch Logs

CloudWatch Logs lets you monitor, store, and access your log files from Amazon EC2 instances, AWS CloudTrail, Route 53, and other sources.

```sh
aws logs describe-log-groups --query 'logGroups[].{logGroupName: logGroupName, retentionInDays: retentionInDays, storedBytes: storedBytes}' --output table
```

|   logGroupName   |  retentionInDays  |  storedBytes  |
|------------------|-------------------|---------------|
|  RDSOSMetrics    |  30               |  0            |
|  django-log-group|  1                |  867416       |
|  sh-test         |  5                |  0            |



### Control - Are there any alarms or thresholds setup? If so, what kind?

```sh
aws cloudwatch describe-alarms --query 'MetricAlarms[].{AlarmName: AlarmName, MetricName: MetricName, Threshold: Threshold}' --output table 
```


|    AlarmName    |    MetricName      |  Threshold  |
|-----------------|--------------------|-------------|
|  BudgetThreshold|  EstimatedCharges  |  10.0       |



### Control - What can you learn about how this environment was deployed?

The environment has three clusters, the first (default) has nothing in it. The other two have active services and 1 running task each.

ECS( Elastic Container Service ) is a highly scalable, high performance container management service that supports Docker containers and allows you to easily run applications on a managed cluster of Amazon EC2 instances. ECS eliminates the need for you to install, operate, and scale your own cluster management infrastructure. 
- With simple API calls, you can launch and stop Docker-enabled applications, query the complete state of your cluster, and access many familiar features like security groups, Elastic Load Balancing, EBS volumes, and IAM roles. 
- You can use Amazon ECS to schedule the placement of containers across your cluster based on your resource needs and availability requirements. 
- You can also integrate your own scheduler or third-party schedulers to meet business or application specific requirements. 


```sh
aws ecs list-clusters 
```

```json
{
    "clusterArns": [
        "arn:aws:ecs:us-east-1:849265715259:cluster/default",
        "arn:aws:ecs:us-east-1:849265715259:cluster/wp-example-fe-ecs-afcdf21",
        "arn:aws:ecs:us-east-1:849265715259:cluster/app-cluster-c52122b"
    ]
}
```

```sh
aws ecs describe-clusters --clusters arn:aws:ecs:us-east-1:849265715259:cluster/default arn:aws:ecs:us-east-1:849265715259:cluster/wp-example-fe-ecs-afcdf21 arn:aws:ecs:us-east-1:849265715259:cluster/app-cluster-c52122b --query 'clusters[].{clusterName: clusterName, registeredContainerInstancesCount: registeredContainerInstancesCount, runningTasksCount: runningTasksCount, pendingTasksCount: pendingTasksCount, activeServicesCount: activeServicesCount}' --output table
```

| activeServicesCount |        clusterName         | pendingTasksCount  |  registeredContainerInstancesCount  |  runningTasksCount  |
|---------------------|----------------------------|--------------------|-------------------------------------|---------------------|
|  2                  |  app-cluster-c52122b       |  0                 |  0                                  |  1                  |
|  1                  |  wp-example-fe-ecs-afcdf21 |  0                 |  0                                  |  1                  |
|  0                  |  default                   |  0                 |  0                                  |  0                  |

## Compute

### Compute - What sorts of services are used to run the application? Virtual instances? Containers?

The applications are run in containers controlled by the ECS (elastic container services). It is a highly scalable, performant management service that supports Docker containers.

#### Cluster: wp-example-fe-ecs-afcdf21

```sh
aws ecs list-services --cluster arn:aws:ecs:us-east-1:849265715259:cluster/wp-example-fe-ecs-afcdf21
```

```json
{
    "serviceArns": [
        "arn:aws:ecs:us-east-1:849265715259:service/wp-example-fe-ecs-afcdf21/wp-example-fe-app-svc-95499f4"
    ]
}
```

```sh
aws ecs describe-services --services arn:aws:ecs:us-east-1:849265715259:service/wp-example-fe-ecs-afcdf21/wp-example-fe-app-svc-95499f4 --cluster arn:aws:ecs:us-east-1:849265715259:cluster/wp-example-fe-ecs-afcdf21 --query 'services[].{serviceName: serviceName, loadBalancers: loadBalancers[].{targetGroupArn: targetGroupArn, containerName: containerName, containerPort: containerPort}, desiredCount: desiredCount, runningCount: runningCount, pendingCount: pendingCount, deployments: deployments[].{networkConfiguration: networkConfiguration, taskDefinition: taskDefinition}, deploymentController: deploymentController}' --output json
```

```json
[
    {
        "serviceName": "wp-example-fe-app-svc-95499f4",
        "loadBalancers": [
            {
                "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:849265715259:targetgroup/wp-example-fe-app-tg-3871f0d/8e1252b145a079c6",
                "containerName": "wp-example-fe-app-container",
                "containerPort": 80
            }
        ],
        "desiredCount": 1,
        "runningCount": 1,
        "pendingCount": 0,
        "deployments": [
            {
                "networkConfiguration": {
                    "awsvpcConfiguration": {
                        "subnets": [
                            "subnet-07972be9fde6f1568",
                            "subnet-02dac479c6c596843"
                        ],
                        "securityGroups": [
                            "sg-0e4cc49750bfab958"
                        ],
                        "assignPublicIp": "ENABLED"
                    }
                },
                "taskDefinition": "arn:aws:ecs:us-east-1:849265715259:task-definition/fargate-task-definition:2"
            }
        ],
        "deploymentController": {
            "type": "ECS"
        }
    }
]
```

#### Cluster: app-cluster-c52122b

```sh
aws ecs list-services --cluster arn:aws:ecs:us-east-1:849265715259:cluster/app-cluster-c52122b
```

```json
{
    "serviceArns": [
        "arn:aws:ecs:us-east-1:849265715259:service/app-cluster-c52122b/django-site-service-dc1f82d",
        "arn:aws:ecs:us-east-1:849265715259:service/app-cluster-c52122b/django-database-service-0e31783"
    ]
}
```

```sh
aws ecs describe-services --services arn:aws:ecs:us-east-1:849265715259:service/app-cluster-c52122b/django-site-service-dc1f82d arn:aws:ecs:us-east-1:849265715259:service/app-cluster-c52122b/django-database-service-0e31783 --cluster arn:aws:ecs:us-east-1:849265715259:cluster/app-cluster-c52122b --query 'services[].{serviceName: serviceName, loadBalancers: loadBalancers[].{targetGroupArn: targetGroupArn, containerName: containerName, containerPort: containerPort}, desiredCount: desiredCount, runningCount: runningCount, pendingCount: pendingCount, deployments: deployments[].{id: id, subnets: networkConfiguration.awsvpcConfiguration.subnets[*], securityGroups: networkConfiguration.awsvpcConfiguration.securityGroups[*], assignPublicIp: networkConfiguration.awsvpcConfiguration.assignPublicIp, taskDefinition: taskDefinition}, deploymentController: deploymentController.type}' --output json
```

```json
[
    {
        "serviceName": "django-site-service-dc1f82d",
        "loadBalancers": [
            {
                "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:849265715259:targetgroup/django-targetgroup-24e34d2/73867b73f1fdde85",
                "containerName": "django-container",
                "containerPort": 80
            }
        ],
        "desiredCount": 1,
        "runningCount": 1,
        "pendingCount": 0,
        "deployments": [
            {
                "id": "ecs-svc/4164589488824098278",
                "subnets": [
                    "subnet-09836c640047448e1"
                ],
                "securityGroups": [
                    "sg-0b5a53e845d09af1d"
                ],
                "assignPublicIp": "ENABLED",
                "taskDefinition": "arn:aws:ecs:us-east-1:849265715259:task-definition/django-site-task-definition-family:2"
            }
        ],
        "deploymentController": "ECS"
    },
    {
        "serviceName": "django-database-service-0e31783",
        "loadBalancers": [
            {
                "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:849265715259:targetgroup/django-targetgroup-24e34d2/73867b73f1fdde85",
                "containerName": "django-container",
                "containerPort": 80
            }
        ],
        "desiredCount": 1,
        "runningCount": 0,
        "pendingCount": 0,
        "deployments": [
            {
                "id": "ecs-svc/4040761822402673745",
                "subnets": [
                    "subnet-09836c640047448e1"
                ],
                "securityGroups": [
                    "sg-0b5a53e845d09af1d"
                ],
                "assignPublicIp": "ENABLED",
                "taskDefinition": "arn:aws:ecs:us-east-1:849265715259:task-definition/django_database_task_definition-family:2"
            }
        ],
        "deploymentController": "ECS"
    }
]
```


### Compute - Provide details of the compute entities that you discover

There are three fargate tasks in total, one for the wordpress based thoughtpad, one for the django site for polly and another for the django DB access. AWS Fargate is a serverless, pay-as-you-go compute engine that lets you focus on building applications without managing servers. AWS Fargate is compatible with both Amazon Elastic Container Service (ECS) and Amazon Elastic Kubernetes Service (EKS). 

#### Task Definition: fargate-task-definition

```sh
aws ecs describe-task-definition --task-definition arn:aws:ecs:us-east-1:849265715259:task-definition/fargate-task-definition:2 --query 'taskDefinition.{cpu: cpu, memory: memory, containerDefinitions: containerDefinitions[].{name: name, image: image, cpu: cpu, memory: memory, portMappings: portMappings[].{containerPort: containerPort, hostPort: hostPort, protocol: protocol}, environment: environment[*].{name: name, value: value}, awslogsGroup: logConfiguration.options.["awslogs-group"][0]}}' --output json
```

```json
{
    "cpu": "256",
    "memory": "512",
    "containerDefinitions": [
        {
            "name": "wp-example-fe-app-container",
            "image": "wordpress",
            "cpu": 0,
            "memory": null,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80,
                    "protocol": "tcp"
                }
            ],
            "environment": [
                {
                    "name": "WORDPRESS_DB_USER",
                    "value": "admin"
                },
                {
                    "name": "WORDPRESS_DB_HOST",
                    "value": "wp-example-be-rdsbce40b7.cq51gjhxucto.us-east-1.rds.amazonaws.com:3306"
                },
                {
                    "name": "WORDPRESS_DB_PASSWORD",
                    "value": "GYyR2rDLk34ltcKn"
                },
                {
                    "name": "WORDPRESS_DB_NAME",
                    "value": "lampdb"
                }
            ],
            "awslogsGroup": null
        }
    ]
}
```

#### Task Definition: django-database-task-definition-family

```sh
aws ecs describe-task-definition --task-definition arn:aws:ecs:us-east-1:849265715259:task-definition/django_database_task_definition-family:2  --query 'taskDefinition.{cpu: cpu, memory: memory, containerDefinitions: containerDefinitions[].{name: name, image: image, cpu: cpu, memory: memory, portMappings: portMappings[].{containerPort: containerPort, hostPort: hostPort, protocol: protocol}, environment: environment[*].{name: name, value: value}, awslogsGroup: logConfiguration.options.["awslogs-group"][0]}}' --output json
```

Note that because a field (awslogs-group) used a dash it was necessary to use the bracket notation to access it. This returned a list value which is why I am then accessing the first element.

```json
{
    "cpu": "256",
    "memory": "512",
    "containerDefinitions": [
        {
            "name": "django-container",
            "image": "849265715259.dkr.ecr.us-east-1.amazonaws.com/app-ecr-repo-709a364:29a8e93cea2bb1138479b1a982cff4dfa6a59b3288be9ae7572052ca69f9e73f",
            "cpu": 0,
            "memory": 512,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80,
                    "protocol": "tcp"
                }
            ],
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
            ],
            "awslogsGroup": "django-log-group"
        }
    ]
}
```

#### Task Definition: django-site-task-definition-family

```sh
aws ecs describe-task-definition --task-definition arn:aws:ecs:us-east-1:849265715259:task-definition/django-site-task-definition-family:2 --query 'taskDefinition.{cpu: cpu, memory: memory, containerDefinitions: containerDefinitions[].{name: name, image: image, cpu: cpu, memory: memory, portMappings: portMappings[].{containerPort: containerPort, hostPort: hostPort, protocol: protocol}, environment: environment[*].{name: name, value: value}, awslogsGroup: logConfiguration.options.["awslogs-group"][0]}}' --output json
```

```json
{
    "cpu": "256",
    "memory": "512",
    "containerDefinitions": [
        {
            "name": "django-container",
            "image": "849265715259.dkr.ecr.us-east-1.amazonaws.com/app-ecr-repo-709a364:29a8e93cea2bb1138479b1a982cff4dfa6a59b3288be9ae7572052ca69f9e73f",
            "cpu": 0,
            "memory": 512,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80,
                    "protocol": "tcp"
                }
            ],
            "environment": [
                {
                    "name": "DATABASE_PORT",
                    "value": "3306"
                },
                {
                    "name": "SECRET_KEY",
                    "value": "corise-django-secret-key"
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
                    "name": "USER_NAME",
                    "value": "coriseuser"
                },
                {
                    "name": "USER_PASSWORD",
                    "value": "corise-user-pass"
                }
            ],
            "awslogsGroup": "django-log-group"
        }
    ]
}
```

## Communication

### Communication - What service ports are in use by the application

The applications all use port 80 for their container and host.

### Communication - Are there any load balancers in use? If so, what type(s)?

There are two load balancers, a network one which handles traffic to Polly and an application one that handles traffic to Thoughtpad. (I used dig to discover the underlying DNS for both applications)

far```sh
aws elbv2 describe-load-balancers --region us-east-1 --query 'LoadBalancers[*].{LoadBalancerName: LoadBalancerName, LoadBalancerArn:LoadBalancerArn, DNSName:DNSName, Type:Type, VpcId:VpcId, SecurityGroups: SecurityGroups[*]  | [0]}' --output table
```

|                                DNSName                                |                                                  LoadBalancerArn                                                  |     LoadBalancerName       |    SecurityGroups     |    Type      |         VpcId          |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|----------------------------|-----------------------|--------------|------------------------|
|  django-balancer-ddf0f5f-74d469718e97d074.elb.us-east-1.amazonaws.com |  arn:aws:elasticloadbalancing:us-east-1:849265715259:loadbalancer/net/django-balancer-ddf0f5f/74d469718e97d074    |  django-balancer-ddf0f5f   |  None                 |  network     |  vpc-07ddda53af641648f |
|  wp-example-fe-alb-1112877-1287015604.us-east-1.elb.amazonaws.com     |  arn:aws:elasticloadbalancing:us-east-1:849265715259:loadbalancer/app/wp-example-fe-alb-1112877/d2c57dbccfa6eea6  |  wp-example-fe-alb-1112877 |  sg-0e4cc49750bfab958 |  application |  vpc-06eb467fa397406d9 |

`dig thoughtpad.acmeventures.net` : wp-example-fe-alb-1112877-1287015604.us-east-1.elb.amazonaws.com

`dig polly.acmeventures.com` : acmeventures.com


### Communication - What service ports and protocols are exposed by this application?

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


### Communication - What does the network configuration look like? List all subnets that you can find

Subnets are a way to segment your network within a VPC.

```sh
aws ec2 describe-subnets --query 'Subnets[].{SubnetId: SubnetId, State: State, VpcId: VpcId}' --output table
```

|   State   |         SubnetId           |          VpcId          |
|-----------|----------------------------|-------------------------|
|  available|  subnet-02d3a6d083b437ad3  |  vpc-07ddda53af641648f  |
|  available|  subnet-07972be9fde6f1568  |  vpc-06eb467fa397406d9  |
|  available|  subnet-09836c640047448e1  |  vpc-07ddda53af641648f  |
|  available|  subnet-02dac479c6c596843  |  vpc-06eb467fa397406d9  |

Network ACLs (NACLs) can be used to create rules that allow or deny traffic between subnets. Examine the ingress and egress rules associated with each NACL to determine if there are any specific rules controlling the traffic between subnets.

```sh
aws ec2 describe-network-acls --query 'NetworkAcls[].{NetworkAclId: NetworkAclId, VpcId: VpcId}' --output table
```

|      NetworkAclId      |          VpcId          |
|------------------------|-------------------------|
|  acl-006a77c2a16553f57 |  vpc-06eb467fa397406d9  |
|  acl-068a7a63df7af5c01 |  vpc-07ddda53af641648f  |

### Communication - Can you determine what network segmentation, if any, is in use?

Network segmentation is the practice of dividing a computer network into smaller, isolated subnetworks or segments. It is often used to improve network performance, enhance security, and simplify management. Network segmentation can be achieved through various techniques, such as creating subnets, implementing virtual local area networks (VLANs), using network access control lists (NACLs), or applying security groups.

The primary benefits of network segmentation are:

- Security: By dividing the network into smaller segments, you can isolate sensitive data and applications from the rest of the network. This limits the potential impact of a security breach by preventing unauthorized access to other parts of the network.
- Performance: Segmentation can help to reduce network congestion by limiting broadcast traffic and containing it within a specific segment. This allows for better utilization of network resources and improved performance.
- Management: Network segmentation makes it easier to manage and troubleshoot issues in the network. It allows administrators to apply specific policies, monitor traffic, and maintain each segment independently.
- Compliance: Many regulatory standards, such as PCI DSS and HIPAA, require network segmentation to protect sensitive data and systems.

Information about network segmentation can help you understand:

- How the network is organized: Segmentation reveals the structure of the network, including the various subnets, VLANs, and security zones.
- Traffic flow: By examining network segmentation, you can understand how traffic is allowed or restricted between different segments, which can help you identify potential bottlenecks or security risks.
- Access control: Network segmentation provides insight into the access controls applied to different parts of the network, such as security groups, NACLs, and firewall rules.
- Resource allocation: Understanding network segmentation can help you identify how resources are distributed across the network, enabling you to optimize resource allocation for better performance and cost-efficiency.

Overall, network segmentation provides valuable information about the organization, security, and performance of a computer network, helping you make informed decisions about its design, management, and optimization.



### Communication - What are the configured security groups?

```sh
aws ec2 describe-security-groups --query 'SecurityGroups[].{GroupName: GroupName, GroupId: GroupId, Description: Description}' --output table
```

|  GroupName                    |  GroupId               |  Description                 |
|-------------------------------|------------------------|------------------------------|
|  default                      |  sg-04f6b17fae6878018  |  default VPC security group  |
|  wp-example-net-rds-sg-ae7bf93|  sg-0a2c8507070927c39  |  Allow client access.        |
|  wp-example-net-fe-sg-e78826b |  sg-0e4cc49750bfab958  |  Allow all HTTP(s) traffic.  |
|  security-group-efe9341       |  sg-0b5a53e845d09af1d  |  Enables HTTP access         |
|  default                      |  sg-0e2eebe4acfea8d1a  |  default VPC security group  |

```sh
aws ec2 describe-security-groups --query 'SecurityGroups[].{GroupName: GroupName, Description: Description, IpPermissions: IpPermissions[].{FromPort: FromPort, ToPort: ToPort,IpProtocol: IpProtocol}}' 
```

```json
[
    {
        "GroupName": "default",
        "Description": "default VPC security group",
        "IpPermissions": [




            {
                "FromPort": null,
                "ToPort": null,
                "IpProtocol": "-1"
            }
        ]
    },
    {
        "GroupName": "wp-example-net-rds-sg-ae7bf93",
        "Description": "Allow client access.",
        "IpPermissions": [
            {
                "FromPort": 3306,
                "ToPort": 3306,
                "IpProtocol": "tcp"
            }
        ]
    },
    {
        "GroupName": "wp-example-net-fe-sg-e78826b",
        "Description": "Allow all HTTP(s) traffic.",
        "IpPermissions": [
            {
                "FromPort": 80,
                "ToPort": 80,
                "IpProtocol": "tcp"
            },
            {
                "FromPort": 443,
                "ToPort": 443,
                "IpProtocol": "tcp"
            }
        ]
    },
    {
        "GroupName": "security-group-efe9341",
        "Description": "Enables HTTP access",
        "IpPermissions": [
            {
                "FromPort": 0,
                "ToPort": 65535,
                "IpProtocol": "tcp"
            }
        ]
    },
    {
        "GroupName": "default",
        "Description": "default VPC security group",
        "IpPermissions": [
            {
                "FromPort": null,
                "ToPort": null,
                "IpProtocol": "-1"
            }
        ]
    }
]
```

### Communication - What are the VPCs associated with the applications?

VPC (Virtual Private Cloud) is a virtual network dedicated to your AWS account. It is logically isolated from other virtual networks in the AWS Cloud. You can launch your AWS resources, such as Amazon EC2 instances, into your VPC. You can specify an IP address range for the VPC, add subnets, associate security groups, and configure route tables.

```
aws ec2 describe-vpcs --query 'Vpcs[].{VpcId: VpcId, State: State, Tags: Tags[?Key==`Name`].Value | [0]}' --output table
```

|   State   |        Tags          |          VpcId          |
|-----------|----------------------|-------------------------|
|  available|  None                |  vpc-07ddda53af641648f  |
|  available|  wp-example-net-vpc  |  vpc-06eb467fa397406d9  |


### Communication - can you describe the end-to-end traffic flow from user to the application servers? Make sure to list all intermediate entities

Traffic from users is handled by the loadbalancer which distributes it between the instance/s of the application. Security groups and the NACLs allow the application to handle traffic between the load balancer and also to and from the RDS instance which is used as storage.

## Storage

### Storage - What types of storage are used by the applications? Break this down per service

There are two RDS instances, one for each application in use. For more details see below question.

### Storage - Provide the configuration details of each storage service (service ports, network configuration, e.t.c.)

```sh
aws rds describe-db-instances --query 'DBInstances[].{DBInstanceIdentifier: DBInstanceIdentifier, DBInstanceClass: DBInstanceClass, AvailabilityZone: AvailabilityZone, Engine: Engine, EngineVersion:EngineVersion, MasterUsername: MasterUsername, Address: Endpoint.Address, Port: Endpoint.Port, AllocatedStorage: AllocatedStorage, DBSubnetGroupName: DBSubnetGroup.DBSubnetGroupName, VpcSecurityGroups: join(`, `, VpcSecurityGroups[*].VpcSecurityGroupId), Subnets: join(`, `, DBSubnetGroup.Subnets[*].SubnetIdentifier), PubliclyAccessible: PubliclyAccessible}' --output table
```

|                              Address                              | AllocatedStorage  | AvailabilityZone  | DBInstanceClass  |   DBInstanceIdentifier    |         DBSubnetGroupName         | Engine  | EngineVersion  | MasterUsername  | Port  | PubliclyAccessible  |                       Subnets                        |   VpcSecurityGroups    |
|-------------------------------------------------------------------|-------------------|-------------------|------------------|---------------------------|-----------------------------------|---------|----------------|-----------------|-------|---------------------|------------------------------------------------------|------------------------|
|  mysql-server6e646b6.cq51gjhxucto.us-east-1.rds.amazonaws.com     |  20               |  us-east-1b       |  db.t2.micro     |  mysql-server6e646b6      |  app-database-subnetgroup-d4d12da |  mysql  |  8.0.28        |  coriseadmin    |  3306 |  True               |  subnet-02d3a6d083b437ad3, subnet-09836c640047448e1  |  sg-0b5a53e845d09af1d  |
|  wp-example-be-rdsbce40b7.cq51gjhxucto.us-east-1.rds.amazonaws.com|  20               |  us-east-1a       |  db.t2.micro     |  wp-example-be-rdsbce40b7 |  wp-example-be-sng-bda77ae        |  mysql  |  5.7.38        |  admin          |  3306 |  False              |  subnet-07972be9fde6f1568, subnet-02dac479c6c596843  |  sg-0a2c8507070927c39  |


## Optional

### How would you update the documentation for Thoughtpad or Polly to better reflect the infrastructure you explored? Are there any inaccuracies you would address?

I had downloaded the instructions the weekend before kick off and these instructions mentioned there being 2 ECR instances, however I noticed this was already updated in the documentation.

We can confirm the database for Polly is an RDS. 

We could also add the relevant subnet and security group information, though this might be too detailed for general use.

We could also break the architecture description into the CCS:

Control - ECS
Compute - fargate
Communication - Elastic loadbalancer (network/application) and NACLs
Storage - RDS



### How would you change the design of either project? Share why you would make the specific changes that you are proposing.




## Do these deployments share any commonalities with projects you have deployed or managed in real life?

Our services are deployed in AWS with our images saved in ECR. Our deployment set up is done via CD with infrastructure being provisioned with terraform. There is a lot of boilerplate that I as a developer can use and our kubernete deployments are generated from jsonnet manifests. We use loadbalancers to hand traffic into the cluster. I have provisioned security groups before but I have not done the complete set up so I am somewhat unsure where the differences are. We also use RDS instances for some of our storage.

