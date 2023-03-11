# Week 0 — Billing and Architecture

  - [Homework](#homework)
    + [Create a new User and Generate AWS Credentials](#create-a-new-user-and-generate-aws-credentials)
    + [Install and Verify AWS CLI](#install-and-verify-aws-cli)
    + [Enable Billing](#enable-billing)
    + [Create Billing Alarm](#create-billing-alarm)
    + [Create AWS Budget](#create-aws-budget)
    + [Recreate Conceptual Diagram in Lucid Charts](#recreate-conceptual-diagram-in-lucid-charts)
    + [Recreate Logical Architectual Diagram in Lucid Charts](#recreate-logical-architectual-diagram-in-lucid-charts)
  - [Challenges](#challenges)
    + [Set MFA for Root Acccount and Create IAM Role](#set-mfa-for-root-acccount-and-create-iam-role)
    + [Use EventBridge to hookup Health Dashboard to SNS](#use-eventbridge-to-hookup-health-dashboard-to-sns)
    + [Open Support Case](#open-support-case)
    + [Scrub Github History of Sensitive Data](#scrub-github-history-of-sensitive-data)

## Homework

### Create a new User and Generate AWS Credentials

- I used IAM User groups Console to create a new user group *admin*. During the set up of *admin* group I attached *AdministratorAccess* policy to it.
- Then I went to IAM Users Console to create a new user *daria-bootcamp*. During the set up of *daria-bootcamp* user I assigned it to the *admin* group.
- Following that, I started working on additional setting for *daria-bootcamp* user. I went to Security Credentianls tab and, just to be on a safe side, assigned MFA device to it. After that I created a new access key to be able to access my AWS account through CLI.

![Proof of AWS User](/_docs/assets/user.png)

I also added an alias to my AWS account though IAM Dashboard-AWS Account to make the sign-in for my IAM users easier in the future.

### Install and Verify AWS CLI 

I installed the AWS CLI on Gitpod, following a set of commands provided in [AWS CLI Install Documentation Page](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). To make it easier to debug the CLI commands, I also set AWS CLI to use partial autoprompt mode.

To automate these manual steps so that a new environment can be set up repeatedly by Gitpod, I defined *tasks* in the *.gitpod.yml* configuration file:
```
tasks:
  - name: aws-cli
    env:
      AWS_CLI_AUTO_PROMPT: on-partial
    init: |
      cd /workspace
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      cd $THEIA_WORKSPACE_ROOT
```

Following that I set my AWS credentials as environment variables, which let me have the persistent variables in my workspace that could be used in the code:
```
gp env AWS_ACCESS_KEY_ID="___"
gp env AWS_SECRET_ACCESS_KEY="___"
gp env AWS_DEFAULT_REGION="eu-central-1"
```

After that I verified that AWS CLI is working with the expected user:
```
aws sts get-caller-identity
```
Indeed, everything was set up correctly:

![Proof of Working AWS CLI](/_docs/assets/sanity-check.jpg)

Just in case of a future need, I also installed AWS CLI on my laptop by following the same instructions sited earlier but for Windows OS.

![Proof of Working AWS CLI v2](/_docs/assets/command_prompt.png)

### Enable Billing

I went to Account settings in my root account and activated IAM access to billing information.

I also opened Billing page and under Billing preferences chose Receive Billing Alerts specifying my email.

### Create Billing Alarm

I created SNS Topic called *billing-alarm* using Gitpod:
```
aws sns create-topic --name billing-alarm
```
Then I subscribed my email to this topic:
```
aws sns subscribe \
    --topic-arn XXX \
    --protocol email \
    --notification-endpoint XXX@gmail.com
```

Then I confirmed the subscription via email and checked that everything works with the help of AWS Console:

![Billing Alarm](/_docs/assets/billing_alarm.png)

Following [AWS Documentation Page](https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-estimatedcharges-alarm/), I created a billing alarm called *DailyEstimatedCharges* that would be triggered in case the daily estimated charges exceed 1$.

First, I had to create *alarm_config.json* file. The only modifications done in the file were changing the amount of dollars to 1 and adding the TopicARN that was generated earlier.

Then the following command was run:
```
aws cloudwatch put-metric-alarm --cli-input-json file://aws/json/alarm_config.json
```

Following that I confirmed that everything works with the help of AWS Console:

![Proof of AWS Billing Alarm](/_docs/assets/cloudwatch.png)

### Create AWS Budget

I followed [AWS Documentation Page](https://docs.aws.amazon.com/cli/latest/reference/budgets/create-budget.html#examples) to set up a new monthly AWS budget of 1$ using Gitpod.

In order to do that, two new json files were created as described in the documentation (only an amount of dollars was modified and email replaced).

To get my AWS Account ID the following command was run:
```
aws sts get-caller-identity --query Account --output text
```

Then I created a new environmental variable *AWS_ACCOUNT_ID*:
```
gp env AWS_ACCOUNT_ID="___"
```
After that the following *create-budget* command was run and my monthly budget was set up:

![AWS Budget](/_docs/assets/create_budget.png)

Following that I confirmed that everything worked with the help of AWS Console:

![Proof of AWS Budget](/_docs/assets/budget.png)

### Recreate Conceptual Diagram in Lucid Charts

![Conceptual Diagram](/_docs/assets/conceptual_diagram.png)

[Lucid Charts Share Link](https://lucid.app/lucidchart/c2c6acfa-134c-42ee-8c86-9fe74ad92a0e/edit?viewport_loc=-2528%2C-397%2C3072%2C1545%2C0_0&invitationId=inv_979c4aef-86a4-4b39-adb9-c934c197d36b)

### Recreate Logical Architectual Diagram in Lucid Charts

![Logical Diagram](/_docs/assets/logical_diagram.png)

[Lucid Charts Share Link](https://lucid.app/lucidchart/34db2c23-3489-4730-b85c-d52518f2be5f/edit?viewport_loc=-56%2C-76%2C2304%2C1159%2C0_0&invitationId=inv_e7b4d2ba-97b6-446c-8567-dc6dd65adddd)

## Challenges

### Set MFA for Root Acccount and Create IAM Role

I set up MFA for my root account and stopped using it as my newly created *daria-bootcamp* user had all the required permissions.

![MFA for Root User](/_docs/assets/MFA_root.png)

I also created IAM role *DemoRoleForEC2* for EC2 instances and attached *IAMReadOnlyAccess* policy to it.

![IAM Role](/_docs/assets/role.png)

### Use EventBridge to hookup Health Dashboard to SNS

Following a set of instructions provided in [AWS Documentation Page](https://aws.amazon.com/premiumsupport/knowledge-center/sns-email-notifications-eventbridge/), I created a new health topic in SNS and subscribed my email to it.

![SNS health topic](/_docs/assets/sns.png)

Following a set of instructions provided in [AWS Documentation Page](https://docs.aws.amazon.com/health/latest/ug/cloudwatch-events-health.html), I used EventBridge to hookup Health Dashboard to SNS, so that in the future I will be able to receive notifications whenever there is a service health issue.

![Eventbridge health rule](/_docs/assets/health.png)

### Open Support Case

With the help of AWS Support Console, I opened a new support ticket and requested to increase the service limit for SNS.

![Support case](/_docs/assets/support_case.png)

### Scrub Github History of Sensitive Data

I accidentally revealed my sensitive data in Github, so had to clean up the Github history.

Because of that I also had to deactive my secret access key and create a new one, as a result the variables in Gitpod had to be reset as well.

At first, I used [truffleHog](https://github.com/trufflesecurity/trufflehog) to scan GitHub repository and look into my commit history for any sensitive data:
```
brew install trufflesecurity/trufflehog/trufflehog
trufflehog git https://github.com/trufflesecurity/test_keys --only-verified
```

After that I used [BFG tool](https://rtyley.github.io/bfg-repo-cleaner/) to cleanse bad data out of my Git repository history. For that a new *passwords.txt* file had to be created with all the data that had to be removed (keys, passwords, emails). Then the following set of commands were run:
```
brew install bfg
bfg --replace-text passwords.txt
git reflog expire --expire=now --all && git gc --prune=now --aggressive
bfg --delete-files sanity-check.png
git reflog expire --expire=now --all && git gc --prune=now --aggressive
git push --force
```

Following that I verified that all the sensitive data was indeed removed:

![Sensitive data](/_docs/assets/sensitive_data.png)
