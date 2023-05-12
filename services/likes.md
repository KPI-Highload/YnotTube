# Likes

The service is responsible for tracking likes on videos.

As any other service, it requires Redis instance that can be used as a fast cache and that is synced with the Redis instance of the User Management Service (all authentication tokens are synced).

Also, any endpoint that receives the access token in in the Authorization header uses that token to authenticate the user and obtain their ID for further operations.

# DB schema

**Likes**:
| Column      | Type            | Description                              |
| ----------- | --------------- | ---------------------------------------- |
| `id`        | integer or uuid | unique id of like (counter or random id) |
| `user_id`   | uuid            | the id of a user                         |
| `video_id`  | uuid            | the id of a video that user liked        |
| `timestamp` | timestamp       | The time of like                         |

# Endpoints

## Like Video

An endpoint for registering the like on a video. It does so just by inserting a new record into the `likes` table with the following fields:
- `user_id`
- `video_id`
- `timestamp`

**Endpoint**: `POST /videos/video_id/like`

**Input**: The `video_id` parameter in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON object containing the following field:
- `message`: A confirmation message indicating that the like was recorded successfully.

## Unlike Video

Removes the like from video. It does so by removing the entry in the database with the corresponding `user_id` and `video_id`.

**Endpoint**: `DELETE /videos/<video_id>/like`

**Input**: The `video_id` parameter in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON object containing the following field:
- `message`: A confirmation message indicating that the like was removed successfully.

## Check if video is liked

An endpoint for checking if a video is liked by a particular user. To do so, the endpoint just verifies if there is a record in database with the given `video_id` and `user_id`.

However, to ensure high availability, the query result is cached for some time in Redis. For example, it may be 10 minutes as users don't often change their mind about some video.

**Endpoint**: `GET /videos/<video_id>/like`

**Input**: The `video_id` parameter in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON object containing the following field:
- `is_liked`: true, if video is liked by this user.

Additionally, the request can contain headers for caching the response for some amount of time.

## Get Video Likes

Returns a number of likes on a video. It does so by querying the database.

However, to ensure high availability, the query result is cached for some time in Redis (for example, 30 seconds).
Additional headers for caching the result in the private cache can also be added for the same amount of time.

**Endpoint**: `GET /videos/<video_id>/likes`

**Input**: The `video_id` parameter in the URL.

**Output**: HTTP response with the cache headers and JSON object in the body containing the following field:
- `likes_count`: The total number of likes for the video.

## Get User Likes

Retrieve all liked videos of a particular user. Requesting is paginated and the likes are retrieved from the database ordered from the newest.

**Endpoint**: `GET /videos/likes`

**Input**:
- The `limit` and `offset` query parameters in URL.
- The `access_token` in the `Authorization` header.

**Output**: JSON array of `like` objects, each containing the following fields:
- `videoId`: The ID of the video that was liked
- `timestamp`: The time when the video was liked

Additionally, the response can contain some headers for caching strategy. For example, the response can be cached for 30-60 seconds.
