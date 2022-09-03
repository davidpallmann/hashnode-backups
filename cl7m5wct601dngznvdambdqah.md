## AWS & IoT -  Philips Hue Lights, Part 1: Home Setup

I decided recently to jump into Internet of Things, and blog over time about various ways to connect devices, AWS, and .NET. It's been quite a while since I did any device work, so I know I'll have to walk before I can run. In this first post, I'll take you along my quest to set up my first devices, Philips Hue Smart Lights and Bridge. This is sure to be an adventure with some fun, some frustration, and some missteps, as I'm learning as I go.

Here in Part 1, I'll cover unpacking and home setup. In Part 2, we'll set up and work with the API. After that, .NET on AWS code examples will follow. 

# Internet of Things: What is it, and why should I care?

The term ["Internet of Things"](https://en.wikipedia.org/wiki/Internet_of_things) was made popular by British technologist [Kevin Ashtin](https://en.wikipedia.org/wiki/Kevin_Ashton), who was focusing on Radio Frequency Identification (RFID) at the time. He pointed out that in the 20th century, computers were brains without senses: they only knew what we told them. Here's how Ashton originally described it:

> "Today computers—and, therefore, the Internet—are almost wholly dependent on human beings for information. Nearly all of the roughly 50 petabytes (a petabyte is 1,024 terabytes) of data available on the Internet were first captured and created by human beings—by typing, pressing a record button, taking a digital picture or scanning a bar code. Conventional diagrams of the Internet ... leave out the most numerous and important routers of all - people. The problem is, people have limited time, attention and accuracy—all of which means they are not very good at capturing data about things in the real world. And that's a big deal. We're physical, and so is our environment ... You can't eat bits, burn them to stay warm or put them in your gas tank. Ideas and information are important, but things matter much more. Yet today's information technology is so dependent on data originated by people that our computers know more about ideas than things. If we had computers that knew everything there was to know about things—using data they gathered without any help from us—we would be able to track and count everything, and greatly reduce waste, loss and cost. We would know when things needed replacing, repairing or recalling, and whether they were fresh or past their best. The Internet of Things has the potential to change the world, just as the Internet did. Maybe even more so."

The term Internet of Things has caught on, and today is used to refer to physical devices that can connect to networks and interact with them, from sensors to devices that carry out actions in the physical world. Pretty much nobody likes this term, pointing out that "Internet of Things" is hardly appropriate when the majority of devices are in fact connected to local networks. Yet, no one seems to have found a better name, and so IoT it is.

The types of IoT devices available today are numerous indeed, and range from the smart home to the smart city. Categories of IoT include:
* home automation (light bulbs, doorbells, cameras, security sensors)
* automotive (GPS, maintenance, safety, connected cars, fleet management, autonomous driving)
* smart city (traffic and parking sensors, environmental monitoring)
* medical devices (wearable sensors, remote health monitoring, emergency notification systems)
* industrial (RFID tags, sensors, digital control systems)
* agricultural (weather and soil sensors, water pumps)
* building energy management (sensors, control of lighting and heating systems)
* military (reconnaissance, surveillance, combat)

Depending on the device and application, you can use a variety of AWS services with devices, including AWS IoT Core, AWS Lambda, Amazon Alexa, and Amazon Kinesis.

# Where to Start?

If you're going to jump into IoT, you're going to need devices - but where to start? I live dangerously close to a Best Buy, and walking in I quickly came to the large Philips Hue display. Philips offers a multitude of products for controllable lighting under the brand name [Hue](https://www.philips-hue.com/en-us?origin=p71805997253&gclid=CjwKCAjw9suYBhBIEiwA7iMhNEpKkdoPlAXfFxNtWSrBFJg3ium_m00iTog7jz2Dzm7pEOXJfeHJKxoC1OIQAvD_BwE&gclsrc=aw.ds). I imagine I'll eventually have a variety of connected devices including sensors, switches, control devices, and perhaps something robotic. Lights would be useful indicators, and Hue has a strong presence, so I decide to start there.

After looking over the devices on display, some conversations with Best Buy staff, and impromptu online research, I decide I want some Hue smart bulbs, the kind that will fit most home lamps. I learn that these bulbs work via Bluetooth, but you can add a Hue Bridge to connect them to your Internet router. Since doing something with AWS and .NET is my ultimate aim, I select the [Philips Hue Starter Kit E26](https://www.philips-hue.com/en-us/p/hue-white-and-color-ambiance-starter-kit-e26/046677563295), which includes a Hue Bridge and 4 Smart Bulbs. It's $199, which is about $199 more than my wife wanted me to spend that weekend on unnecessary purchases. Clearly I'll have to build up my device inventory slowly over time, and interleave that with some spouse-focused purchases.

![get-started-kit-box.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661774900961/FyHp9ar8W.jpg align="left")

I asked the Best Buy person where I could get something to put the bulbs in, so I could have them in an array to look at, and to my surprise they had nothing in the store. Fortunately, there was a Target next store, and I purchase 2 of the cheapest lamps I could find. Make that 4, I went back out and got 2 more. My adventure begins!

![kit-and-lamps.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661774966123/A0sCFNupN.jpg align="left")

# The Philips Hue Starter Kit E26

## Unboxing

Inside the box of the Hue Starter Kit you first see the bridge, and below it a smart bulb. 

![box-1-bridge-bulb.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661775432646/n85my6zpF.jpg align="left")

Removing the bridge, you see 3 smart bulbs. 

![box-2-bulbs.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661775460624/7kO8hXoxk.jpg align="left")

Below that is a power adapter for the bridge, an Ethernet cable, and the fourth bulb. I hadn't expected an Ethernet cable, thinking WiFi, but that will be fine.

![box-3-adaptor-cable.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661775469924/o-_rA5JT9.jpg align="left")

There are instructions, but they are super minimal, graphical for an international audience.

![box-4-instructions.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661775373345/9eH_wZny3.jpg align="left")

## Connecting to Router

Connecting the bridge to my router is simple enough. I have a connector for the Ethernet cable and a power outlet.

![bridge-router.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661775823427/MlS019Np0.jpg align="left")

The bridge itself has an attractive, clean design. 3 blue lights tell you that you have power, are connected, and are connected to the Internet. There's a large, circular button called the Link button. You'll use this to connect to mobile apps or your PC, and as a security mechanism when configuring external access.

![bridge.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661775975876/gBQv1p1tG.jpg align="left")

## Configuring the Hue Phone App

Anxious to see something happen with the lights, I go to the Google Play Store on my Android phone and look for something under Hue. Make sure you get the right app, there are several. If you're going to be interacting with your Hue devices beyond your own home network, the app you want to install is called the **Hue App for Hue Light**. That will set up an identity for you and let you define a home for your Hue devices, which is what you need to do anything with IoT.

![phone_store_hue_app.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662214272071/qNEBp-rnC.jpg?height=500 align="left")

The app walks you through setup. It also shakes you down for paying for Premium features, which I resent, but decide to pay the one-time $19.99 for lifetime access. I add my Hue Bridge, and am rewarded with an IP address.

![phone_hue_configure.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662215022565/Ukv__sxge.jpg align="left")

There's also an option to enable Alexa access, which I'll set up eventually, but right now I'm focused on programmatic access. Next up, the App offers to set up third-party app connections, which is what we're after. 

### Connecting the Bridge

This next part was a bit frustrating, and took an hour, as the app went through a repetitive dance of having me press the Hue Bridge Link button, detecting the bridge, not finding it, asking me to repeat, finding the bridge and deciding it needs software updates. This was the only part of setup that wasn't smooth and simple. It's not obvious how long the Link button sends out its signal, and there aren't any visual indicators.

![phone_hue_configure_2.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662218943699/REBLs89uj.jpg align="left")

### Setting up a Home

It's now time to define a home, with rooms, and the lights assigned to rooms. I define a room named Office.

![phone_hue_configure_3.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662219192548/WudAxenT4.jpg align="left")

The app detects 4 lights. This surprised me, as I only had two of them plugged into lamps at the time. I assign all 4 lights to room Office.

![Screenshot_4LightsAreFound.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662219785001/Hz1UGcfhQ.jpg?height=500 align="left")

Now, going to the Hue app shows me my home, which currently has the one room defined, Office.

![Screenshot_Hue_Rooms.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662219413251/dJlTBoogj.jpg?height=500 align="left")

Clicking on Office, I see my 4 lights. Well, 3 of them. You have to scroll right to see more.

![Screenshot_Hue_Lights.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662219433638/SemXfMSKH.jpg?height=500 align="left")

### Fun with Lights

At this point, a lot of Hue light purchasers are happy to use the mobile app or their home assistant to set up and play with their lighting, it's all they need. It's tempting for me as well, because it's fun, so I spend some time learning how to adjust light color and brightness. 

You can switch your lights between white light and color modes. In either mode, you use a color wheel to select the hue you want.

![phone_hue_configure_3_light_colors.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662220339684/46l-ml9DE.jpg align="left")

You can control both color and brightness, giving you millions of colors.

![phone_hue_configure_4_LightsExample.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1662220705941/nZqryRfzM.jpg align="left")

All of that was fun and satisfying, but I want to do it programatically. Well, that's where we go in Part 2. Philips has done a good job of catering to two distinct audiences: consumers who want to connect smart lighting to their homes with minimal fuss, and developers. Develpment will be our focus going forward.

# What's Next?

In Part 2, we'll learn about the API you can use to control your Philips Hue lights. Once we have API access, we can starting leveraging .NET code and AWS services in a variety of ways, such as AWS Lambda functions and Alexa voice skills.

# Further Reading

[Kevin Ashton describes the "Internet of Things"](https://www.smithsonianmag.com/innovation/kevin-ashton-describes-the-internet-of-things-180953749/)

[Philips Hue Starter Kit E26](https://www.philips-hue.com/en-us/p/hue-white-and-color-ambiance-starter-kit-e26/046677563295)
