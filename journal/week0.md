# Week 0 — Billing and Architecture

## Homework

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

![Proof of Working AWS CLI](/_docs/assets/sanity-check.png)
