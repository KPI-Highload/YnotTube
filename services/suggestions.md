# Suggestions Microservice

This service implements suggestions of videos.

As any other service, it requires Redis instance that can be used as a fast cache and that is synced with the Redis instance of the User Management Service (all authentication tokens are synced).

# DB schema

**Videos (Read-only)**:
| Column         | Type            | Description                |
| -------------- | --------------- | -------------------------- |
| `id`           | integer or uuid | Unique ID of video         |
| `title`        | string          | Title of the video         |
| `desription`   | text            | Video description          |
| `hosting_link` | string          | Link to the video hosting  |
| `preview_link` | string          | Link to the preview        |
| `image_link`   | string          | Video thumbnail            |
| `duration`     | int             | Duration                   |
| `size`         | int             | Size of the video          |
| `created_at`   | timestamp       | The time of video creation |
| `category_id`  | uuid            | The ID of a category       |
| `user_id`      | uuid            | The ID of a user           |
| `channel_id`   | uuid            | The ID of a channel        |

**Views (Read-only)**:
| Column     | Type            | Description                        |
| ---------- | --------------- | ---------------------------------- |
| `id`       | integer or uuid | Unique ID of view                  |
| `user_id`  | uuid            | The ID of a user                   |
| `video_id` | uuid            | The ID of a video that user viewed |

**Comments (Read-only)**:
| Column      | Type            | Description                              |
| ----------- | --------------- | ---------------------------------------- |
| `id`        | integer or uuid | Unique ID of like (counter or random ID) |
| `content`   | text            | Body of comment                          |
| `created_at`| timestamp       | The time of comment                      |
| `user_id`   | uuid            | The id of a user                         |
| `video_id`  | uuid            | The id of a video that user liked        |

**Likes (Read-only)**:
| Column      | Type            | Description                              |
| ----------- | --------------- | ---------------------------------------- |
| `id`        | integer or uuid | Unique ID of like (counter or random ID) |
| `user_id`   | uuid            | The ID of a user                         |
| `video_id`  | uuid            | The ID of a video that user liked        |
| `timestamp` | timestamp       | The time of like                         |

# Endpoints

## Make suggestions

This endpoint retrieves a video that is considered to be a good suggestion to user, using some algorithm. Criteria for suggestion are based on categories and tags of viewed videos, user comments and likes. Requesting is paginated and the suggestions are retrieved from the database and are ordered in accordance to the suggestion algorithm.

**Endpoint**: `GET /suggestions`

**Input**: the `access_token` in the `Authorization` header.

**Output**: JSON array of `suggestion` objects, each containing the following fields:

- `video_id`: The ID of a suggested video.
- `title`: The title of the video.
- `preview_link`: The URL of a preview of the video.
- `image_link`: The URL of a thumbnail of the video.
- `duration`: The duration of the video.
