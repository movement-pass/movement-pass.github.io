# Movement Pass

## Live version

* [Public facing application](https://movement-pass.com)

## Background

Few months ago Bangladesh Government released an application where we can register and apply for movement pass during
the lock-down period for necessary reason. But unfortunately it was suffering scalability issues (I had to refresh the
web page handful of times just to complete the registration process, let alone the pass apply). It was told that on the
very first day of its launch it was hit by 160 million requests, though it was not clear whether it was simple website
hit or people applying for passes. Nevertheless, it is a scalability challenge, to my knowledge none of the local
company or I believe the Government has the infrastructure to withstand such load. This seems a perfect candidate for a
cloud hosted solution and in cloud both kubernetes and serverless are taking the industries by storm. Here we are going
to explore the AWS serverless services that can we can use to create a scalable solution. Some of you may ask why not
kubernetes instead of serverless, well the answer to it is that it is not an either/or thing, you can have both
kubernetes and serverless on the same solution, but working with startups in last 5-6 years where resources(both
financial and knowledge) are limited, I find serverless solution is more viable comparing to kubernetes as it is easier
to get started, almost no administration, auto and fast scaling, faster time to market, pay per usage which applies very
much in startup environments. But it does not mean that serverless solution have limitation and shortcomings, in this
solution we are going to explore those limitations and issues of serverless solution and how to overcome those
challenges.

## Features

As user of the original Movement Pass application, we may have seen the public facing portion, but large chunk of it is
not known to us. But with our logical thinking we can assume, it should have:

* The public facing part (the website which is already available though a mobile application would be more appropriate).

* Approve/Rejection process, maybe human are involved or it may be completely automated.

* The verification, a mobile application for police department on the road for pass verification.

* Management Queries.

## Roadmap

Rather than creating the whole solution in a single go, we are going to create it incrementally in the following order:

- [x] Public facing application
- [ ] Approval process
- [ ] Management dashboard
- [ ] Verification application

On the technical side we are going to start with single AWS region and in the end we are going to refactor our
architecture to make it a multi region active active solution to address the scalability and availability issues.

## AWS Services

Here are the List of AWS services that I am planning to use and one or more of these services will be added to the
solution incrementally as we progress to completion.

- [x] S3
- [x] CloudFront
- [x] Lambda
- [x] API Gateway
- [x] DynamoDB
- [x] Systems Manager Parameter Store
- [ ] SQS
- [ ] Kinesis Data Stream
- [ ] Kinesis Firehose
- [ ] Athena
- [ ] Cognito
- [ ] Amplify
- [x] X-Ray
- [ ] CodeBuild
- [ ] CodeDeploy
- [ ] CodePipeline
- [x] Route53
- [x] CDK

## Implementation

I am starting with both NodeJS and .NET and do have plan to implement it in Golang as well.

As mentioned in the above, that we are going to incrementally implement the features and as we progress and new
documents would be added in the following section:

* [foundation](/foundation.md)
* [public-api](/public-api.md)
* [public-web](/public-web.md)
