# Twilio WhatsApp Adapter (beta)
This is part of the [Bot Builder Community Extensions](https://github.com/BotBuilderCommunity/botbuilder-community-js) project which contains various pieces of middleware, recognizers and other components for use with the Bot Builder JavaScript SDK v4.

The Twilio WhatsApp adapter for the Microsoft Bot Framework allows you to add an additional endpoint to your bot for use with Twilio WhatsApp. The goal of this adapter is to support WhatsApp via Twilio as seamlessly as possible. All supported features on WhatsApp are mapped to the default Bot Framework SDK.

>To send messages with WhatsApp in production, you have to wait for WhatsApp to formally approve your account. But, that doesn't mean you have to wait to start building. [Twilio Sandbox for WhatsApp](https://www.twilio.com/console/messaging/whatsapp) lets you test your app in a developer environment.

This adapter supports the limited capabilities of Twilio WhatsApp, including;
* Send and receive text messages
* Send and receive text messages with attachment (`image`, `audio`, `video`, `document`)
* Send proactive notifications
* Track message deliveries (`sent`, `delivered`, `read` receipts)

## Status
__Currently the Twilio WhatsApp channel is in beta.__
>Products in Beta may occasionally fall short of speed or performance benchmarks, have gaps in functionality, and contain bugs.

>APIs are stable and unlikely to change as the product moves from Beta to Generally Available (GA). We will do our best to minimize any changes and their impact on your applications. Products in Beta are not covered by Twilio's SLA's and are not recommended for production applications.

## Installation

To install:

    npm install @botbuildercommunity/adapter-twilio-whatsapp --save

## Usage

1. Use your existing Twilio account or create a new Twilio account.
2. Go to the [Twilio Sandbox for WhatsApp](https://www.twilio.com/console/messaging/whatsapp) and follow the first steps.
3. At the `Configure your Sandbox` step, add your endpoint URLs. Those URLs will be defined by the snippet below, by default the URL will be `[your-bot-url]/api/whatsapp/messages`.
_The `status callback url` is optional and should only be used if you want to track deliveries of your messages._
4. Go to your [Dashboard](https://www.twilio.com/console/sms/dashboard) and click on `Show API Credentials`.
5. Implement the snippet below and add your Account SID, Auth Token, your phone number and the endpoint URL you configured in the sandbox.
6. Give it a try! Your existing bot should be able to operate on the WhatsApp channel via Twilio.

```javascript
const whatsAppAdapter = new TwilioWhatsAppAdapter({
    accountSid: '', // Account SID
    authToken: '', // Auth Token
    phoneNumber: '', // The From parameter consisting of whatsapp: followed by the sending WhatsApp number (using E.164 formatting)
    endpointUrl: '' // Endpoint URL you configured in the sandbox, used for validation
});

// WhatsApp endpoint for Twilio
server.post('/api/whatsapp/messages', (req, res) => {
    whatsAppAdapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.run(context);
    });
});
```

## Advanced

### Send attachments

**Example**
```javascript
const reply = {
    type: 'message',
    text: 'This is a message with an attachment.',
    attachments: [
        {
            contentType: 'image/png',
            contentUrl: 'https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png'
        }
    ]
};

await context.sendActivity(reply);
```

### Send proactive notifications
Proactive notifications are supported using the default SDK functions. Read more about how to [send proactive notifications to users](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-proactive-message?view=azure-bot-service-4.0&tabs=csharp).

A WhatsApp session begins with a user initiated message to your app. Sessions are valid for 24 hours after the most recently received message, during which time you can communicate with them using free form messages. In order to send a message outside the 24 hour Session window, you must use a pre-approved template (see [Sending Notifications](https://www.twilio.com/docs/sms/whatsapp/api#sending-notifications) section).

**Example**
```javascript
// Capture conversation reference
conversationReference = TurnContext.getConversationReference(context.activity);

// Send pro-active message
await whatsAppAdapter.continueConversation(conversationReference, async (turnContext) => {
    await turnContext.sendActivity(`Proactive message!.`);
});
```

### Implement channel-specific functionality
The Twilio WhatsApp channel is using `whatsapp` as the channel id. Within the TurnContext, you can use the following snippet to detect if the request is coming from the Twilio WhatsApp channel and implement your custom logic.
`if (context.activity.channelId === 'whatsapp)'`

Using the `channelData` object on new message activities is not supported, since there is no WhatsApp specific functionality that isn't supported by the Bot Framework SDK.

### Monitor the status of your WhatsApp outbound message
If you configure the `status callback url` in Twilio Configuration, multiple status events will be broadcasted to your bot. You can use this functionality to [monitor the status of your WhatsApp outbound message](https://www.twilio.com/docs/sms/whatsapp/api#monitor-the-status-of-your-whatsapp-outbound-message). Possible values include: 'messageRead', 'messageDelivered', 'messageSent', 'messageQueued', 'messageFailed'

Within the TurnContext you are able to differentiate between the events by reading the value of `context.activity.type`.

**Example (JavaScript)**
```javascript
if (context.activity.type === 'messageRead') {}
```

**Example (TypeScript)**
```typescript
if (context.activity.type === WhatsAppActivityTypes.MessageRead) {}
```