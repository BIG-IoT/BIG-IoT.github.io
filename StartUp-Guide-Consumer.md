---
layout: single
title: Start-Up Guide for BIG IoT Consumer Developers
sidebar: 
  nav: "docs"
---

## Step-by-step Instructions

### What you will need
- Basic Java experience
- Git (e.g. Git-SCM, see here at [https://git-scm.com](https://git-scm.com/))
- A Google or GitHub Account - to authenticate on the Marketplace 
- An IDE for Java Development (e.g. Eclipse) or simply a text editor :smiley:

### 1. Step: Login on Marketplace

- Select the BIG IoT Marketplace you want to use:
  - Public BIG IoT Marketplace URI: [https://market.big-iot.org/](https://market.big-iot.org/)
    - Here you find our BIG IoT data Offerings. You can also provide your Offerings here.
- Click on `Login`
- You can use your GitHub or Google account to sign in

### 2. Step: Create new Organization on Marketplace

- The first time you login on the Marketplace with your account, you can create a new Organization 
- Click on `New Organization` (see bottom left)
- Enter a name for your new Organization

### 3. Step: Create new Consumer instance

- Click on `MyConsumers` 
- Click on `+Consumer`
- Enter a name for your new Consumer instance (e.g. "SmartCityDashboard")

### 4. Step: Find Offerings

Create a new Offering Query
- Click on `MyConsumers` and select your new Consumer instance
- Click on `+OfferingQuery` and give your OfferingQuery a name
- Define a semantic _Category_ (and if applicable also sub-categories)
- Optionally: define a _Region_, a _Time Period_, a _License_, a _Price_, etc. 
- Save the OfferingQuery
- Scroll down - to the bottom of your OfferingQuery. There you will see all the matching Offerings
  - NOTE: You might need to `refresh` your browser or `reload` the page to get the update
- Explore the Offerings

### 5. Download Java Template Project

`git clone https://github.com/BCX18ConnectedLife/big-iot.git`

### 6. Import the Project into your IDE 

- For Eclipse IDE:
  - Go to `File` -> `Import` -> `Gradle Project`
    - NOTE: If you don't see the Gradle Project import option, you first have to install the Eclipse gradle tooling (`buildship`) - see [here](http://www.vogella.com/tutorials/EclipseGradle/article.html) for further information
  - Select the consumer template project directory: `https://github.com/BCX18ConnectedLife/big-iot/java-template-consumer`
  - Import the project
  
### 7. Update Properties Files 

Update your Consumer ID and Secret in the `example.properties` file (see root directory of the template project)

NOTE: You can copy the Consumer ID and SECRET directly from the Marketplace Web Portal (e.g. https://market.big-iot.org/): Open it in the Browser, Log In and then click on `Copy ID to Clipboard` (see top right) for your Consumer ID or `Load Consumer Secret` followed by `Copy Secret to Clipbard` for your Consumer Secret.

### 8. Edit the Example Consumer Java application 

- Open the Example Consumer Java source file: `./src/main/java/org/bigiot/examples/ExampleConsumer.java`
- Update the `Offering ID` here:
```java
consumer.subscribeByOfferingId("... include the Offering ID that you want to subscribe to and access ...").get();
```

NOTE: You can copy an Offering ID when you open the Offering on the Web Portal, and then click on `Copy ID to Clipboard` (see top right).

### 9. Run the Consumer application 

- From the command line: Use `./gradlew run` in the root directoy of the project
- From Eclipse IDE: Go to `Run` -> `Run` and then select your consumer application

