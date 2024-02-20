# Super GitHub Actions

A set of GitHub workflows we use to release software.

## Prerequisites

The assumption is that we have multiple task definitions and services running
on ECS clusters.

You will need an IAM role with the following permission policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:DescribeServices",
                "ecs:DescribeTaskDefinition",
                "ecs:DescribeTasks",
                "ecs:RegisterTaskDefinition",
                "ecs:RunTask",
                "ecs:UpdateService",
                "events:ListTargetsByRule",
                "events:PutTargets",
                "logs:FilterLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

This role will also require the following trust relationship:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<YOUR ACCOUNT ID WITHOUT DASHES>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:<GITHUB ACCOUNT>/<REPO NAME>:*"
                }
            }
        }
    ]
}
```

## run-deploy-task.yml

Runs the specified task, waits until completion, then displays the logs.
Intended for use with deploy actions.

Arguments:

* `aws-region`: The AWS region the cluster resides in.
* `cluster`: The name of the ECS cluster.
* `github-actions-role`: The ARN of the above role.
* `task-definition-family`: Name of the task definition family.

## update-scheduled-task.yml

Updates a scheduled task with the same name as a task definition to use
the latest version of the task definition.

Arguments:

* `aws-region`: The AWS region the cluster resides in.
* `github-actions-role`: The ARN of the above role.
* `task-definition-family`: Name of the task definition family.

## update-task-definition.yml

Updates a task definition to use a new ECR image from a private ECR repository,
and a service if one is provided. If the task definition contains multiple
containers, then all containers are updated to use the new image.

Arguments:

* `aws-region`: The AWS region the cluster resides in.
* `cluster`: The name of the ECS cluster.
* `github-actions-role`: The ARN of the above role.
* `image`: The new ECR image to use. Should not include the *.amazonaws.com/
prefix.
* `service`: Optional. The name of the service to update to use the new task
definition.
* `task-definition-family`: Name of the task definition family.
* `wait-for-service-stability`: Defaults to false. Whether to wait for the
service to reach stable state after deploying.
