---
title: Two-way SMS for customer engagement
products: messaging/sms
description: Programmable SMS is not only useful for one way notifications. When you combine outbound notifications with inbound messages you create chat-like interactions between your company and your customers.
languages:
    - Ruby
---

# Two-way SMS for customer engagement

Programmable SMS is not only useful for one way notifications. When you combine outbound notifications with inbound messages you create chat-like interactions between your company and your customers.

## In this tutorial

You see how quick it is to build two-way communication into your app; you send a delivery notification to your customer's phone number and handle their reply when they want to change a delivery slot.

The workflow for your app is:

```sequence_diagram
Participant App
Participant Vonage
Participant Phone number
App->>Vonage: Request to SMS API
Vonage-->>App: Response from SMS API
Note over Vonage: Request accepted
Vonage->>Phone number: Send delivery notification SMS
Phone number->>Vonage: Reply to delivery notification
Vonage-->>App: Send reply to webhook endpoint
App->>Vonage: Request to SMS API
Vonage-->>App: Response from SMS API
Note over Vonage: Request accepted
Vonage->>Phone number: Send acknowledgement in SMS
```

To do this:

1. [Configure a Vonage virtual number](#configure-a-nexmo-virtual-number) - rent a virtual number and set the webhook endpoints for inbound messages
2. [Create a basic Web app](#create-a-basic-web-app) - create a Web app to collect your customer's phone number.
3. [Send an SMS notification](#send-an-sms-notification) - send your customer a delivery notification in an SMS and request a reply.
4. [Process the reply SMS](#process-the-reply-sms) - process and acknowledge the SMS reply.

## Prerequisites

In order to work through this tutorial you need:

* A [Vonage account](https://dashboard.nexmo.com/sign-up)
* A publicly accessible Web server so Vonage can make webhook requests to your app. If you're developing locally you must use a tool such as [ngrok](https://ngrok.com/)
* The source code for this tutorial from <https://github.com/Nexmo/ruby-customer-engagement/>.
* All US based customers must register a brand and campaign to comply with [10 DLC guidelines](/messaging/sms/overview#send-an-sms).

## Configure a Vonage virtual number

Vonage forwards inbound messages to the webhook endpoint associated with your Vonage virtual number.

You manage virtual numbers using the [Developer API](/api/developer/numbers) or the [Vonage CLI](https://github.com/vonage/vonage-cli). The following examples use the Vonage CLI to rent a Vonage number:

```sh
$ vonage numbers:search US
$ vonage numbers:buy 1201###### US
```

Then associate the virtual number with your webhook endpoint that [handles inbound SMS](#process-the-reply-sms):

```sh
$ vonage numbers:sms 1201###### http://www.example.com/update
Number updated
```

> **Note**: ensure your server is running and publicly available before associating the webhook endpoint with your virtual number. Vonage must receive a 200 OK response from your webhook endpoint for successful configuration. If you're developing locally use a tool such as [ngrok](https://ngrok.com/) to expose your local web server to the Internet.

Now that you have configured your virtual number, you can send SMS delivery notifications.

## Create a basic Web app

Use [Sinatra](http://www.sinatrarb.com/) to create a single page Web app:

**Gemfile**

```ruby
source 'https://rubygems.org'

# our web server
gem 'sinatra'
```

**app.rb**

```ruby
# web server and flash messages
require 'sinatra'

# load environment variables
# from .env file
require 'dotenv'
Dotenv.load

# Index
# - collects a phone number
#
get '/' do
  erb :index
end
```

Add an HTML form to collect the phone number you will send a notification SMS to:

**views/index.erb**

```erb
<form action="/notify" method="post">
  <div class="field">
    <label for="number">
      Phone number
    </label>
    <input type="text" name="number">
  </div>

  <div class="actions">
    <input type="submit" value="Notify">
  </div>
</form>
```

The form captures the phone number in the [E.164](https://en.wikipedia.org/wiki/E.164) format expected by SMS API:

## Send an SMS notification

In this tutorial, to send an SMS you add the [Vonage Server SDK for Ruby](https://github.com/Nexmo/nexmo-ruby) to your app:

**Gemfile**

```ruby
# the nexmo library
gem 'vonage'
# a way to load environment
# variables
gem 'dotenv'
```

Use your Vonage API [key and secret](/concepts/guides/authentication) to initialize the client:

**app.rb**

```ruby
# Vonage library
require 'vonage'
vonage = Vonage::Client.new(
  api_key: ENV['VONAGE_API_KEY'],
  api_secret: ENV['VONAGE_API_SECRET']
)
```

> **Note**: do not store your API credentials in your code, use environment variables.

To receive replies to your notification SMS, you set your virtual number as the SenderID for outbound messages when you make a request to [SMS API](/api/sms):

**app.rb**

```ruby
# Notify
# - Send the user their delivery
#   notification, asking them
#   to respond back if they
#   want to make any changes
#
post '/notify' do
  notification = "Your delivery is scheduled for tomorrow between " +
                 "8am and 2pm. If you wish to change the delivery date please " +
                 "reply by typing 1 (tomorrow), 2 (Thursday) or 3 (deliver to"
                 "post office) below.\n\n";

  vonage.sms.send(
    from: ENV['VONAGE_NUMBER'],
    to: params['number'],
    text: notification
  )

  "Notification sent to #{params['number']}"
end
```

To verify that this SMS was received by the customer, check the [delivery receipt](/messaging/sms/guides/delivery-receipts). This tutorial does not verify delivery receipts.

## Process the reply SMS

When your customer replies to your notification SMS, Vonage forwards the [inbound message](/api/sms#inbound-sms) to the webhook endpoint associated with the virtual number.

In this tutorial app you process the incoming webhook, extract the text and number, and send a confirmation message back to the customer.

**app.rb**

```ruby
# Receive incoming message
#
# - Receives incoming SMS
#   message, stores it, and
#   notifies sender
#
get '/update' do
  choice = params['text']
  number = params['msisdn']

  # You can store or validate
  # the choice made here

  message = "Thank you for picking option #{choice}. " +
            "Your delivery is now fully scheduled in."

  vonage.sms.send(
    from: ENV['VONAGE_NUMBER'],
    to: number,
    text: message
  )

  body ''
end
```

Storing and validating customers input is beyond the scope of this tutorial.

Now reply back to the SMS you received earlier. You should see it be processed by your app and receive a confirmation of your choice within seconds.

## Conclusion

That's how quick it is to send and receive SMS in your app. With a few lines of code you have sent a SMS to a customer's phone with the SMS API, handled a reply, and responded with a confirmation.

## Get the Code

All the code for this tutorial is available in the [Two-way SMS for customer engagement GitHub repo](https://github.com/Nexmo/ruby-customer-engagement).

## Resources

* [Ruby Server SDK](https://github.com/Vonage/vonage-ruby-sdk)
* [SMS](/sms)
* [SMS API reference guide](/api/sms)
