---
title: Understanding WhatsApp messaging
navigation_weight: 3
description: WhatsApp messaging solution for businesses.
---

# Understanding WhatsApp messaging

WhatsApp Business Solution messages can only be sent by businesses that have been approved by WhatsApp. This business profile will also have a green verified label to indicate that it is a legitimate business.

The advantage of WhatsApp is that the identifier of users on the platform is their mobile phone number.

> **NOTE:** WhatsApp is in Limited Availability and Vonage cannot guarantee you will receive a WhatsApp account.

## Using existing WhatsApp Business Number

If you already have a WhatsApp Business Number and would like to use that number with the Vonage Messages API, the [WhatsApp Product](https://www.nexmo.com/products/messages/whatsapp) page has more details about how to get started using WhatsApp with Vonage.

> **NOTE:** Once a WhatsApp number is integrated with the API it cannot be used in the mobile app.

## Important WhatsApp rules

If your customer initiates messaging with you, WhatsApp will not charge you for any MTMs (Template Messages) that you send back to the customer, for up to 24 hours following the last message that your customer sent you. This 24 hour period is known as the Customer Care Window. Any additional message you send to that customer beyond the Customer Care Window must be an MTM, for which you will be charged. The Vonage API Usage fee for every outbound message applies regardless of the customer care window. There are no charges for inbound messages. Further pricing information is available on the [WhatsApp Product](https://www.nexmo.com/products/messages/whatsapp) Page.

> **IMPORTANT**: The WhatsApp Business Solution may not be used to send any messages to or receive messages from the following countries or regions: Crimea, Cuba, Iran, North Korea, and Syria.

## WhatsApp message types

There are a number of different WhatsApp message types:

Message Type | Description
---|---
Text Message | A plain text message. This is the default message type.
Media Message | A media message. Types are: image, audio, document and video.
Message Template | Message Templates are created in the WhatsApp Manager. Outside of the Customer Care Window messages sent to a customer must be a Message Template type. Only templates created in your own namespace will work. Using an template with a namespace outside of your own results in an error code 1022 being returned.
Media Message Templates | Media message templates expand the content you can send to recipients beyond the standard message template type to include media, headers, and footers using a `components` object.
Contacts Message | Send a contact list as a message.
Location Message | Send a location as a message.
Interactive Message | The Vonage Messages API v1 supports two types of WhatsApp Interactive Messages: **List Messages** and **Reply Buttons**. [Read more](/messages/concepts/whatsapp-interactive-messages)

## How WhatsApp works

A business can start a conversation with a user and a user can start a conversation with a business.

WhatsApp has a core concept of Messages Templates (MTM). These were previously known as Highly Structured Messages (HSM).

> **IMPORTANT:** WhatsApp requires that a message that is sent to a user for the first time, or that is outside the Customer Care Window, be an MTM message. WhatsApp also requires that you obtain opt-in from your customers prior to sending them a message, this may be obtained on your website, IVR, or other standard means see [Facebook's docs](https://developers.facebook.com/docs/whatsapp/guides/opt-in/) for more details.

The MTM allows a business to send only the template identifier along with the appropriate parameters instead of the full message content.

> **NOTE:** New templates need to be approved by WhatsApp. Please contact your Vonage API Account Manager to submit the templates. Only templates created in your own namespace are valid. Using an template with a namespace outside of your own results in an error code 1022 being returned.

> **NOTE:** Templates are subject to a restriction of 60 characters in their header and footer, and 1024 characters in their body.

MTMs are designed to reduce the likelihood of spam to users on WhatsApp.

For the purpose of testing Vonage provides a template, `whatsapp:hsm:technology:nexmo:verify`, that you can use:

``` shell
{{1}} code: {{2}}. Valid for {{3}} minutes.
```

The parameters are an array. The first value being `{{1}}` in the MTM.

Below is an example API call:

``` shell
curl -X POST \
  https://api.nexmo.com/beta/messages \
  -H 'Authorization: Bearer' $JWT \
  -H 'Content-Type: application/json' \
  -d '{
   "from": "WHATSAPP_NUMBER",
   "to": "TO_NUMBER",
   "channel": "whatsapp",
   "whatsapp": {
     "policy": "deterministic",
     "locale": "en-GB"
   }
   "message_type": "template",
   "template":{
      "name":"whatsapp:hsm:technology:nexmo:verify",
      "parameters":[
         "Vonage Verification",
         "64873",
         "10"
      ]
   }
}'
```

## WhatsApp deterministic language policy

> **NOTE:** WhatsApp deprecated the "fallback" locale method when sending template messages on January 1st 2020. As of April 8, 2020, messages bearing the "fallback" policy will fail with a 1020 error in your message status webhook.

When a message template is sent with the deterministic language policy, the receiving device will query its cache for a *language pack* for the language and locale specified in the message. If not available in the cache, the device will query the server for the required language pack. With the deterministic language policy the target device language and locale settings are ignored. If the language pack specified for the message is not available an error will be logged.

Further information is available in the [WhatsApp documentation](https://developers.facebook.com/docs/whatsapp/message-templates/sending/#language).

## WhatsApp Provisioning API

The WhatsApp Provisioning API enables you to deploy a WhatsApp cluster, perform one time password (OTP) verification, and update profile information for a WhatsApp business account.

For more information see the [WhatsApp Provisioning API](/messages/whatsapp-provisioning/overview) documentation.

## Further information

* [Custom objects](/messages/concepts/custom-objects)
* [Interactive Messages: Overview](/messages/concepts/whatsapp-interactive-messages)
* [Working with Interactive Messages](/messages/concepts/working-with-whatsapp-interactive-messages)

WhatsApp developer documentation:

* [WhatsApp Developer documentation](https://developers.facebook.com/docs/whatsapp)
* [Text Message](https://developers.facebook.com/docs/whatsapp/api/messages/text)
* [Media Message](https://developers.facebook.com/docs/whatsapp/api/messages/media)
* [Message Template](https://developers.facebook.com/docs/whatsapp/api/messages/message-templates)
* [Media Message Template](https://developers.facebook.com/docs/whatsapp/api/messages/message-templates/media-message-templates)
* [Contacts message](https://developers.facebook.com/docs/whatsapp/api/messages/others#contacts-messages)
* [Location message](https://developers.facebook.com/docs/whatsapp/api/messages/others#location-messages)
* [Interactive Message](https://developers.facebook.com/docs/whatsapp/guides/interactive-messages)
