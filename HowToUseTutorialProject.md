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

7) Go to DynamoDB:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/42.png "DynamoDB")

8) Click the "Create table" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/57.png "Create table")

9) Name the table "TestTable", name the partion key "UserId", add a sort key called "recNum", and click the "Create" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/58.png "Create table save")

10) It might take a few minutes for the table to be created. Once the spinning arrows stop, click the "Access Control" tab. Set the Identity provider to Facebook, put checkmarks for all the actions, and click the "Create Policy" button, then copy it to the clipboard:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/49.png "Access control")

12) Go to Identity & Access Management:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/6.png "Create table save")

13) Go to the Roles section. You should see an auth role and an unauth role. Click on the auth role (the top one):

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/59.png "auth role")

14) Click the "Create Role Policy" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/60.png "role policy")

15) Select "Custom Policy" and click the "Select" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/61.png "select policy")

16) Name the policy "Table_Policy", paste the text that you copied from DynamoDB, and click the "Apply Policy" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/62.png "apply policy")

TO DO--SHOW HOW TO SET UP THE SWIFT PROJECT TO DO THE WRITING TO DYNAMODB, PASTING THE FACEBOOK INFO, ETC.

(still working on this)
