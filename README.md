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

This illustrates how the Mobile Hub provides a more user-friendly way to understand the proper configuration of an identity pool. Everything we do in the Mobile Hub is really just translated to a setting in one of the AWS services. The Facebook App ID is here too:

| ![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/31.png "Mobile Hub FB ID") | ![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/32.png "Cognito FB ID") |
| :---: | :---: |
| __The Facebook App ID set in the Mobile Hub...__ | __...is put here in the identity pool automatically.__ |

I had mentioned earlier that the Cognito interface is confusing, especially with regard to user pools vs identity pools. I just wanted to give a photo example of what I mean. 

Let's say you're going to add a new user pool, so you open the user pool screen and click this button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/53.png "new user pool")

Note how the button is to the far right, and is stylized in the flat design ethos.

By contrast, if you're going to add a new identity pool, you open the identity pool screen (or "federated identity" screen, as it still can't decide which it is), and click this button:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/54.png "new identity pool")

Note how this one's button is on the left, and uses a gradient, not flat design. Even stranger, though, the entire page is styled differently! Everything from the 3D orange cube to the font size to the format of the subheading is so different that it almost makes you think you're on a different website. It's pretty apparent that some other team designed each one, or perhaps one is a holdover from an old design that they've never gotten around to redesigning. It's an odd and amateurish inconsistency from a company with such massive resources.

## Roles

Understanding the "role of roles", so to speak, in AWS is crucial to everything you do, because they control how much access users have. If your roles aren't set up correctly, you can accidentally give users too much access and they can edit someone else's records, or you can prevent users from logging in at all even though you want them to.

They're actually pretty neat in how they work, but for me they're so radically different from anything I've encountered before that it took a while just to understand the basics (and I know that I'm barely scratching the surface even with what little I do know).

The biggest difference is that you create them using a markup language, rather than using a GUI or typing a command. The AWS console uses a point-and-click web interface for most of your interactions with it, and the roles seem to be this way too, until all of a sudden you're told to copy-and-paste a big JSON-looking block of text and "add a policy variable like so":

` "Condition": {"StringLike": {"s3:prefix": ["David/*"]}} `

With SQLServer, you set permissions with a point-and-click GUI:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/33.png "sqlserver")

With MySql (or SQLServer too), you might set permissions by typing commands:

`GRANT ALL ON mydb.mytbl TO 'someuser'@'somehost';`

With the AWS console, whether you're setting login permissions, or setting read/write/delete permissions on a DynamoDB table, you're going to do it by editing these blocks of markup language.

Getting back to our Mobile Hub example, we can see that in addition to creating the identity pool, it created roles for the authorized user and the unauthorized user:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/34.png "roles")

We can see these roles by going back to the "Identity & Access Management" (aka IAM) service:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/6.png "IAM")

Remember when we first created the Mobile Hub project that it automatically created a role named "MobileHub_Service_Role"? Well, now it has created 2 additional roles:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/35.png "roles")

Similar to how it autonamed the identity pool, AWS provided names that make the roles' purpose clear:
1) The "...auth..." role will be the one applied when a user has authenticated, 
and
2) The "...unauth..." role will be to users who haven't authenticated.

It's not the name that matters; the roles don't need the words "auth" and "unauth" in them (AWS just auto-applied those names to make them clearer to us, but you could choose any name you want if you're creating your own). The important thing is the mapping of the role names in the identity pool, as shown a couple of pictures earlier.

To see what the language in the role consists of, click on the role name to open it (another confusing UI design--there's no visible button/link/etc to click on). Here I'm viewing the "auth" role. The role actually can hold multiple policies. Each policy holds one complete unit of the markup language. Click on the "Edit policy" link as shown below:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/36.png "Edit policy")

This shows us the actual text of the policy:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/37.png "Policy text")

There's not much to this policy. It's pretty clear that it's allowing the user to call Cognito's GetId function in order to get a Cognito ID. You can read about the function here:

http://docs.aws.amazon.com/cognitoidentity/latest/APIReference/API_GetId.html

Interestingly, if you look at the "unauth" role, you'll see that it has the exact same policy as the "auth" role, but this makes sense because we set the Mobile Hub app to make log-ins optional, so unauthenticated users are allowed to use our AWS resources also.

The roles and their policies get more complex when we get DynamoDB involved, which is good because they give us a chance to see how to restrict a user's permissions and only allow them to do certain things.

## DynamoDB

The main feature of my app will be the ability to write data to a DynamoDB table and be able to read other users' data. Users should be able to read anyone's data, but only edit their own records.

Back in the Mobile Hub, there was another feature called NoSQL Database. Let's click on that one:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/38.png "DynamoDB")

Click to "Enable NoSQL", then "Add a new table":

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/39.png "DynamoDB")

