# tranmere-web-dynamic-page

This project was autogenerated using the Mephisto Yeoman template 

## Introduction

## Prerequisites

* SAM CLI - [Install the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
* Node.js - [Install Node.js 12x](https://nodejs.org/en/), including the NPM package management tool.
* Docker - [Install Docker community edition](https://hub.docker.com/search/?type=edition&offering=community)

## GitHub Actions

You'll need the following GitHub secrets in your repository - the deploy bucket is an s3 bucket for deploying/storing your Lambda packages

* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY
* AWS_DEPLOY_BUCKET

## Project Structure

This project includes the following building blocks:

   * A single JavaScript Lambda to return a hello world JSON response with the right CORS headers
   * A SAM template to deploy the Lambda and make it accesible to HTTP requests via an API Gateway
   * An example unit test
   * An example API test, used as a post deployment check

### The SAM Template

The SAM template includes definition for an API Gateway and a basepathmapping definition so that the this gateway will be mapped to a URI stem on a custom domain. This approach is similar to how we define RMS routes and allows a single custom domain to have multiple uristems, each mapping to a different API Gateway instance.

The API Gateway definition also defines a simple [OpenAPI](https://swagger.io/specification/) structure. In the deploy pipeline, this swagger file is uploaded to an s3 bucket with a unique id on for each deployment. This definition add information to the documentation sections of API Gateway. This specification can be used to generate SDK's from within API Gateway or via the AWS API portal.

### Running locally with the SAM CLI

Build your application with the `sam build` command.

```bash
$ npm run build
```

The SAM CLI installs dependencies defined in `DynamicPage/package.json`, creates a deployment package, and saves it in the `.aws-sam/build` folder.

Test a single function by invoking it directly with a test event. An event is a JSON document that represents the input that the function receives from the event source. Test events are included in the `events` folder in this project.

Run functions locally and invoke them with the `sam local invoke` command.

```bash
$ sam local invoke DynamicPage --event events/event.json
```

The SAM CLI can also emulate your application's API. Use the `sam local start-api` to run the API locally on port 3000.

```bash
$ sam local start-api
$ curl http://localhost:3000/
```

### Unit Testing

Tests are defined in the `tests` folder in this project. Use NPM to install the [Mocha test framework](https://mochajs.org/) and run tests.

```bash
$ npm run test
```

### API Tests

API tests are written using a POSTMAN collection in the `api_tests` folder and exceuted via [Newman](https://learning.postman.com/docs/running-collections/using-newman-cli/command-line-integration-with-newman/).


You'll need to start your local api first.
```bash
$ sam local start-api
```

In a separate terminal
```bash
$ npm install -g newman
$ newman run ./api_tests/postman-collection.json -e ./api_tests/env/aws-local.json
```

## Continuous Integration & Deployment

The project comes with a suggested CI/CD approach using [GitHub Actions](https://github.com/features/actions) and Git triggered deployments. You raise PR's off the master branch (which will kick off a series of validation steps including unit tests)

After merging to master you can then deploy to environment by merging to an environment branch, sit1,pre1,prod1 etc.

These have a delploy action triggered on merge which also runs any post-deployment checks using the newman tests. For non-prod enviromment deployments are done `AllAtOnce`, for production they use `Canary10Percent5Minutes` - see [here for more details](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/automating-updates-to-serverless-apps.html).

## Depoying to AWS Manually

If you have permissions to deploy to an AWS account for the CLI to build and deploy your application for the first time, run the following in your shell:

```bash
$ npm run build
$ sam deploy --stack-name DynamicPage --capabilities CAPABILITY_IAM --s3-bucket tranmere-web-api
```

### Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs generated by your deployed Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
$ sam logs -n DynamicPageFunction --stack-name DynamicPage --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

### Cleanup

To delete the sample application that you created, use the AWS CLI. Assuming you used your project name for the stack name, you can run the following:

```bash
$ aws cloudformation delete-stack --stack-name DynamicPage
```
