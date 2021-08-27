<!-- Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
SPDX-License-Identifier: MIT-0

Permission is hereby granted, free of charge, to any person obtaining a copy of this
software and associated documentation files (the "Software"), to deal in the Software
without restriction, including without limitation the rights to use, copy, modify,
merge, publish, distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE. -->

# Simpler Functionless URL Shortener

This app creates a URL shortener without using any compute. All business logic is handled at the Amazon API Gateway level. The basic app will create an API Gateway instance utilizing Cognito for authentication and authorization. It will also create an Amazon DynamoDB table for data storage. It will also create a simple Vuejs application as a demo client.

Read the blog series about this application:
1. [Building a serverless URL shortener app without AWS Lambda – part 1](https://aws.amazon.com/blogs/compute/building-a-serverless-url-shortener-app-without-lambda-part-1)
1. [Building a serverless URL shortener app without AWS Lambda – part 2](https://aws.amazon.com/blogs/compute/building-a-serverless-url-shortener-app-without-lambda-part-2)
1. [Building a serverless URL shortener app without AWS Lambda – part 3](https://aws.amazon.com/blogs/compute/building-a-serverless-url-shortener-app-without-lambda-part-3)

# About this fork

There is no Amplify, Cognito, Cloud Front. You will target the API directly.

Authentication is done using API Gateway API Keys. You must configure one and provide a key in your queries header using the `X-API-Key` header.

DynamoDB table now allows item auto expiration after one month. We add a new field `expiration` to all entries and set the table to use this field
to auto expire items.

`owner` is now set to `bFAN` and not dynamic as there is no more Cognito UserPool but you can change it directly in the `api.xml` file.

## The Backend

### Services Used
* <a href="https://aws.amazon.com/api-gateway/" target="_blank">Amazon API Gateway</a>
* <a href="https://aws.amazon.com/dynamodb/" target="_bank">Amazon DynamoDB</a>

### Requirements for deployment
* <a href="https://aws.amazon.com/cli/" target="_blank">AWS CLI</a>
* <a href="https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html" target="_blank">AWS SAM CLI v0.37.0+</a>
* Forked copy of this repository. Instructions for forking a GitHib repository can be found <a href="https://help.github.com/en/github/getting-started-with-github/fork-a-repo" target="_blank">here</a>
* A GitHub personal access token with the *repo* scope as shown below. Instructions for creating a personal access token can be found <a href="https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line#creating-a-token" target="blank">here</a>

    ![Personal access token scopes](./assets/pat.png)

    **Be sure and store you new token in a place that you can find it.**

### Deploying

In the terminal, use the SAM CLI guided deployment the first time you deploy
```bash
sam deploy -g
```

#### Choose options
You can choose the default for all options except *GithubRepository* and **

```bash
## The name of the CloudFormation stack
Stack Name [URLShortener]:

## The region you want to deploy in
AWS Region [eu-west-1]:

## The name of the application (lowercase no spaces). This must be globally unique
Parameter AppName [shortener]:

## GitHub forked repository URL
Parameter GithubRepository []:

## Github Personal access token
Parameter PersonalAccessToken:

## Shows you resources changes to be deployed and requires a 'Y' to initiate deploy
Confirm changes before deploy [y/N]: 

## SAM needs permission to be able to create roles to connect to the resources in your template
Allow SAM CLI IAM role creation [Y/n]:

## Save your choice for later deployments
Save arguments to samconfig.toml [Y/n]:
```

SAM will then deploy the AWS CloudFormation stack to your AWS account and provide required outputs for the included client.

After the first deploy you may re-deploy using `sam deploy` or redeploy with different options using `sam deploy -g`.

### Usage

You must include the `X-API-Key` Header in your http queries to authenticate with API Gateway. The Key is available in API Gateway `API Keys` section in the console.

The URLs to call are:

   * POST `shorter.bfansports.com/app`: Create a new short URL.
      * body: `{"id":"ABCDEFGHIJ12345", "url":"https://the.url.I.want.to.reach.com"}`
   * GET  `shorter.bfansports.com/${id}`: Get the URL associated with an ID
      * path: The ID of the URL to redirect to

Other calls are available. See the `api.yaml` file


