[Calendar Sync] Integration Design

# Calendar Sync Feature

As a customer I want to connect the MadeUp Co. application to a single calendar provider (ie.
google) with multiple calendars (ie. my-work-calendar) and have the event information provided by my
provider to synchronize and display on my MadeUp Co. dashboard.

### Requirements

- Only the calendar organizer and system admin may CRUD the event
- Any Person within the application may be invited to an event
- When an event changes on the provider it is reflected in MadeUp Co. (push notifications)
- The user should be able to create events on any permitted calendar offered by the provider

## Summary

The following outlines the high level view of how we will introduce this MVP. The business logic
should be separated from the functional code in order to allow for shared resources across multiple
3rd parties. Introduce OmniAuth for handling authentication via a simple strategy. Although a gem is
provided by the 3rd party we are opting not to use it as there are several threads indicating that
it comes with a lot of bloat and we only have need of a few endpoints. We would also like to use our
HTTP implementation to ensure that the response object matches with our other integrations.

### Workflow

![sequence-diagram]()

## Work Brief

The following outline provide an initial pass as to the quantity of work that would need to be
completed to have an MVP for this feature.

- [Main Project]
  - Build the [3rd Party] Rest Party Client Calendars CRUD
    - Events CRUD
    - PN Subscription CRUD
  - Build Omniauth Strategy
  - Models
    - [3rd Party] Channel: Tracks push notification subscriptions for a user
    - [3rd Party] Auth Token: Tracks state for the user Oauth2 token
  - Build Service:
    - Webhooks handler
    - Calendars
    - Channels
    - Events
    - Auth

### Outstanding Questions

- None

### Action Items

#### Route Updates

- /api/calendar/notifications/create
  - Definition: Endpoint to handle any push notifications from 3rd party
  - Methods: POST
  - Responds To: JSON
- /calendars_event(/new)(/:id/edit)(/:id)
  - Definition: Route handles workflows relating to CRUD of calendar events for a user
  - Methods: CREATE, READ, DELETE, LIST
  - Responds To: HTML

#### ERD Updates

**Calendar Event**

- Definition: Tracks event state for a user event
- Required Resource Views: CRUD & List

| Note | Field       | Type    | Definition                                    |
| ---- | ----------- | ------- | --------------------------------------------- |
| PK   | id          | integer |                                               |
|      | uuid        | string  | The 3rd party record ID                       |
|      | start_time  | string  | ISO8601 Time with zone                        |
|      | end_time    | string  | ISO8601 Time with zone                        |
|      | location    | string  | Free text identifier (max 140 char)           |
|      | summary     | string  | Free text title of event (max 140 char)       |
|      | description | string  | Free text description of event (max 600 char) |
| FK   | matter_id   | integer | ID of linked matter                           |
|      | Timestampse | string  | create & update                               |

**Calendar Event Invitation**

- Definition: Join table linking many Persons to a Calendar Event record
- Required Resource Views: None

| Note | Field      | Type    | Definition         |
| ---- | ---------- | ------- | ------------------ |
| PK   | ID         | integer |                    |
| FK   | invited_id | integer | A Person record ID |
| FK   | event_id   | integer | A CalendarEvent ID |

**Calendar Subscription**

- Definition: Details for a push notification subscription for a user
- Required Resource Views: Create, Update & Delete

| Note | Field        | Type    | Definition                                       |
| ---- | ------------ | ------- | ------------------------------------------------ |
| PK   | id           | integer |                                                  |
|      | uuid         | string  | The 3rd party record ID                          |
|      | calendar_ids | array   | The 3rd party calendars included in subscription |
| FK   | user_id      | integer | The user ID                                      |

**[3rd Party] Oauth2 Token**

- Definition: Manages details for the user oauth2 token
- Required Resource Views: None

| Note | Field         | Type    | Definition              |
| ---- | ------------- | ------- | ----------------------- |
| PK   | id            | integer |                         |
|      | access_token  | string  | The API access token    |
|      | refresh_token | string  | The refresh token       |
|      | expires_at    | string  | ISO8601 expiration time |
| FK   | user_id       | integer | The user ID             |
