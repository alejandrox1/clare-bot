## clare-bot

![](https://github.com/clareliguori/clare-bot/raw/master/assets/robot.png)

The clare-bot application polls for GitHub notifications like @clare-bot mentions and performs actions.  For example, whitelisted GitHub users (namely, @clareliguori) can mention @clare-bot with a command "preview this" in a pull request to provision a preview environment.

Built with GitHub APIs, AWS Fargate, and Amazon CloudWatch Events

### Set up the bot

Create a GitHub user for your bot, like @clare-bot.  Create a [personal access token](https://github.com/settings/tokens) for the bot user with the following scopes:

* `repo` (Full control of private repositories)
* `notifications` (Access notifications)

Store the token in AWS Systems Manager Parameter Store:

```aws ssm put-parameter --region us-west-2 --name clare-bot-github-token --type SecureString --value <personal access token>```

Create an Amazon ECR repository, then build and push the Docker image:

```
ECR_REPO=`aws ecr create-repository --region us-west-2 --repository-name clare-bot --output text --query 'repository.repositoryUri'`
echo $ECR_REPO

$(aws ecr get-login --no-include-email --region us-west-2)

docker build -t clare-bot .

docker tag clare-bot $ECR_REPO

docker push $ECR_REPO
```

Provision the stack in CloudFormation:
```
aws cloudformation deploy --region us-west-2 --stack-name clare-bot --template-file template.yml --capabilities CAPABILITY_NAMED_IAM
```

### Test Locally

```
docker run --rm -v $HOME/.aws:/root/.aws:ro -e AWS_REGION=us-west-2 clare-bot
```