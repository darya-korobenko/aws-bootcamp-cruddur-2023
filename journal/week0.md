# Week 0 — Billing and Architecture

## Homework

### Create a new User and Generate AWS Credentials

- I used IAM User groups Console to create a new user group *admin*. During the set up of *admin* group I attached *AdministratorAccess* policy to it.
- Then I went to IAM Users Console to create a new user *daria-bootcamp*. During the set up of *daria-bootcamp* user I assigned it to the *admin* group.
- Following that, I started working on additional setting for *daria-bootcamp* user. I went to Security Credentianls tab and, just to be on a safe side, assigned MFA device to it. After that I created a new access key to be able to access my AWS account through CLI.

![Proof of AWS User](/_docs/assets/user.png)

I also added an alias to my AWS account though IAM Dashboard-AWS Account to make the sign-in for my IAM users easier in the future.

Last but not least, I went to Account settings and activated IAM access to billing information.

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

## Challenges

I set up MFA for my root account and stopped using it as my newly created *daria-bootcamp* user had all the required permissions.

![MFA for Root User](/_docs/assets/MFA_root.png)

I also created IAM role *DemoRoleForEC2* for EC2 instances and attached *IAMReadOnlyAccess* policy to it.

![IAM Role](/_docs/assets/role.png)

Use EventBridge to hookup Health Dashboard to SNS and send notification when there is a service health issue.
Review all the questions of each pillars in the Well Architected Tool (No specialized lens)
Create an architectural diagram (to the best of your ability) the CI/CD logical pipeline in Lucid Charts
Research the technical and service limits of specific services and how they could impact the technical path for technical flexibility. 
Open a support ticket and request a service limit
