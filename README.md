# CodeBuild Demo with Github, Maven, SAM and Lambda

AWS offer two overlapping tools that help with Continuous Integration (CI) and Continuous Delivery / Continuous Deployment (CD) - [CodeBuild](https://aws.amazon.com/codebuild/) and [CodePipeline](https://aws.amazon.com/codepipeline/) .
To read an introduction to these two technologies, please see our primer [here](https://blog.symphonia.io/continuous-integration-continuous-delivery-on-aws-9b0d9cfe2f76).

This project is a demonstration of using CodeBuild, in isolation (i.e. not embedded within CodePipeline.)
CodeBuild is especially useful as a standalone tool when performing implementing CI, however it can also be used as the driver of a CD system too.
This demonstration performs CD, but it's simple to strip out the "deploy to production part".

This demonstration makes use of a few other common technologies:

* Source control is Github
* The application is implemented in Java, using Maven as a build tool
* AWS Lambda is used as the compute platform
* Deployment packaging is implemented with AWS SAM

## What this demonstration does

This project shows the use of CodeBuild to drive building, testing, and deployment of a multi-module Maven project.

To use this demo:

* Copy the project to your own Github repo.
* Update at least the `SourceLocation` setting at the top of the `codebuild.yaml` file.
* Use, or create, a Github Personal Access Token.
This is described for CodePipeline [here](https://docs.aws.amazon.com/codepipeline/latest/userguide/GitHub-create-personal-token-CLI.html) - it's the same mechanism for CodeBuild.
* Deploy the CodeBuild project as follows to AWS. You'll need the AWS CLI configured with appropriate permissions, you should override `=**YOUR_GITHUB_PERSONAL_ACCESS_TOKEN**` with your token, and feel free to use a different value for `--stack-name`:

```
$ aws cloudformation deploy \
    --parameter-overrides GitHubToken=**YOUR_GITHUB_PERSONAL_ACCESS_TOKEN** \
    --template-file codebuild.yaml \
    --stack-name codebuild-github-example \
    --capabilities CAPABILITY_IAM
```

Once CloudFormation has completed deploying this stack your CodeBuild project will be available, if everything has worked correctly.
You can kick off a first build by committing a change to your source repo, or by triggering a build manually from the CodeBuild console.

## How this Demo works

_I'd like to get into more detail on this at some point, but here's a summary_ :

* The application is a multi-module maven project. There are two "top-level" modules that contain the code, each for one Lambda
* Unit tests are defined, which are run as part of the `mvn package` goal (and others, but we use `package` here)
* `mvn package` will produce zip files for each of the two top level modules
* The application itself is an example of a data pipeline - you can see the structure in the SAM `template.yaml` file.
* This application comes from a forthcoming book that we've written ... we'll explain more in the book - keep your eyes peeled on [our twitter](https://www.twitter.com/symphoniacloud) to see when that's released! 
* CodeBuild configuration is defined in `codebuild.yaml`, and `buildspec.yml` . We're using various useful CodeBuild behaviors here, e.g. caching
* CodeBuild performs three application level steps
  * `mvn package` to compile, run unit tests, and produce artifacts. If any of these fail, the build will fail.
  * `sam package` to create a CloudFormation package in S3 with all of our resources.
  * `sam deploy` to perform the actual deployment.
* If you just wanted to perform CI you could delete the last two steps, and you'd be good to go
* We cache maven and python artifacts - this speeds the build up by more than a minute

## Costs

CodeBuild has costs (see https://aws.amazon.com/codebuild/pricing/) but includes 100 free minutes per month of the smallest instance type, which this project uses.

## Teardown

To tear down the demo delete the application stack in Cloudformation, and then the CodeBuild stack. Make sure to do so in that order.
