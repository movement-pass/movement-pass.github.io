# CI/CD pipeline

Until we set up a full-blown CI/CD pipeline with CodeBuild, CodeDeploy and CodePipeline, lets use GitHub Actions as an
intermediary solution. Please note that from GitHub we are not going to apply any infrastructural changes only change in
the implementation will be deployed. So far we have two deployment unit a lambda function and a s3 bucket that serves
the React app. In order to deploy, an IAM user has been created which access and secret key would be used in the
deployment. I created an IAM policy with the following permissions and attached it with that IAM user:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "lambda:UpdateFunctionCode"
      ],
      "Resource": [
        "arn:aws:lambda:*:*:function:movement-pass*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::*.movement-pass.com/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:ListDistributions",
        "cloudfront:GetInvalidation",
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
```

Next, I used two of my GitHub
actions [aws-lambda-update-action](https://github.com/kazimanzurrashid/aws-lambda-update-action) and
[aws-static-web-app-update-action](https://github.com/kazimanzurrashid/aws-static-web-app-update-action) which does the
aws deployment.
