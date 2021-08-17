---
title: Okta Identity Engine API Products Release Notes 2021
---
<ApiLifecycle access="ie" /><br>
<ApiLifecycle access="Limited GA" /><br>

## August

### Weekly Release 2021.08.2

| Change                                                                     | Expected in Preview Orgs |
|----------------------------------------------------------------------------|--------------------------|
| [Bugs fixed in 2021.08.2](#bugs-fixed-in-2021-08-2)                          | August 18, 2021          |

#### Bugs fixed in 2021.08.2

- In the [Device Authorization grant flow](/docs/guides/device-authorization-grant/main/), the URI link that was used in a QR Code was missing if the org wasn't on Okta Identity Engine. (OKTA-413425)

- When using the `/token` endpoint, OAuth 2.0 refreshed the [access and ID tokens](/docs/guides/refresh-tokens/overview/) for all application users, which included deactivated users instead of just active users. (OKTA-417991)

- In cases where a suspended user, deactivated or unknown user, or a valid user with the wrong password tried to sign in, an HTTP 400 response code was returned instead of an HTTP 401 response code. (OKTA-418923)

### Weekly Release 2021.08.1

| Change                                                                     | Expected in Preview Orgs |
|----------------------------------------------------------------------------|--------------------------|
| [Bug fixed in 2021.08.1](#bug-fixed-in-2021-08-1)                          | August 11, 2021          |

#### Bug fixed in 2021.08.1

When Single Sign-On (SSO) was used with an IdP that included a `fromURI` parameter, an HTTP 500 error was returned. (OKTA-407425)