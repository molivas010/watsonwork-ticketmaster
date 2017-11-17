# watsonwork-ticketmaster

[![Build Status](https://travis-ci.org/watsonwork/watsonwork-ticketmaster.svg)](https://travis-ci.org/watsonwork/watsonwork-ticketmaster)

A sample Watson enabled cognitive app that integrates with the Ticketmaster api's 
to illustrate action fulfillment flows. You can find out more about this at 
http://developer.ticketmaster.com and details follow on how to register.

The Watson Work platform provides **spaces** for people to exchange
**messages** in conversations. This sample app shows the following
aspects of a Watson Work cognitive application:

* how to implement an action fulfillment flow.
* how to act upon actionSelected annotations received from users clicking on underlined links from triggered Watson Conversation intents.
* sending a createTargetedMessage graphql mutation targeted at a particular user and ui dialog.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. 
See deployment for notes on how to deploy the project on a live system.

## Obtain a Ticketmaster api key

1. Navigate to https://developer.ticketmaster.com/ and select 'Get Your API Key'.
2. Follow the steps to create an account and request api access.
3. Once approved you will receive an consumer key and a consumer secret which you can use when filling out the application.yml file below.

## Obtain a Google Maps API key
1. Navigate to https://developers.google.com/maps/documentation/javascript/get-api-key?refresh=1&pli=1 and select Get A Key
2. Login and save your key for use in the application.yml file (Up to 2500 calls a day is free)

## Creating Watson Work Services App

1. In your Web browser, go to [Watson Work Services / Apps](https://developer.watsonwork.ibm.com/apps)
2. Click on `Create new app`
3. Give your app an unique name with description and click on `Create`
4. Take a note of the `App ID` and `App Secret`
5. Click on `Listen to Events -> Add an outbound webhook`
6. In the callback URL, specify the URL for your app. This code assumes that the webhook listener is at https://yoururl/webhook so don't forget to add /webhook to the end of the URL (_if you don't know where the app will be deployed, use a sample URL for now, like https://test.acme.com/webhook and you can modify that later_)
7. Register for the Events 'message-annotation-added' and 'message-created'.
8. Take a note of the Webhook Secret as you will not receive it again.

**NOTE:** _**Do not commit your app Id, secret, and webhook secret when pushing your changes to Github**_

### Configuring the Bluemix Watson Conversation service

The sample ticketmaster app uses Watson Conversation to understand natural
language and provide a natural language conversational interface, so
you need to configure a Watson Conversation Bluemix service for it.

Go to the
[Bluemix Watson Dashboard](https://console.bluemix.net/developer/watson/services)
and create a Watson Conversation service.

Note the Watson Conversation service user name and password, as you will
need to configure the ticketmaster app with them.

From the Watson Conversation service page click **Launch tool** to open
the Watson Conversation tooling, 
When you get to the Workspaces screen, click on up arrow next to the Create button and import the file **watson.json** that 
is in the root of this repository, into a new Watson Conversation workspace.

Note the Watson Conversation workspace id, as you will need to configure this
app with it.

### Running locally using IntelliJ IDEA

Prerequisite for running the app using IntelliJ IDEA:
- Install IntelliJ IDEA Community Edition first if you don't have it (https://www.jetbrains.com/idea/download)
- Install [Lombok Plugin](https://plugins.jetbrains.com/plugin/6317-lombok-plugin) for IntelliJ IDEA
- Install [Ngrok](https://ngrok.com/) - used for testing the app locally without deploying on any PaaS

1. [Fork it](https://github.com/molivas010/watsonwork/watsonwork-ticketmaster/fork)
2. Clone the project `git clone https://github.com/YOUR_GITHUB_USERNAME/watsonwork-ticketmaster.git`
3. Open IntelliJ IDEA `File -> New -> Project from Exisiting Sources... `
4. Navigate to the project and select `build.gradle`. Click `OK`
5. Click `OK` on Import Project from Gradle window
6. Open `src/main/resources/application.yml`

```yaml
    watsonwork:
            webhook:
                secret: ${WEBHOOK_SECRET:yourkey} #replace yourkey with your webhook secret 
            app:
                id: ${APP_ID:yourappID} #replace yourappID with your app Id 
                secret: ${APP_SECRET:yourappsecret} #replace yourappsecret with your app secret 
            api:
                uri: ${WATSON_WORK_API_URI:https://api.watsonwork.ibm.com} 
                
    ticketmaster:
            api:
                uri: ${TICKETMASTER_API_URI:https://app.ticketmaster.com/discovery/v2/}
                consumer-key: ${TICKETMASTER_CONSUMER_KEY:your-consumer-key}
                consumer-secret: ${TICKETMASTER_CONSUMER_SECRET:your-consumer-secret}  
                
    google:
        api:
            uri: ${GOOGLE_API_URI:https://maps.googleapis.com/}
            key: ${GOOGLE_API_KEY:yourAPIkey}  #replace yourAPIkey with your google API key
 ```
7. Right click on `ApplicationBoostrap.java` `->` `Run ApplicationBootstrap..`
8. Open up terminal where ngrok is installed and run the command `ngrok.exe http 9080` on windows or `./ngrok http 9080` on unix. By default the app runs on `http port 9080`. This exposes your app via a publicly accessible URL. ngrok displays a forwarding url after executing the command above which might look as such: `http://sd2323.ngrok.io`. Take a note of that URL
9. Navigate to your app on [Watson Work Services / Apps](https://developer.watsonwork.ibm.com/apps). Select `Listen to Events`. Edit the webhook configuration with the new publicly available URL from ngrok. Make sure your callback URL has /webhook path. For example: `http://sd2323.ngrok.io/webhook`
10. Select `message-created` Event. Click `Save` and `Enable` the webhook
11. Add your app to a space on Watson Workspace and watch it echo your messages..

## Deployment

### Deploying  on IBM Bluemix

Assuming you have completed steps 1-6 in `Running locally using IntelliJ IDEA`

1. Sign up for a free trial on [Bluemix](https://console.ng.bluemix.net/)
2. Install the [Cloud Foundry CLI](https://github.com/cloudfoundry/cli/releases) and then Install the [Bluemix CLI](http://clis.ng.bluemix.net/ui/home.html). On Windows, be sure to add the bluemix cli to your PATH.
3. Run `bluemix login -a https://api.ng.bluemix.net --sso`, and get your single access passcode from the link provdided.
4. (optional) You may have to target a space once you get the single access passcode. For me that was required
 `bx target -o molivas -s dev` (for example)
5. Open a terminal in the project root. Run `./gradlew clean build` on linux/mac or `gradlew clean build` on windows.
6. Run `bx cf push my-app-name -p build/libs/watsonwork-ticketmasterXX.jar -m 512m` (Tip: Make sure the name you want to use is not taken on Bluemix already, since it must be unique.)  Also make sure the use bx in front of cf if you want to use the credential created earlier otherwise you get an authentication failure
7. When it's finished pushing to bluemix, visit your app's url.
8. Navigate to your app on [Watson Work Services / Apps](https://developer.watsonwork.ibm.com/apps). Select `Listen to Events`. Edit the webhook configuration with the new publicly available URL from Bluemix. Make sure your callback URL has /webhook path. For example: `https://my-host-name.mybluemix.net/webhook`


## What API does the app use?

The app leverages the Watson Work Services and Watson Conversation
cognitive capabilities to understand natural language intents, entities,
and determine the necessary app actions.

It uses the [Watson Work OAuth API](https://workspace.ibm.com/developer/docs)
to authenticate and get an OAuth token.

It implements a Webhook endpoint according to the
[Watson Work Webhook API](https://workspace.ibm.com/developer/docs) to
listen to conversations in a space and receive messages and message
annotations.

Finally, it uses the [Watson Work Spaces API](https://workspace.ibm.com/developer/docs)
to send back weather information messages to the space.

## Built With

* [Spring Boot 1.5.2](https://projects.spring.io/spring-boot) - Web Framework
* [Gradle](https://gradle.org/) - Dependency Management
* 
## How can I contribute?

Pull requests welcome!
