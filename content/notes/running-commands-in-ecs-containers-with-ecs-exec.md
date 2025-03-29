---
title: Running commands in ECS Containers with ECS Exec
date: 2025-03-29
categories: [AWS, ECS, SSM, SSH]
---

{{< admonition type="tip" title="TL;DR" >}}
[Amazon ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html) lets you securely interact with your running ECS containers (get a shell or run single commands) using [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html).
{{< /admonition >}}

[Amazon ECS Exec](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-exec.html) is a powerful feature that allows you to directly execute commands in or start an interactive shell session with a running container within an Amazon ECS task (on both EC2 and Fargate).

This is incredibly useful for debugging, running one-off administrative tasks (like database migrations), or inspecting the container's environment without needing to set up SSH or other complex access methods.

Under the hood, ECS Exec leverages [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) to establish a secure connection to the container.

## How to Set Up ECS Exec

To use ECS Exec, you need to configure a few things:

### Update IAM Task Role

The IAM role associated with your ECS task needs permissions to communicate with the SSM service. Add the following permissions to your task execution role policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssmmessages:CreateControlChannel",
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenControlChannel",
                "ssmmessages:OpenDataChannel"
            ],
            "Resource": "*"
        }
    ]
}
```

{{< admonition type="note" >}}
**Optionally**, if you want session logging, you might also need permissions for CloudWatch Logs (`logs:CreateLogStream`, `logs:PutLogEvents`, etc.) or S3 (`s3:PutObject`), depending on your SSM Session Manager logging configuration.
{{< /admonition >}}

### Enable Execute Command on the ECS Service

When creating or updating your ECS service, you need to explicitly enable the `Execute Command` feature. Using the AWS CLI, you'd add the `--enable-execute-command` flag:

```shell
aws ecs update-service --cluster <your-cluster-name> \
  --service <your-service-name> \
  --enable-execute-command \
  # ...
```

### Update the Task Definition

This was the crucial step I initially missed! You need to ensure the SSM agent running inside the container can function correctly.

For tasks using Linux containers, you must set `initProcessEnabled` to `true` within the `linuxParameters` of your container definition. This ensures an init process is started in the container, which is required by the SSM agent.

```json
"containerDefinitions": [
    {
        "name": "your-container-name",
        "image": "your-image",
        // ...
        "linuxParameters": {
            "initProcessEnabled": true
        },
        // ...
    }
]
```

Without `initProcessEnabled: true`, you might encounter errors like `TargetNotConnectedException`, as the SSM agent inside the container won't be properly initialized to receive commands.

## Using ECS Exec

Once configured, you can use the AWS CLI to execute commands. You'll need the cluster name, task ID (or ARN), and container name.

### Running a simple command

```shell
aws ecs execute-command \
    --region <your-region> \
    --cluster <your-cluster-name> \
    --task <your-task-id> \
    --container <your-container-name> \
    --command "whoami" \
    --interactive
```

This will connect to the specified container and run the `whoami` command.

### Starting an interactive shell session

To get a full shell inside the container:

```shell
aws ecs execute-command \
    --region <your-region> \
    --cluster <your-cluster-name> \
    --task <your-task-id> \
    --container <your-container-name> \
    --command "/bin/bash" \
    --interactive
```

This drops you into a bash shell running inside the container, allowing you to look around the filesystem, check environment variables, inspect running processes, etc.
