# Inbound-Calling

### Background

As a customer, I want call calls to auto-populate the source, caller phone #, and intake
Specialist. Additionally, I would like call calls to be automatically recorded and attached to
the related case.

### Requirements

- The caller is notified automatically that their call may be recorded
- When a new intake is created the following fields are prepopulated
- Source: The number called connected to an ad campaign
- Phone Number: The caller's phone number
- Assigned Intake Specialist: (New) the user assigned to the intake
- Unknown calls(not lined to a campaign) are routed to the default user and manually entered
- Known call calls are routed to the Intake Specialist (paralegal/attorney)
- Unanswered calls go to voicemail
- Calls are attached to a case in under 1 hours time
- Calls may be answered from within the app
- ALL calls will result in an intake being started

### Additional Documentation

- [twiML](https://www.twilio.com/docs/voice/twiml): Twilio’s XML mark-up
- [answering-calls](https://www.twilio.com/docs/voice/tutorials/how-to-respond-to-incoming-phone-calls-ruby_): How to answer calls with Twilio
- [twilio-call-recording](https://www.twilio.com/docs/voice/api/recording): Record calls
- [twilio-call-tracking](https://www.twilio.com/docs/voice/tutorials/call-tracking-ruby-rails): track effectiveness of marketing campaigns
- [call queue](https://www.twilio.com/docs/voice/queue-calls): How to queue up calls

## Answering Inbound Phone Calls Inside The App

### Summary

This approach assumes that calls will be answered from within the app. If the agent is not logged in
or otherwise cannot answer then calls will go to voicemail.

When adeUp Co. receives a call notification the provided endpoint generates a call record which is
linked to a campaign record. A new intake is started using the provided information from the
webhook. After informing the caller that the call may be recorded, the call is placed into a call
queue relating to intakes. The specialists may navigate to the intake queue dashboard and select
calls manually as the appear. Only active calls will be present. From here the specialist may
initiate a new pre-filled intake using details gleaned from the active call. Once the call
completes, a “post call” workflow polls and retrieves Twilio for the call recording and attaches it
to the matter.

### Workflow

markdown: [sequence-diagram](https://github.com/rubyDoomsday/architecture-design-diagram/blob/main/telephony/sequence-diagram.seq)

![sequence-diagram](https://github.com/rubyDoomsday/architecture-design-diagram/blob/main/telephony/sequence-diagram.png)

## Work Brief

The following outline provides an initial pass as to the quantity of work that would need to be
completed to have an MVP for call recording and intake generation via the app.

### Action Items

The following notes are a high level first pass at what routes additions and modifications along
with the database, resources would be needed at a minimum to support this feature.

### Routes:

There are existing routes for phones which handle outbound calls and SMS messaging. Additionally,
there is a route for outbound calls and SMS messages.

#### /api/webhooks/sms (move from /api/twilio_inbound_sms)

- Definition: Endpoint to handle any SMS notifications from 3rd party
- Methods: POST
- Responds To: XML

#### /api/phone/webhooks/call

- Definition: Endpoint provided to handle any webhooks from 3rd party for receiving calls.
- Methods: POST
- Responds To: XML

#### /api/phone/webhooks/voicemail

- Definition: Endpoint to handle any webhooks from 3rd party for call queue timeout or full. When a
  call queue is full or the caller has been on hold for too long this workflow is triggered.
- Methods: POST
- Responds To: XML
-

#### /api/phone/webhooks/disconnect

- Definition: Endpoint to handle any webhooks from 3rd party for call ending or call disconnect.
- This would be responsible for post call workflows such as retrieving the call recording.
- Methods: POST
- Responds TO: XML

#### /phone/sms(/new)(/:id) (move from /phones)

- Definition: Route handles workflows relating to the sending of SMS messages
- Methods: CREATE, READ, DELETE, LIST
- Responds To: JSON/HTML
-

#### /phone/call(/new)(/:id) (move from /phones)

- Definition: Route handles workflows relating to the outbound calling
- Methods: CREATE, READ, DELETE, LIST
- Responds To: JSON/HTML

#### /phone/queues(/:id)

- Definition: The queues endpoint handles actions for call queue dashboard views. This allows for
  expanding to multiple queues.
- Methods: DELETE, READ, LIST
- Responds to: JSON/HTML

#### /phone/queues/:id/calls(/:id)

- Definition: The calls endpoint allows listing all calls for a queue or connecting to a specific
  call resource. Optionally these can be filtered to the logged in user.
- Methods: LIST, READ
- Responds to: JSON/HTML

### DB Models

#### Call Campaign

- Definition: Details of pertinent to an advertising campaign
- Required Resource Views: CRUD & List

| Note | Field                   | Type    | Definition                              |
| ---- | ----------------------- | ------- | --------------------------------------- |
| PK   | id                      | integer |                                         |
|      | office_phone_number     | integer | the formatted phone number (routed to ) |
|      | advertised_phone_number | integer | the formatted phone number (public)     |
| FK   | marketing_source_id     | integer | the ID of the related marketing type    |
|      | active                  | boolean | enabled or disabled for use             |
