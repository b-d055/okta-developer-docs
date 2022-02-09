---
title: Telephony Inline Hook Reference
excerpt: Customizes Okta's flows that send SMS or Voice messages
---

# Telephony Inline Hook Reference

This page provides reference documentation for:

- JSON objects that are contained in the outbound request from Okta to your external service

- JSON objects that you can include in your response

This information is specific to the Telephony Inline Hook, one type of Inline Hook supported by Okta.

## See also

For a general introduction to Okta Inline Hooks, see [Inline Hooks](/docs/concepts/inline-hooks/).

For information on the API for registering external service endpoints with Okta, see [Inline Hooks Management API](/docs/reference/api/inline-hooks/).

For steps to enable this Inline Hook, see below, [Enabling a Token Inline Hook](#enabling-a-token-inline-hook).

For an example implementation of this Inline Hook, see [Token Inline Hook](/docs/guides/token-inline-hook#Create-Inline-Hook).

## About

The Okta Telephony Inline Hook allows you to integrate your own custom code into several of Okta's flows that send SMS or CALL.
This hook is currently available to be integrated with Enrollment, Authentication and Recovery flows that involve phone
factor as an authenticator. While sending the OTP (One Time Passcode) to the requester, Okta calls out to your external
service to deliver the OTP, and your service can respond with commands that indicate success or failure in delivering the OTP.

Only one active telephony inline hook is allowed to be created for any org.

All telephony hooks require authentication field and secret to be specified while creating the hook. It is passed as the
`authScheme` parameter during hook creation. See [how to create inline hooks](/docs/reference/api/inline-hooks/)

## Objects in the request from Okta

For the Telephony Inline Hook, the outbound call from Okta to your external service includes the following objects in its JSON payload:

### data.userProfile

Provides information on the OTP requester.

| Property | Description                   | Data Type                    |
|----------|-------------------------------|------------------------------|
| firstName   | First name of the OTP requester. | String     |
| lastName | Last name of the OTP requester.        | String |
| login | Okta Login of the OTP requester.        | String |
| userId | Okta userId of the OTP requester.        | String |

### data.messageProfile

Provides information on the properties of the message being sent to OTP requester.

| Property | Description                        | Data Type                    |
|----------|------------------------------------|------------------------------|
| msgTemplate   | SMS message template. Not applicable for CALL.      | String     |
| phoneNumber | Phone number enrolled for phone factor by OTP requester.             | String |
| otpExpires   | Time when the OTP expires. | String     |
| deliveryChannel   | OTP delivery method - SMS or CALL. | String     |
| otpCode   | OTP code. | String     |
| locale   | Locale of the OTP requester. | String     |
| eventAction   | Event for which OTP is requested - authentication, enrollment, recovery. | String     |

## Objects in the response that you send

For the Telephony Inline Hook, the `commands` and `error` objects that you can return in the JSON payload of your response are defined as follows:

### commands

The `commands` object is where you can provide commands to Okta. It is where you can tell Okta whether your attempt to
send the OTP using your own telephony provider was successful.

The `commands` object is an array, allowing you to send multiple commands.
In each array element, there needs to be a `type` property and `value` property. The `value` property is where you specify
the status of your telephony transaction and other relevant transaction metadata.

In the case of the Telephony hook type, the `value` property is itself a nested object in which you specify a status, provider,
transaction id and transaction metadata.

| Property | Description                                                              | Data Type       |
|----------|--------------------------------------------------------------------------|-----------------|
| type     | One of the [supported commands](#supported-commands).                    | String          |
| value    | Result of the send OTP operation. It specifies the outcome of whether the OTP was sent successfully by external webservice/provider. | [value](#value) |

#### Supported commands

The following commands are supported for the Telephony Inline Hook type:

| Command                 | Description             |
|-------------------------|-------------------------|
| com.okta.telephony.action | Telephony operation action result.     |

#### value

The `value` object is where you specify the result of the send OTP operation.

| Property | Description                                                                                                                                                                                                       | Data Type       |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| status       | Whether the OTP was sent successfully using customer's webservice. | [status](#status)          |
| provider     | Provider that was used for sending OTP using customer's webservice. | String          |
| transactionId    | Transaction ID that uniquely identifies an attempt to deliver OTP to requester. | String |
| transactionMetadata    | Any relevant transaction metadata. e.g. duration | String |

#### Status

| Status      | Description               |
|---------|---------------------------|
| SUCCESSFUL     | External webservice was able to deliver the OTP to requester. |
| PENDING | External webservice was not able to confirm delivery of the OTP to requester. |
| FAILED  | External webservice was unable to deliver the OTP to requester. |

### error

When you return an error object, it should have the following structure:

| Property     | Description                          | Data Type |
|--------------|--------------------------------------|-----------|
| errorSummary | Human-readable summary of the error. | String    |

Returning an error object causes Okta to retry sending the OTP to requester using Okta's telephony provider(s).

> **Note:** If the error object doesn't include the `errorSummary` property defined, the following common default message is returned to the end user: `The callback service returned an error`.

## Sample JSON payload of a request

```json
{
  "eventId": "uS5871kJThSsU8qlA1LTcg",
  "eventTime": "2022-01-28T21:43:40.000Z",
  "eventType": "com.okta.telephony.provider",
  "eventTypeVersion": "1.0",
  "contentType": "application/json",
  "cloudEventVersion": "0.1",
  "source": "https://${yourOktaDomain}/api/v1/inlineHooks/calz6lVQA77AwFeEe0g3",
  "data": {
    "context": {
      "request": {
        "id": "reqRgSk8IBBRhuo0YdlEDTmUw",
        "method": "POST",
        "url": {
          "value": "/api/internal/v1/inlineHooks/com.okta.telephony.provider/generatePreview"
        },
        "ipAddress": "127.0.0.1"
      }
    },
    "userProfile": {
      "firstName": "test",
      "lastName": "user",
      "login": "test.user@okta.com",
      "userId": "00uyxxSknGtK8022w0g3"
    },
    "messageProfile": {
      "msgTemplate": "(HOOK)Your code is 11111",
      "phoneNumber": "9876543210",
      "otpExpires": "2022-01-28T21:48:34.321Z",
      "deliveryChannel": "SMS",
      "otpCode": "11111",
      "locale": "EN-US",
      "eventAction": "com.okta.user.telephony.pre-enrollment"
    }
  }
}
```

## Sample JSON payloads of responses

This section provides example JSON payloads for the supported operations.

### Sample response for successful OTP delivery

```json
{
  "commands":[
    {
      "type":"com.okta.telephony.action",
      "value":[
        {
          "status":"SUCCESSFUL",
          "provider":"VONAGE",
          "transactionId":"SM49a8ece2822d44e4adaccd7ed268f954",
          "transactionMetadata":"Duration=300ms"
        }
      ]
    }
  ]
}
```

### Sample response for failed OTP delivery

```json
{
  "error":{
    "errorSummary":"Failed to deliver SMS OTP to test.user@okta.com",
    "errorCauses":[
      {
        "errorSummary":"Provider could not deliver OTP",
        "reason":"The content of the message is not supported",
        "location":"South Africa"
      }
    ]
  }
}
```

## Timeout behavior

After receiving the Okta request, if there is a response timeout, the Okta process flow proceeds with trying to send the
OTP using Okta's telephony provider(s). See [Troubleshooting](#troubleshooting).

## Troubleshooting

This section covers what happens when a telephony inline hook flow fails either due to the external inline hook service returning an error object or not returning a successful response.

> **Note:** Administrators can use the [Okta System Log](/docs/reference/api/system-log/) to view errors. See the [Troubleshooting](/docs/concepts/inline-hooks/#troubleshooting) section in the Inline Hooks concept piece for more information on the events related to Inline Hooks that the Okta System Log captures.

- When there is a communication failure with the external service, a timeout for example, the inline hook operation is skipped.
  OTP is delivered to requester using Okta's telephony provider(s).

  **Who can see this error?** Administrators

- When the external service returns a response with any other HTTP status code besides `200`, the inline hook operation is skipped.
  OTP is delivered to requester using Okta's telephony provider(s).

  **Who can see this error?** Administrators

- When the external service returns an error object in the response, the telephony inline hook flow fails with no OTP delivered using the hook.

  **Who can see this error?** Administrators, developers, and end users.

- When a hook response is malformed or could not be mappeed to expected API response, the inline hook operation is skipped.
  The OTP is delivered to requester using Okta's telephony provider(s). There may be scenarios where requester may receive
  duplicate SMS/CALL because of external service and Okta provider(s) both attempting to deliver the OTP.


  **Who can see this error?** Administrators

  The following actions result in an error:

  - Using an invalid status in response

  - Attempting to add an active telephony hook when one already exists

  - Not including auth scheme in request header