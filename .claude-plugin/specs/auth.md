# Mapao - Auth

### `POST /api/v1/auth/accept-invite`

_Accept Invite_

Public endpoint — consume an invite token and finalize the account.

Two flows:
  * Fresh invite (no user row yet): we create the user with the role+email
    the admin chose, set the password to what the user typed.
  * Re-invite of an existing user: we just update password + activate.

Returns a token pair so the UI can log the user in directly.

**Auth:** public

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `token` | `string` | yes |  |
| `password` | `string` | yes |  |
| `full_name` | `string | null` | no |  |

**Responses**

- **200** Successful Response - `TokenResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/auth/external/{tenant_slug}/{provider_slug}/authorize`

_Authorize External Login_

**Auth:** public

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `tenant_slug` | path | `string` | yes |  |
| `provider_slug` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `ExternalAuthorizeResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/auth/external/{tenant_slug}/{provider_slug}/callback`

_External Auth Callback_

**Auth:** public

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `tenant_slug` | path | `string` | yes |  |
| `provider_slug` | path | `string` | yes |  |
| `state` | query | `string` | yes |  |
| `code` | query | `string` | yes |  |

**Responses**

- **200** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/auth/forgot-password`

_Forgot Password_

Email a one-time reset link. Always 200 to avoid revealing which
addresses are registered.

**Auth:** public

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | `string` | yes |  |

**Responses**

- **200** Successful Response - `GenericMessageResponse`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/auth/login`

_Login_

Login and get JWT tokens.

On top of the per-IP slowapi rate limit, we also keep a per-(email, ip)
failure counter in Redis. After ``login_lockout_max_attempts``
consecutive 401s the endpoint returns 429 for the rest of the window.
This is specifically aimed at credential stuffing that rotates IPs —
slowapi can't see through that, but the email side of the key can.

**Auth:** public

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | `string` | yes |  |
| `password` | `string` | yes |  |

**Responses**

- **200** Successful Response - `TokenResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/auth/login-options/{tenant_slug}`

_Get Login Options_

**Auth:** public

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `tenant_slug` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `LoginOptionsResponse`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/auth/logout`

_Logout_

Revoke a refresh token so it can no longer be used for rotation.

Always clears the auth cookies first, even when no valid refresh token
is provided — that way ``logout`` is idempotent for all clients
(cookie-only browsers, bearer-only scripts, sessions where the token
is already revoked or expired).

Access tokens remain valid until their 30-minute expiry; cookie clients
are immediately logged out via cookie deletion. Bearer-only clients
should discard their access token locally on logout.

**Auth:** public

**Request body** (`application/json`)

**Responses**

- **204** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/auth/me`

_Get Current User Profile_

Get current user profile from JWT.

**Auth:** required (cookie or Bearer)

**Responses**

- **200** Successful Response - `AuthMeResponse`


### `POST /api/v1/auth/refresh`

_Refresh Token_

Rotate a refresh token: revoke the presented one and issue a new pair.

Reuse of an already-revoked refresh token is treated as an intrusion
signal: every token of that user is revoked (family revocation) and
the caller gets 401.

**Auth:** public

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `refresh_token` | `string` | yes |  |

**Responses**

- **200** Successful Response - `TokenResponse`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/auth/register`

_Register_

Register a new user in an existing tenant.

Refuses by default — tenants must opt *in* by setting
``tenant.settings.allow_self_signup = true``. This flipped from
open-by-default in Sprint 1.4 to stop random registrations on freshly
created tenants (the old default let anyone with the slug sign up as a
member). The expected flow is now: admin creates the tenant → admin
invites real users via ``/users/invite-link``.

**Auth:** public

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | `string` | yes |  |
| `password` | `string` | yes |  |
| `full_name` | `string | null` | no |  |
| `tenant_slug` | `string` | yes |  |

**Responses**

- **201** Successful Response - `UserResponse`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/auth/reset-password`

_Reset Password_

Consume a reset token and set the new password.

On success we also bump the user's ``tokens_invalidated_at`` so every
outstanding access / refresh token for that user is revoked.

**Auth:** public

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `token` | `string` | yes | Opaque token from the email link |
| `new_password` | `string` | yes |  |

**Responses**

- **200** Successful Response - `GenericMessageResponse`
- **422** Validation Error - `HTTPValidationError`