I'm choosing "Protected" because it has what I need for this table (ie, "Any app user can read, only owner can write to item").

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/40.png "Setting up table")

By default it gives it a userId attribute and sets it as the partition key. What is a partition key? I'll do my best to explain.

Coming from a relational database background, it's hard to get my head around the DynamoDB table concept. Part of the problem for me, I think, is that they say they're not the same as relational database tables and there's no such thing as a column, but then their UI presents the data in rows and columns (I'll show that in a picture later), plus their use of the name "table" itself implies a similarity. 

Regarding a partition key, it has to do with how the table divides its records in order to store them. To quote from Amazon's documentation at http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GuidelinesForTables.html:

> The primary key uniquely identifies each item in a table. The primary key can be simple (partition key) or composite (partition key and sort key).
When it stores data, DynamoDB divides a table's items into multiple partitions, and distributes the data primarily based upon the partition key value. The provisioned throughput associated with a table is also divided evenly among the partitions, with no sharing of provisioned throughput across partitions.
Total Provisioned Throughput / Partitions = Throughput Per Partition
Consequently, to achieve the full amount of request throughput you have provisioned for a table, keep your workload spread evenly across the partition key values. Distributing requests across partition key values distributes the requests across partitions.
For example, if a table has a very small number of heavily accessed partition key values, possibly even a single very heavily used partition key value, request traffic is concentrated on a small number of partitions – potentially only one partition. If the workload is heavily unbalanced, meaning that it is disproportionately focused on one or a few partitions, the requests will not achieve the overall provisioned throughput level. To get the most out of DynamoDB throughput, create tables where the partition key has a large number of distinct values, and values are requested fairly uniformly, as randomly as possible. 

They also provide this table to give an idea of the key to choose:

| Partition key value	| Uniformity |
| ------------------- | --------- |
| User ID, where the application has many users. | Good |
| Status code, where there are only a few possible status codes.	| Bad |
| Item creation date, rounded to the nearest time period (e.g. day, hour, minute)	| Bad |
| Device ID, where each device accesses data at relatively similar intervals | Good |
| Device ID, where even if there are a lot of devices being tracked, one is by far more popular than all the others. | Bad |

It makes sense after reading this why they default the table to having userId as the partion key, because it seems like by far the best and easiest key to use. Beyond this, though, my relational-db habits start kicking in as I start trying to design these tables the way I usually do. Here are some pointers I gradually figured out through reading and trial-and-error:

* The primary key can have at most 2 fields. If we're using the userId as the partition key, we're almost certainly going to need another field in the primary key, and the only field left is the sort key. You can't have a 3+ field composite key like a relational db table.
* There's no auto-incrementing field, so if you're going to make that sort key a unique identifier, a common practice seems to be to use a UUID (Universally Unique Identifier) in that sort key.
* A consequence of the 2-field primary key limit is that you forget about trying to design the table around a clustered index, but instead add indexes to make the searches faster depending on how you're going to be querying.
* Once you have created a table, you cannot change the primary key field(s) in any way. They are set in stone.
* The additional "fields" that you add in the Mobile Hub (like firstName in my example), aren't even really added to the table's design in any way, unless they're used in an index. This is an example where the Mobile Hub really misled my understanding of how DynamoDB tables really work. It's really more like a Javascript object, where you can add name/value pairs (probably a bad analogy but I'm trying to describe how each record can have zero or more values, whereas a relational database's record will have all the fields that are defined as columns on that table).

For simplicity's sake in the MyFriends example table I have, I'm going to use the last name as the sort key, but in real life this would never work, because each user can only have one record with that last name (because the primary key consists of userId and lastName), eliminating the ability to add more than one person with the same last name.

Notice how it gives you examples showing what kinds of queries will work on this table. If we wanted the ability to quickly sort and search on the first name, we would need to add an index on that field.

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/41.png "Setting up table")

Let's switch over to the DynamoDB service by clicking on its icon in the AWS console:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/42.png "DynamoDB")

We can see that the Mobile Hub has automatically created a table for us, once again giving it an auto-generated name that contains the name we specified in Mobile Hub:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/43.png "DynamoDB Table list")

Click on the table to the various information about the table. Note that lastName is nowhere to be seen. I think that MobileHub lets you add it primarily so it can show you in its sample code how you would write to the field. To me, though, coming from a relational db background, it just confused me that the actual DynamoDB table didn't have the fields that I had set up in Mobile Hub. 

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/44.png "DynamoDB Table list")

You can add records (or "items", as they're more precisely called in DynamoDB's terminology) by clicking the Create Item button and typing in values:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/45.png "Add item")

The new item appears in the table onscreen:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/46.png "New item")

