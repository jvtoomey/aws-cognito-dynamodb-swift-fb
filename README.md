# Introduction
I am developing an iOS app in Swift and wanted these features:

* __Authenticate with Facebook:__ Rather than deal with managing user accounts and the attendant password reset issues, lockouts, etc, I liked the idea of having them log in using their Facebook account, similar to how LinkedIn allows. 
* __Save to Account:__ Once I've authenticated them, I wanted them to save table records that only they can update or delete.
* __Query other Users' Data__: I wanted them to be able to search each other's records. 

I considered getting a cheap domain on some host like GoDaddy and writing a web service in PHP, but it gets complicated when you start adding in the authentication part. I could see this project detouring into a months-long creation of the web service when I really wanted to be working on the app itself. 

After doing some internet research, it quickly became clear that Amazon Web Services (AWS) has a great reputation for abstracting a lot of this plumbing. Interestingly, I came across a lot of articles from 2013-2015 saying, "Facebook Parse is the best product since sliced bread!", followed by a lot of articles from 2016 saying, "Boo hoo, Parse is shutting down, better use AWS." It seemed that not only is AWS one of the better cloud providers, but also one of the few with staying power. Since AWS lets you create a free account, and currently has a generous free trial period, I figured it was worth a try.

## My first experience with AWS
You know that expression "you know so little that you don't even know what you don't know?" Well, that was me with AWS, and now I've graduated to knowing exactly how little I know. AWS is a massive service monster. There are so many tools available that it's hard to figure out what pieces you need to accomplish your goal. When you create your free account, you're presented with a control panel that resembles that of a Boeing 747. There are some catchy names for the various services (Snowball! Redshift! CloudTrail!), and lots of Lego-esque geometric icons, but not much guidance about which ones you might need.

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/1.png "AWS Control Panel")

## Mobile Hub
To help mobile developers in my situation, AWS created a service called Mobile Hub. You'll find it tucked away here on the right side:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/2.png "Mobile Hub")

It's kind of misleading to call Mobile Hub a "service," because it's really just a wizard that creates things for you within the other AWS services, like tables in DynamoDB or User Pools in Cognito. I have mixed feelings about the Mobile Hub. Like any wizard, it hides the reality of what it's really doing to make things easier. At first this is great, because it gives you a starting point. However, I found that pretty quickly it was just confusing me more, because I needed to know what was really happening, and didn't realize that it's not a self-contained service, but rather is reaching out to DynamoDB and Cognito and creating things in them. Once you know this, it's much more clear, but it's not at all obvious. The most useful thing about the Mobile Hub is that it bridges the gap between what you want to accomplish and knowing where in the AWS console to do it.

The Mobile Hub also does something which is a pretty impressive trick--it creates a sample Swift project that uses the AWS services like Cognito and DynamoDB that the Mobile Hub invisibly set up for you. Unfortunately, I didn't find the sample project that helpful, because it accomplishes things differently than many of the code samples I found on the web, plus it didn't have the current version of the Facebook SDK when I tested it. In other words, it had references to the Facebook SDK's files that were out of date compared to the SDK files you get when you go directly to Facebook and download them there. I found it easier in the long run to accomplish things piece by piece--first by figuring out how to log in to Facebook using Facebook's SDK, then figuring out how to pass the Facebook token to Cognito, then figuring out how to write to a DynamoDB tabe.

To give an example of how the Mobile Hub creates items in the AWS console, I'll show you what happens when you click on the Mobile Hub for the first time. You're asked to grant permissions to the Mobile Hub:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/3.png "Permissions")

After clicking "Yes, grant permissions," you'll see the swirling arrows for a few seconds before it continues (get used to these swirling arrows--many actions on AWS seem to be accompanied by a good 5-10 second wait, which feels weirdly laggy, although this could be the impatience of someone used to the instantaneousness of web apps like Gmail).

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/4.png "hourglass")

You're now presented with this screen:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/5.png "new project")

Remember how I said that Mobile Hub creates things on your behalf? Well, in saying yes to the granting of permissions, it's already done so. Under the "Identity & Access Management" service, 

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/6.png "IAM")

it has created a new Role called "MobileHub_Service_Role":

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/7.png "Role")

The "Identity & Access Management" service is usually abbreviated "IAM" in most of the AWS documentation, which is confusing because the term IAM doesn't appear anywhere on the AWS console. For a while I didn't even realize you could get to it from the AWS console, and always resorted to Googling the terms "AWS IAM Console" to get a link to it. 

The IAM service is hugely important to everything I do for my mobile app, but very different from anything I'd encountered before so I took a while to understand it at all. I'll talk about it more later on, but remember that IAM is crucial for making things work right.

## Creating a new Mobile Hub app

Even though I still feel in the long run you're better off without the Mobile Hub, it's useful when you're completely new to AWS because it shows you where in the AWS console to set up services. I'll show you what happens when you create a new app using the Mobile Hub. Start by giving it a name:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/8.png "New project")

This screen appears, 

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/9.png "Mobile Hub options")

which lets you choose the features that you want your app to have. The fine print at the bottom of each button tells you which AWS service it will create items in. For example, in my app I want users to log in with Facebook, then use that authentication in AWS to save records to a table, so I will be using the "User Sign-in" and the "NoSQL Database" buttons. Since these buttons say they're powered by "AWS Cognito" and "AWS DynamoDB," what's really going to happen is the Mobile Hub wizard is going to create items in these 2 services on my behalf.

