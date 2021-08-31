# Public Web

[React public web repository](https://github.com/movement-pass/https://github.com/movement-pass/react-public-web)

The public facing application is implemented in React and written in JavaScript. There are no state management library (
e.g. redux,recoil) is used. Since I am not a UI/UX Guy I need a rich component suite that is the
reason [Material-UI](https://material-ui.com) is used with default theme. This is a very basic React application nothing
special that I should explain. Yes one issue that there is no test and If you recommend instead of pure react, Next.js
or Gatsby would be better the candidate, I would not disagree with you.

## Stack

Like the photos stack in foundation this will be responsible to create a S3 bucket to store the react app build output,
creates a CloudFront distribution to serve the React app from that bucket and finally associates the `movement-pass.com`
domain with the CloudFront. The CDK also creates an additional lambda to deploy the build output to S3 bucket and
invalidate CloudFront cache when bucket content is copied. This lambda will be executed whenever we do a deployment
with `cdk deploy` of this stack.