Here's another of my UI-gripes. You can add additional attributes to an item, and when you view them altogether, it looks just like a traditional relational table with rows and columns. In this picture, I've added an attribute "hobby" to one of the items, and an attribute "lastName" to another item:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/47.png "Items")

If this were a relational table, those blanks in each record would be nulls in those fields, but each record would have all the fields contained in it. That is to say, the "hobby" field would exist for every record, even if the field contained a null for some of the records, as it does for the first and third in my picture. However, with DynamoDB, the "hobby" attribute literaly doesn't exist on the first and 3rd record. It only exists on the second record. You can see this when you click on each item. One item has 2 attributes, one has 3, and another has 3 but the 3rd attribute is different:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/48.png "items with attributes")

One thing that can mislead you is when you want to set up permissions on this table. When you're looking at the table's configuration settings, you'll notice a tab named "Access Control." It would seem like you set up permissions for the table here, since they have the tab available. (This is the way SQLServer works for table permissions--you can set up the table's permissions right there in the table's configuration.) Here in AWS, the "Access Control" tab is a strange and pointless UI control. When you choose the options and click the "Create Policy" button, it writes the text of a policy that you must copy/paste into the IAM console. In fact, when you expand the "Attach Policy Instructions" section, it just gives you instructions for copying & pasting into the IAM console! I can see their rationale, that they're trying to make it easier for a novice user who is just learning how to use AWS, but in my opinion it's just confusing. This is the key point to remember:

* You set up permissions for DynamoDB tables in the IAM console, not in DynamoDB.

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/49.png "table permissions")

I want my table to allow anyone to read its records, but only the creator of the records can edit them. Amazon has an article here that gives a good intro to the use of substitution variables in AIM policies:

http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/specifying-conditions.html

For now, though, let's look at how the Mobile Hub has set up the IAM policies for us. Remember that I chose "Protected", which means "Any app user can read, only owner can write to item." When we look at the IAM console and click on the "auth" role (ie, the 2nd one in this list),

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/35.png "roles")

we can see that it has created a new policy that wasn't there before. The "signin" policy was created to control the sign-in action, and now there's a new policy to control the database access. Let's click on "Edit Policy" to see what it contains:

![alt text](https://github.com/jvtoomey/aws-cognito-dynamodb-swift-fb/raw/master/DocumentationImages/50.png "table policy")

It's instructive to look at the specifics of the policy to get a sense of how you set permissions in AWS. Here's the policy in its entirety:

~~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:BatchGetItem",
                "dynamodb:BatchWriteItem",
                "dynamodb:DeleteItem",
                "dynamodb:DescribeTable",
                "dynamodb:GetItem",
                "dynamodb:ListTables",
                "dynamodb:PutItem",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:UpdateItem"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-east-1:000000000000:table/testthisgizmo-mobilehub-3333333353-MyFriends"
            ],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "dynamodb:LeadingKeys": [
                        "${cognito-identity.amazonaws.com:sub}"
                    ]
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:BatchGetItem",
                "dynamodb:DescribeTable",
                "dynamodb:GetItem",
                "dynamodb:ListTables",
                "dynamodb:Query",
                "dynamodb:Scan"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-east-1:000000000000:table/testthisgizmo-mobilehub-3333333353-MyFriends"
            ]
        }
    ]
}
~~~~

Note that the first set of actions allow for "PutItem" and "UpdateItem" which correspond to the "Add" and "Update" events in relational databases. This is allowed for the `cognito-identity.amazonaws.com:sub` value. As discussed in this article,

https://mobile.awsblog.com/post/Tx1OSMBRHZVM9V0/Understanding-Amazon-Cognito-Authentication-Part-3-Roles-and-Policies

> This value is the identity ID for the individual user, not the identifier of their login account (such as an email address)

I believe "LeadingKeys" refers to the hash key of the table, so basically it's saying that if your Cognito identity ID matches the value that's stored in the hash key attribute of the table item, then you have the right to update that record.

The 2nd half of the policy tells us what happens for users whose identity ID doesn't match: They can get items and query the table, but they can't do updates.

If we go back to the Mobile Hub and change the table's permissions from Protected to Public, which means "any app user can read and write to any item", then look at the IAM policy, we can see how it's changed:

~~~~

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:BatchWriteItem",
                "dynamodb:DeleteItem",
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:BatchGetItem",
                "dynamodb:DescribeTable",
                "dynamodb:GetItem",
                "dynamodb:ListTables",
                "dynamodb:Query",
                "dynamodb:Scan"
            ],
            "Resource": [
                "arn:aws:dynamodb:us-east-1:000000000000:table/testthisgizmo-mobilehub-3333333353-MyFriends"
            ]
        }
    ]
}
~~~~

Note that there's no Condition anymore that restricts who can write to the table; now it's allowed for everyone.