I'm going to start by clicking the User Sign-in button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/10.png "User sign-in")

I'm given choices about how I want my users to log in. For my app, I want users to log in if they want to save items, but I also want them to be able to search for items without necessarily needing to be logged in, so I'm choosing "sign-in is optional":

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/11.png "Log-in choices")

I want to use Facebook as the log-in choice, so I click Facebook. Note how it asks for the Facebook App ID.

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/12.png "Adding Facebook login")

## Creating a Facebook App ID

You need to create a Facebook App ID on the Facebook developer's page, not anywhere in AWS. This isn't something the Mobile Hub (or AWS, for that matter) can do for you--instead, you have to go to 

https://developers.facebook.com/ 

and click the "Add a New App" link over there:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/13.png "Add Facebook app")

It'll ask what platform you're using. I'm using iOS so I click that:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/14.png "App type")

Give the app any name you want. It doesn't need to match your AWS Mobile Hub project name. *You can't give it a name that uses "Facebook" or "fb" or any name that Facebook deems too similar to their own. If you try it, it will stop you.*

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/15.png "App name")

Give it a category and a contact email:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/16.png "Email")

Pass the CAPTCHA:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/17.png "CAPTCHA")

Download the SDK:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/18.png "SDK")

This is the app ID that you will need to put in the AWS console. You will also need to put this in your app's info.plist. I'll show that later:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/19.png "App ID")

Now that you have the Facebook App ID, you don't really need to come back here.

## Connecting the Facebook App and AWS

In the AWS console, paste the Facebook App ID and click the "Save Changes" button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/20.png "Paste App ID")

It should tell you that the changes have been saved. Now click "Configure more features":

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/21.png "More features")

Note how the User Sign-In shows you the green checkmark that it's been configured: 

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/22.png "More features")

Now, what really just happened is that the Mobile Hub has gone to the Cognito service and added some settings for us. To explore these, go to the AWS console and click on Cognito:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/23.png "Click on Cognito")

## Cognito

Cognito has 2 different ways to handle logins, as you can see from the picture below:

* User Pools
* Identity Pools (labeled as "Federated Identities" on the button)

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/24.png "Cognito screen")

Part of what makes Cognito confusing in my opinion is the user interface, starting with the wording on those first buttons. By not using the word "pool" on both buttons, it seems to imply that they're completely different things. Interestingly, I noticed that much of the AWS documentation uses the term "identity pool," not "federated identity," so it does seem to be their preferred term once you get beyond this initial button.

Okay, so here's my understanding of these 2 types of authentication pools:

1) __User Pools__: You use these when you want the person to create an account that is stored at Amazon. It's not the same as an Amazon.com account that they might use to order books or jump ropes. Rather, this is an account that will be created specifically for any AWS services that you are providing them to use. The nice thing about these user pools is that AWS enforces the password rules and account resets for you. I'll show a picture below with more detail. 

2) __Identity Pools (aka Federated Identities)__: You use these pools when you want the user to log in using their credentials from a provider such as Google, Facebook, or Twitter. 

To quote Wikipedia:

> A federated identity in information technology is the means of linking a person's electronic identity and attributes, stored across multiple distinct identity management systems.

Remember back on the Mobile Hub screen when I supplied my Facebook App ID and told it that I'd like my app to have Facebook as a log-in option, but for it not to be required? Here's a good example of what the Mobile Hub is *really* doing behind the scenes when you choose this option. If I open Cognito, then click the "Manage Federated Identities" button, note that there is a identity pool there:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/25.png "Identity pool")

It has an auto-generated name, but it's pretty apparent that the name derives from the Mobile Hub app that I created: It begins with a lower-case version of the name "TestThisGizmo," then "_MOBILEHUB_", then a unique ID number. For curiosity's sake I renamed this identity pool to see what would happen, and AWS changed the name right back, but it did cause a problem with a role that became disconnected, so it's best to leave the name alone.

I found the Mobile Hub helpful at this point because it helped me to understand better some of the settings in the identity pool. If you look at the Mobile Hub's setting on an earlier picture, you can see that I had chosen "Sign-in is optional." Let's open the identity pool to see what it looks like. Begin by clicking on the identity pool's name:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/26.png "Click on name")

Now click the link waaaaaaay over on the far right to edit the pool (another confusing UI design! it took me a while to figure out how to edit the pool):

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/27.png "Edit pool")

You'll see lots of info about the identity pool. The roles are very important to understand, but we'll talk about those in a moment. For now, let's expand the "Unauthenticated identities" section:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/28.png "Identity pool settings")

Note how there's a checkmark next to "Enable access to unauthenticated identities". If you switch over to the Mobile Hub and change the sign-in from optional to required and save the change, then return to the identity pool and refresh the page, notice that it removed both the checkmark and the unauthenticated role:

| ![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/29.png "Change setting") | ![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/30.png "Note changes in pool") |
| :---: | :---: |
| __When you remove the optional sign-in from the Mobile Hub by changing it to "Sign-in is required"...__ | __...it removes the unauthenticated sign-in from the identity pool.__ |

-explain the unauth and auth roles, and show those on the IAM screen

Here's another confusing thing. (Show how the "create new" buttons are different on each screen).

(work in progress, not complete yet)
