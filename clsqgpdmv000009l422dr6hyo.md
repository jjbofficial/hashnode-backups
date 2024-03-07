---
title: "Connecting a Laravel application to AWS IoT Core over MQTT - Part 1"
datePublished: Sat Feb 17 2024 19:18:49 GMT+0000 (Coordinated Universal Time)
cuid: clsqgpdmv000009l422dr6hyo
slug: connecting-a-laravel-application-to-aws-iot-core-over-mqtt-part-1
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/M5tzZtFCOfs/upload/c28ee55afdcec81baed6d3fa0a2b7157.jpeg
tags: laravel, aws, iot, aws-iot-core

---

AWS IoT core provides a suite of features that makes connecting IoT devices to the cloud and managing them a breeze. An MQTT broker is one of the services offered by AWS IoT Core. In this 2-part series, I will go over creating an IoT device or `thing` in AWS parlance on IoT core and how a Laravel application can connect to IoT core to publish and receive messages.

Unlike other MQTT brokers such as Mosquitto, IoT Core requires that you define the topics that devices would be allowed to subscribe, publish messages to and receive data from. This is called a `policy`. A policy is associated with a thing and gives it the permissions defined in the policy document. We will use two topics in our policy. A topic to send commands to the device and another to receive data from the device.

One thing to note is that the AWS IoT core has a different notation when specifying wildcards in your MQTT topic. You can read more about this [here](https://docs.aws.amazon.com/iot/latest/developerguide/pub-sub-policy.html#pub-sub-policy-cert). To summarise, you should use **\*** and **?** for wildcards instead of **+** and **#.**

Before connecting to IoT Core over MQTT, we must create a policy and a `thing`.

## Create a policy

1. Log in to your AWS console and search for IoT Core in the search bar at the top right.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708178778717/c53fc6bd-0712-4aae-adc6-10fdf6151fd0.png align="center")

1. Navigate to **Manage &gt; Security &gt; Policies** and click the **Create Policy** button.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708182329258/08ab8579-b2ef-40e0-81cc-e5738eb8676f.png align="center")

1. Give your policy a name. I will be using **DevicePolicy**. In the policy statement section below, select the JSON view and paste the JSON below into the text box.
    
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "iot:Connect",
          "Resource": "arn:aws:iot:us-east-1:*:client/${iot:Connection.Thing.ThingName}"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Subscribe",
          "Resource": "arn:aws:iot:us-east-1:*:topicfilter/devices/*/status"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Publish",
          "Resource": "arn:aws:iot:us-east-1:*:topic/devices/*/query"
        },
        {
          "Effect": "Allow",
          "Action": "iot:Receive",
          "Resource": "arn:aws:iot:us-east-1:*:topic/devices/*/status"
        }
      ]
    }
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708182803569/7220f56d-1929-42f1-8d51-6e5e9007283c.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708185625224/4338a6c6-9942-4f3c-ba8b-a5c8b6660e05.png align="center")
    
    The `iot:Connect` action allows devices to connect to the MQTT broker. Two devices with the same client ID cannot connect to the broker simultaneously. The variable `iot:Connection.Thing.ThingName` is a placeholder for the client ID. You must use the thing name when connecting to the MQTT broker. You can find a breakdown of all the IoT core policy actions [here](https://docs.aws.amazon.com/iot/latest/developerguide/iot-policy-actions.html) and example policies [here](https://docs.aws.amazon.com/iot/latest/developerguide/example-iot-policies.html).
    
2. Click on the **Create** button on your bottom left to save the policy.
    

> If you ever encounter the error **The action failed because the input is not valid. Policy cannot be created - size exceeds hard limit (2048),** which means you have too many topics. You have to consolidate some of your topics into one or create an extra policy to handle some of your topics.

## Create a thing

1. Navigate to **Manage &gt; All Devices &gt; Things** and click the **Create Things** button.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708183080860/9e6b4e9b-5a27-4e7b-98d6-385e2befcdc6.png align="center")
    
2. Select **Create Single Thing** and click on the **Next** button.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708183161548/ebb477cf-1358-466b-b2a3-db859995967d.png align="center")
    
3. Give the thing a name. I will be using **device\_monitor**. Then, scroll down and click on the **Next** button.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708183633597/a1ff47c4-c300-464e-98de-0cb15e2c7017.png align="center")
    
4. Select **Auto-generate a new certificate** and click on the **Next** button.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708183657645/2c7c1e50-db41-4d45-8e66-72adb9915806.png align="center")
    
5. Select **DevicePolicy** from the list of policies and click on the **Create Thing** button.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708183770731/2c5cbf9a-6d61-4abe-846e-e80b88833327.png align="center")
    
6. Download the device certificate, public key, private key, and the Amazon Root Certificate CA 1
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1708184035782/ce1d20f9-c0cf-4cab-a54d-1eea34fe4b69.png align="center")
    
    You must download the device certificates at this step. You will not be able to download them after you close the pop-up.
    

In Part 2, I go over how to use the device certificates to connect a laravel application to AWS IoT core over MQTT.