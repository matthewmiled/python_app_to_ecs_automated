# Intro

This project template builds on the deployment principles described in `python_app_to_k8s_automated`, but deploys to AWS Elastic Container Services using Fargate instead of Kubernetes/EKS.

# Workflow

1. Clone repo, create new local branch

2. Make desired changes to application

3. Push to new remote branch (git push -u origin <local-branch-name>). A new PR can be opened.

4. The test.yml workflow will then execute via GitHub actions (the trigger is a push to any branch apart from main). It will install python, install the dependencies and run pytest via a virtual Ubuntu machine.

5. If the tests pass, the PR can be approved and merged. A second workflow (build_and_deploy.yml) will trigger when it detects a merged PR. This workflow builds the docker image, pushes to AWS ECR, then deploys any application changes or scheduling changes to the task definition in ECR. Ensure that the ECS cluster is already set up by following the steps below.

# User Configuration

1. Create a AWS ECR and manually push a version of the application docker image to it (with a tag of `latest`). Make a note of the `ECR_REPOSITORY` name.

2. Create a cluster in ECS (using Fargate). Make a note of the `ECS_CLUSTER` name.

3. Create a task definition in ECS (using Fargate). In the container definition, point to the image defined above in ECR. Make a note of the container `name` and task definition name (aka `family`).

4. Optionally you can create a service in ECS (using Fargate) that uses the task definition defined above. This is used for if you want your application/container to run continuously. For this example project, we just want the application to run every hour, and this can be done with a cron-like scheduler instead. 
If you want to deploy to a service, see the additional [deploy steps](https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-amazon-elastic-container-service) needed in the yml file.

5. Set the `ECR_REPOSITORY`, `ECS_CLUSTER`, and `CONTAINER_NAME` to the relevant names within the environment variables section of `build_and_deploy.yml`.

6. Within `task-definition.json`, set the `family` param to the task definition name that you made. Set the `containerDefinitions.name` param to the same as `CONTAINER_NAME`.

7. Set the cluster `Arn` to yours in `scheduledtask.json`. Also change the account id in `RoleArn` to yours. You can set the `Id` to whatever you want.

8. Set your task definition ARN as the value for `TaskDefinitionArn` in `scheduledtask.json`. If you create new version of the task definition, make sure to use the latest version.

9. `scheduledtask.json` is used to set up the cron-like scheduled job. You need to change `Subnets` and `SecurityGroups` to your values. The easiest way to find these is to manually create a schedule rule in the 'Scheduled Tasks' section of your cluster in the AWS Console, and afterwards copy the subnet and security group values into the json file. The manually created schedule can then be deleted.

10. Change the cron schedule to your desired schedule at the bottom of `build_and_deploy.yml`. Note that the syntax is slightly different to cron, see details [here.](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) 
You can also change the rule `--name` to whatever you want.
__There are 2 commands - one for adding a schedule and one for removing. Comment out whatever one is not relevant for you. Note that you need to pass the `Id` of the schedule if you are turning off.__
