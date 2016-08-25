# Introduction
This project demonstrates how to log in to Facebook using Facebook's SDK, then pass the Facebook token to Cognito to receive a Cognito token, then write to a DynamoDB table using the Cognito token as the hash key.

It's pretty basic, but the Mobile Hub sample project glosses over so many steps and does so many "magical" things that it's hard to tell what is really happening (see my writeup in [README.md](README.md)). I wrote this to reduce it to the most basic steps possible so the user can understand what is going on.

## Steps to take

1) Make a Facebook app,  [as described here](README.md#creating-a-facebook-app-id).

2) Go to Cognito:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/23.png "Click on Cognito")

3) Click the "Manage Federated Identities" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/53.png "Federated Identities")

4) Click the "Create new identity pool" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/54.png "Identity pool")

5) Do these steps:
* Name the pool "HandleFacebook"
* Check the "Enable access to unauthenticated identities" option
* Paste your Facebook app's ID
* Click "Create Pool"

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/55.png "New pool")

6) Click "Allow" to this question. This will create the Roles for us in the IAM console. You could create the Roles yourself, but it's easier to let AWS create them since it will get the syntax correct.

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/56.png "Allow Cognito")

7) TO DO -- In DynamoDB, make a table

8) TO DO -- In IAM, delete the unauth role.

9) TO DO -- In IAM, make a policy for the login, and a policy for the table.

(still working on this)
