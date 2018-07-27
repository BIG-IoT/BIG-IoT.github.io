---
layout: single
title: Tutorial
sidebar: 
  nav: "docs"
---

Using the BIG IoT libraries, it is easy to get started to dive into the BIG IoT ecosystem. These libs allow providers to register and manage their offerings and consumers to access offerings from the BIG IoT Marketplace. 

![BIG IoT Architecture](../img/architecture.png)

In this developer guide, three parts are described:

1. in [this example](../providerPerspective), we show how to bring an existing IoT platforms into the BIG IoT ecosystem. The example platform is a simple IoT platform which offers parking data in Barcelona and makes it public on the BIG IoT Marketplace. 
1. in [this example](../consumerPerspective), we show how to use the BIG IoT Consumer Library to access this data from the perspective of a developer who wants to retrieve that data in an application or service.
1. [Here](../moreLibFunctionality) you find additional information about advanced usage of the BIG IoT Lib.

# Before you begin...

## Installation of the tooling
In order to try the examples or to get started for development, you have to setup your de-velopment environment. If you want to follow this example, it helps you a lot if you have installed the tools from the table below. Of course you can you use your own tooling, but if you are not sure, just get exactly the version from the "We used" column.

| Install | We used                                                                                                                                                                                             | Find it at                                                |
|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| Git     | The Git distribution of,git-scm.com (v2.11.0 )                                                                                                                                                      | https://git-scm.com/                                                    |
| Java    | Java SE Development Kit 8,(JDK 8)                                                                                                                                                                   | http://www.oracle.com/technetwork/java/javase/downloads/index.html      |
| Eclipse | Eclipse Oxygen (v4.7.0) or newer | http://www.eclipse.org                                                  |
| Gradle  | part of Eclipse since Oxygen |

### Setup Eclipse
Our tutorials shows a walkthrough using Eclipse. You’ll find the example code at [https://github.com/BIG-IoT/example-projects]. If you want to try the tutorial by your-self, execute the following steps:
```
cd <your Eclipse workspace folder> 
git clone https://github.com/BIG-IoT/example-projects  
```
The BIG IoT examples come as Gradle projects. In Eclipse versions Oxygen and later, Gradle is already included out-of-the-box. 
We recommend that you start from a fresh workspace. This is the case if you just installed Eclipse else just switch to a new workspace. 
Import `java-example-consumer` and `java-example-provider` as a Gradle project.
If you are not sure, how this works. Select from the menu "File" -> "Import".
 ![Import](../img/importGradle.png)
Select ![Gradle\Gradle Project](../img/importGradleProject.png).
Hit "Next"
In the “Project root directory” text field, set the folder where the example sources are located (here we imported the consumer example)). It takes a while until Gradle retrieves all the dependencies. ![Eclipse](../img/eclipseExampleProject.png).

### Get your Marketplace Credentials

Before you start the demo, you have to create a provider and a consumer at the BIG IoT Marketplace. The steps for creating a provider and a consumer are quiet similar. We explain here only the steps for the provider. 
Open a web browser and go to [https://market.big-iot.org]. 

!!!login button

Find the yellow login button in the lower left corner. Click it and authenticate with your Google or GitHub account. Then, click on the text "New Organization" and create your organization. Now, you should see a screen similar to ![this](../img/marketplace.png).
As you can see, my organization has the name "Rain Cloud". Within this organization, we can create a provider for the demo. Therefore, click on "My Providers" and then on "+ Provider".

Find a good name for your provider and hit "Save".

In the upper right corner, you find two buttons to load and copy the marketplace credentials to your clipboard.  The provider ID and secret have to be inserted in the example.properties file, located in the project folder.
Now, you start the example provider: Select the file ExampleProvider.java in the “Project Explorer”, in the menu bar hit “Run”  “Run as”  “Java Application”. The provider is now running, it offers a parking availability service, which is registered in the background and deployed as a web service on your machine.
Before you can see something interesting in the console, you have to start the consumer too. Therefore, you have to create also a consumer at the marketplace and transfer the credentials to the code. The steps are similar to creating a provider. Just switch back to your web browser and start the same process, now, by clicking on “My Consumers”. The creden-tials have to be inserted in ExampleConsumer.java, which is below the directory then java-example-consumer. 
Now, start the Consumer and see what happens. Enjoy!