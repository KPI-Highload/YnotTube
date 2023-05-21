# Comments

The service is responsible for comments on videos.

As any other service, it requires Redis instance that can be used as a fast cache and that is synced with the Redis instance of the User Management Service (all authentication tokens are synced).

Also, any endpoint that receives the access token in in the Authorization header uses that token to authenticate the user and obtain their ID for further operations.

# DB schema

**Comments**:
| Column      | Type            | Description                              |
| ----------- | --------------- | ---------------------------------------- |
| `id`        | integer or uuid | unique id of like (counter or random id) |
| `content`   | text            | body of comment                          |
| `created_at`| timestamp       | the time of comment                      |
| `user_id`   | uuid            | the id of a user                         |
| `video_id`  | uuid            | the id of a video that user liked        |

# Endpoints

## Comment Video

An endpoint for adding the comment on a video. It does so just by inserting a new record into the `comments` table with the following fields:
- `content`
- `created_at`
- `user_id`
- `video_id`

**Endpoint**: `POST /videos/<video_id>/comments`

**Input**:   The `video_id` parameter in the URL, the `access_token` in the Authorization header, and JSON object containing the following field: 
- `content`

**Output**: JSON object containing the following field:
- `comment_id`: unique identifier of the comment.
- `video_id`: unique identifier of the video.
- `content`: body of the comment.

## Edit Comment

Edit comment content. It does so by updating the entry in the database with the corresponding `comment_id`.

**Endpoint**: `PUT /videos/<video_id>/comments/<comment_id>`

**Input**: The `video_id` and `comment_id` parameters in the URL, the `access_token` in the `Authorization` header, and JSON object containing the following field: 
- `content`

**Output**: JSON object containing the following fields:
- `message`: A confirmation message indicating that the comment was edited successfully.

## Delete Comment

Removes the comment from video. It does so by removing the entry in the database with the corresponding `comment_id`.

**Endpoint**: `DELETE /videos/<video_id>/comments/<comment_id>`

**Input**: The `video_id` and `comment_id` parameters in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON object containing the following field:
- `message`: A confirmation message indicating that the comment was removed successfully.

## Get Video Comments

Retrieve all comments for the particelar video. Requesting is paginated and the comments are retrieved from the database ordered from the newest.

**Endpoint**: `GET /videos/<video_id>/comments`

**Input**:
- The `limit` and `offset` query parameters in URL.
- The `video_id` in url path.

**Output**: JSON array of `comment` objects, each containing the following fields:
- `video_id`: The ID of the video that was commented
- `user_id`: The ID of the user
- `username`: The string with username
- `comment_id`: unique identifier of the comment.
- `content`: The string with comment
- `created_at`: The time when the video was commented

Additional headers for caching the result in the private cache can also be added for the same amount of time.

## Get User Commented Videos

Retrieve all commented videos of a particular user. Requesting is paginated and the likes are retrieved from the database ordered from the newest.

**Endpoint**: `GET /users/<user_id>/comments`

**Input**:
- The `limit` and `offset` query parameters in URL.
- The `user_id` in url path.
- The `access_token` in the `Authorization` header.

**Output**: JSON array of `commented_video` objects, each containing the following fields:
- `video_id`: The ID of the video that was liked
- `created_at`: The time when the video was liked

Additionally, the response can contain some headers for caching strategy. For example, the response can be cached for 30-60 seconds.
