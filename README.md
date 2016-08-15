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

I have mixed feelings about the Mobile Hub. Like any wizard, it hides the reality of what it's really doing to make things easier. At first this is great, because it gives you a starting point. However, I found that pretty quickly it was just confusing me more, because I needed to know what was really happening. The most useful thing about the Mobile Hub is that it bridges the gap between what you want to accomplish and knowing where in the AWS console to do it.

To explain better, I'll show you what happens when you click on the Mobile Hub. You'

