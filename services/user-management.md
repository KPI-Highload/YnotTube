# User management service

This service will handle user registration, authentication, and management of user profiles. When a user registers, this service will also create a user channel. It will expose APIs for registration, login, and profile updates.

The service is self-contained on its own. However, it still requires an access to some (possibly external) service for delivering emails and fast cache for tokens, like Redis.

# DB schema

**Users:**

| Column            | Type       | Description                                                              |
| ----------------- | ---------- | ------------------------------------------------------------------------ |
| `id`              | uuid       | Unique identifier of the user. This ID is referenced from other services |
| `email`           | string     | Email address of the user                                                |
| `password_hash`   | PHC string | (argon2id) Hash of the user password                                     |
| `username`        | string     | Username of the user. By default, the first channel uses this name       |
| `full_name`       | string     | Full name of the user. Most often, it's first+last name                  |
| `created_at`      | timestamp  | Time of creation                                                         |
| `updated_at`      | timestamp  | Time of updating                                                         |
| `email_confirmed` | bool       | Whether the user confirmed their email                                   |
| `localization`    | string     | Country and language specific information.                               |

# Endpoints

## User Registration

An endpoint for user registration. Verifies that:
- the username and email are not used
- the password is strong

If ok, generates new `id` for the user, hashes the password with some password-based KDF (Argon2id) and stores all information in the `Users` table.

To confirm the email address, service generates a short lived JWT token `confirm_token` with user `id`.
Then, it communicates with the **Email** service to send a confirmation link to the user's address with this token:
```
https://<address>/users/confirm-email?token=<confirm_token>
```

**Endpoint**: `POST /users/register`

**Input**: JSON object containing the following fields:
- `email`: The email address of the user.
- `password`: The password for the user's account.
- `username`: The username of the user.
- `full_name`: The full name of the user.
- `localization`: The language specific information.


**Output**: JSON object containing the following fields:
- `id`: The unique identifier of the created user.
- `message`: A confirmation message indicating that the user registration was successful (string).

## User Login

Authenticates the user.

The endpoint verifies that the account exists and that the password is correct. However, users can login only if they confirmed their email (`email_confirmed` is `true`).

The result of authentication is a short-lived `access_token`. It's generated as a random string and saved in the external cache together with the `user_id` and expiration time.

**Endpoint**: `POST /users/login`

**Input**: JSON object containing the following fields:
- `email`: The email address of the user.
- `password`: The password for the user's account.

**Output**: JSON object containing the following fields:
- `access_token`: A token for authenticating subsequent requests.
- `user_id`: The unique identifier of the logged-in user.
- `username`: The username of the logged-in user.

## Confirm Email

This endpoint is used to confirm the email.

The `token` is validated. If it's valid, the `email_confirmed` is set to `true` for the user.
If it's not valid, an error message with suggestion to go to `/users/<user_id>/resend-email-confirmation` if the token is expired.

**Endpoint**: `GET /users/confirm-email`

**Input**: `token` field in URL parameters.

**Outputs**:
- Empty response, if the token is valid.
- Generic error message in other casese.

When email is confirmed, the user is considered registered. Therefore, in the end of confirmation the service generates a message about the new user and sends it to a message broker with the following fields:
- `id`
- `email`
- `username`
- `full_name`
- `localization`

This message is used by other services. For example, the channel service can use it to register the new channel for the user.

## Resend confirmation

The endpoint resends the same `confirm_token` as during the registration. However, if the user has already confirmed their token, the endpoint responds with an error.

**Endpoint**: `GET /users/resend-email-confirmation?user_id=<user_id>`

**Input**: `user_id` field in the URL params.

**Outputs**: A confirmation message indicating that the resend request was successful or not.


## Get User Profile

This endpoint allows retrieving generic user information. The information can be retrieved only by the user themselves.

**Endpoint**: `GET /users/<user_d>`

**Input**: The `user_id` parameter in the URL, and the `access_token` in the Authorization header for authentication.

**Output**: JSON object containing the following fields:
- `id`: The unique identifier of the user.
- `email`: The email address of the user.
- `username`: The username of the user.
- `full_name`: The full name of the user.
- `localization`: The language specific information.

## Update User Profile

This endpoint allows updating separate fields of user information. The information can be changed only by the user themselves.


**Endpoint**: `PUT /users/<user_id>`

**Input**: The `user_id` parameter in the URL, the `access_token` in the Authorization header for authentication, and a JSON object containing the following **optional** fields:
- `username`: The new username of the user.
- `full_name`: The new full name of the user.
- `localization`: The new language specific information.

Output: JSON object containing the following fields:
- `message`: A confirmation message indicating that the user profile was updated successfully or not


## Request Password Reset

An endpoint to request a password reset to an email.

When this endpoint is called, the server generates a JWT short-lived `reset_token` with the user's ID. Then, it uses the email service to send and email with a link to the reset page. The link includes the reset token.


**Endpoint**: `POST /users/request-password-reset`

**Input**: JSON object containing the following field:
- `email`: The email address of the user requesting a password reset

**Output**: A generic message to not leak information whether the email exists or not.

## Reset Password

This endpoint resets the user password. The user should provide the `reset_token`, issued in the `/users/request-password-reset`. Then, the service:
1. Verifies that the token is valid and not expired.
2. Derives the hash of the new password and update the user's password in the database.
3. Send a confirmation email to the user's email address to inform them that their password has been changed.

**Endpoint**: `POST /users/reset-password?reset-token=<token>`

**Input**:
- `reset-token`: The password reset token in URL query parameters.
- `new_password`: The new password for the user's account in JSON body of the request.

**Output**: JSON object containing the following field:
  - `message`: A confirmation message indicating that the password reset was successful.


## Logout

The endpoint that invalidates the access token. It does so by removing the token from the cache.

**Endpoint**: `POST /users/logout`

**Input**: The `access_token` in the Authorization header for authentication.

**Output**: JSON object containing the following field:
- `message`: A confirmation message indicating that the logout was successful.

## Authenticate user

This endpoint is used by other endpoints to authenticate the user. It does so by checking if the access token provided by the user exists in the cache. If it does, the endpoint extracts the user ID and returns it in the response.

**Endpoint**: `GET /users/auth-check`.

**Input**: The `user_id` parameter in the URL, and the `access_token` in the Authorization header for authentication.

**Output**: JSON object containing the following fields:
- `is_authenticated`: A boolean value indicating whether the user is authenticated or not.
- `user_id`: The user ID, if the user is authenticated.
