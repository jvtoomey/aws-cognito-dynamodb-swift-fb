# Introduction
This project demonstrates how to log in to Facebook using Facebook's SDK, then pass the Facebook token to Cognito to receive a Cognito token, then write to a DynamoDB table using the Cognito token as the hash key.

It's pretty basic, but the Mobile Hub sample project glosses over so many steps and does so many "magical" things that it's hard to tell what is really happening (see my writeup in [README.md](README.md)). I wrote this to reduce it to the most basic steps possible so the user can understand what is going on.

## Steps to take
The first step is to make a Facebook app,  [as described here](README.md#creating-a-facebook-app-id).




1) Make a Facebook app
2) In Cognito, make an AWS Identity pool
3) In DynamoDB, make a table
4) In IAM, make a role
5) In that role, make a policy for the login, and a policy for the table

(still working on this)
