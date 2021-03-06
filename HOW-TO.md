About
===

The elastic nature of the AWS cloud enables customers to access large compute capacity no sooner than needed, and let go of the same once done.
Moreover, the use of automation makes it possible to configure, operate, and change the form and function of such compute fleets in a seamless fashion.
With pricing options such as spot, customers gain access to compute capacity at low cost. 

Load testing with spot fleet
===

Current limitations
====

- All of the services and templates are deployed out of the AWS "us-east-1" region
- See [TODO.md](TODO.md) for more

Overview
====

Generating high load for performance tests requires large compute capacity for limited periods of time. 
We use the popular and versatile open-source testing tool: JMeter; to demonstrate the use of an elastic fleet of EC2 instances to run a performance test

- The load is generated by a fleet of inexpensive EC2 instances provisioned as part of a [[Spot Fleet]](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-fleet.html). Each instance acts as a "slave node" and runs JMeter in "server" mode for [["Distributed Testing"]](http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html) 
- Another EC2 instance runs JMeter (as "master controller") to co-ordinate the test run on the "slave nodes"
- The test software binaries, test plan, test configuration, and any utility scripts are published to version control (and subsequently stored in S3, to be retrieved by the test fleet) 
- We use automation with CloudFormation to provision the environment, CodeBuild to prepare and copy various artifacts, and CodePipeline to provision the various elements of the test fleet
- Logs from JMeter on the master controller instance are sent to CloudWatch Logs, and can be viewed in real time

Organization
====

- [environments](environments): contains CloudFormation templates, example parameter lists, and description of how multiple stacks are organised and should be used together
- [downloads](downloads): contains the software binaries for JMeter
- [test-plans](test-plans): contains the JMeter test plan (.jmx) and test run properties file
- [utility](utility): contains scripts to perform some utility tasks
- [buildspec.yml](buildspec.yml): a build specification file for use with CodeBuild

Setup and run tests
====

- Populate the information required for the various CloudFormation stacks in the parameter files
- Update test-plans/test-run.properties to customise test runs
- Commit the contents to a version control repository (such as [[CodeCommit]](https://aws.amazon.com/codecommit/))
- Create a build project for the repository in [[CodeBuild]](https://aws.amazon.com/codebuild/)
- Create the bucket in S3 that will store test artifacts
- Setup a pipeline in [[CodePipeline]](https://aws.amazon.com/codepipeline/)
- Monitor the progress of tests using CloudWatch Logs
- Obtain the test results from the controller instance

Pipeline stages and actions
====

Source stage:
Source action:

- detect changes in version control

Build stage: 
CodeBuild action:

- copy test artifacts into S3
- prepare and publish artifacts required for CloudFormation stacks

Provision-test-fleet stage:
provision-pre-requisites action:

- create stack with common elements for spot fleet and test runner instance

provision-spot-fleet action:

- create spot fleet request and bootstrap spot fleet instances with test artifacts

provision-test-runner action:

- provision test runner instance, copy test artifacts, and run test, send JMeter logs to CloudWatch logs

get-test-results action:

- copy test results from test-runner instance and approve to de-provision all of the test instances

cleanup stage:
remove-test-fleet action:

- delete test runner and spot fleet stacks
