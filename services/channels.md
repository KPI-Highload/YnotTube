# Channels Microservice

This service works with user channels. It handles channel creation, channel management, subscription management.

As any other service, it requires Redis instance that can be used as a fast cache and that is synced with the Redis instance of the User Management Service (all authentication tokens are synced).

# DB schema

**Channels:**


| Column        | Type      | Description                 |
| ------------- | --------- | --------------------------- |
| `id`          | uuid      | Unique ID of a user channel |
| `name`        | string    | Channel name                |
| `description` | string    | Channel description         |
| `created_at`  | timestamp | Time of creation            |
| `user_id`     | uuid      | The ID of a user            |

**Subscriptions:**


| Column       | Type            | Description                                        |
| ------------ | --------------- | -------------------------------------------------- |
| `id`         | integer or uuid | Unique id of a subscription (counter or random id) |
| `user_id`    | uuid            | The ID of a user                                   |
| `channel_id` | string          | Unique ID of a user channel                        |

# Endpoints

## Channel Creation

An endpoint for channel creation of a certain user. It also verifies that the channel name is unique. If so, it adds a new record to the `channels` table with the following fields:

- `name`
- `description`
- `created_at`
- `user_id`

**Endpoint**: `POST /channels`

**Input**: the `access_token` in the `Authorization` header and the JSON object with the following fields:

- `channel_name`: The name of a channel;
- `channel_description`: The description of a channel.

**Output**: JSON object containing the following field:
- `channel_id`: The ID of a newly created channel
- `message`: A confirmation message indicating that the channel was created successfully.

## Get Channel Information

This endpoint allows to obtain information about a certain channel.

**Endpoint**: `GET /channels/<channel_id>`

**Input**: The `channel_id` parameter in the URL.

**Output**: JSON object containing the following fields:

- `channel_id`: The unique ID of the channel.
- `channel_name`: The name of the channel.
- `channel_description`: The description of the channel.
- `created_at`: The creation time of the channel.

## Update Channel Information

This endpoint allows to update separate fields of channel information.

**Endpoint**: `PUT /channels/<channel_id>`

**Input**: The `channel_id` parameter in the URL, the `access_token` in the Authorization header, and a JSON object containing the following **optional** fields:

- `channel_name`: The new channel name.
- `channel_description`: The new channel description.

**Output**: JSON object containing the following field:

- `message`: A confirmation message indicating that the channel information was updated successfully.

## Get channels of a user

An endpoint that allows to obtain a list of all channels of a certain user.

**Endpoint**: `GET /users/<user_id>/channels`

**Input**: `user_id` parameter in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON array of `channel` objects, each containing the following fields:

- `channel_id`: The unique ID of the channel.
- `channel_name`: The new channel name.
- `channel_description`: The new channel description.
- `created_at`: The creation time of the channel.

## Subscribe to the channel

This endpoint allows a certain user to subscribe to the desired channel.
It adds a new record to the `subscriptions` table with the following fields:

- `user_id`
- `channel_id`

**Endpoint**: `POST /channels/<channel_id>/subscriptions`

**Input**: The `channel_id` parameter in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON object containing the following field:
- `message`: A confirmation message indicating that the user subscribed to the channel successfully.

## Get subscriptions to a channel

An endpoint that allows to obtain a number of subscriptions to a certain channel.

**Endpoint**: `GET /channels/<channel_id>/subscriptions`

**Input**: `channel_id` parameter in the URL.

**Output**: HTTP response with the cache headers and JSON object in the body containing the following field:

- `subscriptions_count`: The total number of subscriptions to the channel.

## Get subscriptions of a user

An endpoint that allows to obtain all subscriptions of a certain user. Requesting is paginated and the subscriptions are retrieved from the database ordered alphabetically.

**Endpoint**: `GET /users/<user_id>/subscriptions`

**Input**:

- The `limit` and `offset` query parameters in URL.
- The `user_id` in url path.
- The `access_token` in the `Authorization` header.

**Output**: HTTP response with the cache headers and JSON array of `subscription` objects, each containing the following field:

- `channel_id`: The ID of the channel that the user has subscribed to.
