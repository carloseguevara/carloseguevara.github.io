---
title: "How to execute an ECS task with more or less resources"
layout: post
author: "Carlos Guevara"
header-style: text
tags:
- AWS
- Python
- DataEngineering
---
To execute an ECS task definition exist three ways:

    - Run task with boto3
    - Run task from AWS page UI
    - Run task with Scheduled Task (Amazon EventBridge)

With any of these solutions, exist the `overrides` option to alter/override the defined resources in the task definition.

The normal way to run one task using the pre-defined resources in the task definition with `boto3` is:

```python
import json

import boto3

if __name__ == "__main__":
    ecs_client = boto3.client("ecs", region_name="us-east-1")

    response = ecs_client.run_task(
        taskDefinition="task_name",
        launchType="FARGATE",
        cluster="cluster_name",
        platformVersion="LATEST",
        count=1,
        networkConfiguration={
            "awsvpcConfiguration": {
                "subnets": [
                    "subnet-XXXX",
                ],
                "assignPublicIp": "ENABLED",
                "securityGroups": ["sg-XXXX"],
            }
        }
    )

    print(json.dumps(response, indent=4, default=str))

```

With the same example, you can add the `overrides` parameter, that according to the [official documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ecs.html#ECS.Client.run_task)

> It's a list of container overrides in JSON format that specify the name of a container in the specified task definition and the overrides it should receive. You can override the default command for a container (that's specified in the task definition or Docker image) with a command override. You can also override existing environment variables (that are specified in the task definition or Docker image) on a container or add new environment variables to it with an environment override.
Here is an important point, an ECS task could have different containers inside it. You have in one hand the resources use for the task and in the other hand the individual resources per each inside container.
So, the override parameter have the `cpu` and `memory` nested parameter that defined the "task resources", and inside the `containerOverrides` nested parameter you define per each container inside the task its resources. This is essential because the "task resources" must be equal or more than the sum of the containers resources, in other case the run command will give error.
In the `containerOverrides` nested parameter, you must define a list with one element per each container used inside the task that you want to change the default resources. You must specify at least the container name, with additional the override parameter which could be `command`, `environment`, `environmentFiles`, and the most important for our case `cpu` and `memory`.

Remember, the task resource must be equal or more than the sum of the container resources.

```python
ecs_client = boto3.client("ecs", region_name="us-east-1")

response = ecs_client.run_task(
    taskDefinition="task_name",
    launchType="FARGATE",
    cluster="cluster_name",
    platformVersion="LATEST",
    count=1,
    networkConfiguration={
        "awsvpcConfiguration": {
            "subnets": [
                "subnet-XXXX",
            ],
            "assignPublicIp": "ENABLED",
            "securityGroups": ["sg-XXXX"],
        }
    },
    overrides={
        "containerOverrides": [
            {
                "name": "container_name",
                "cpu": 512, # <- container cpu resource
                "memory": 3072, # <- container memory resource
            },
        ],
        "cpu": "512", # <- task cpu resource
        "memory": "3072", # task memory resource
    },
)

print(json.dumps(response, indent=4, default=str)

```
You can see the official AWS documentation for more details about the parameters:

    - https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_ContainerOverride.html
    - https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ecs.html#ECS.Client.run_task

> Note, if you define more resources for the task than the sum of the containers resources, you could have resources without use.