# Videos

The service is responsible for information about videos and views.

As any other service, it requires Redis instance that can be used as a fast cache and that is synced with the Redis instance of the User Management Service (all authentication tokens are synced).

Also, any endpoint that receives the access token in in the Authorization header uses that token to authenticate the user and obtain their ID for further operations.

# DB schema

**Videos**:
| Column         | Type            | Description                |
| -------------- | --------------- | -------------------------- |
| `id`           | integer or uuid | unique id of video         |
| `title`        | string          | title of the video         |
| `desription`   | text            | video description          |
| `hosting_link` | string          | link to the video hosting  |
| `preview_link` | string          | link to the preview        |
| `image_link`   | string          | video thumbnail            |
| `duration`     | int             | duration                   |
| `size`         | int             | size of the video          |
| `created_at`   | timestamp       | the time of video creation |
| `category_id`  | uuid            | the id of a category       |
| `user_id`      | uuid            | the id of a user           |
| `channel_id`   | uuid            | the id of a channel        |

**Categories**:
| Column | Type            | Description           |
| ------ | --------------- | --------------------- |
| `id`   | integer or uuid | unique id of category |
| `name` | string          | category name         |

**VideosTags**:
| Column     | Type            |
| ---------- | --------------- |
| `video_id` | integer or uuid |
| `tag_id`   | integer or uuid |

**Tags**:
| Column | Type            | Description          |
| ------ | --------------- | -------------------- |
| `id`   | integer or uuid | unique id of a tag   |
| `name` | string          | unique name of a tag |

**Views**:
| Column     | Type            | Description                        |
| ---------- | --------------- | ---------------------------------- |
| `id`       | integer or uuid | unique id of view                  |
| `user_id`  | uuid            | the id of a user                   |
| `video_id` | uuid            | the id of a video that user viewed |

# Endpoints

## Add Video Details
An endpoint for adding the video. It does so just by inserting a new records into the `Videos`, `VideosTags`, `Tags` table with the following fields:
  
**Endpoint**: `POST /videos`

**Input**: the `access_token` in the Authorization header, and JSON object containing the following field:           
- `title`: the title of the newly added video
- `desription`: the description of the newly added video
- `hosting_link`: the URL of the newly added video's location
- `preview_link`: the URL of a preview of the newly added video
- `image_link`: the URL of a thumbnail of the newly added video
- `duration`: the length of the newly added video
- `size`: the size of the newly added video
- `category_id`: the category of the newly added video
- `channel_id`: the channel of the newly added video
- `tags`: an array of strings representing the tags or keywords associated with the newly added video

**Output**: JSON object containing the following field:
- `video_id`: unique identifier of the video.
- `message`: A confirmation message indicating that the video info was added successfully.

## Get Video Details

An endpoint that allows to retrieve video details, such as title, description and other metadata.

**Endpoint**: `GET /videos/<video_id>`

**Input**:
- The `video_id` in url path.

**Output**: JSON object containing the following fields:
- `title`: the title of the video
- `desription`: the description of the video
- `hosting_link`: the URL of the video's location
- `preview_link`: the URL of a preview of the video
- `image_link`: the URL of a thumbnail of the video
- `duration`: the length of the video
- `size`: the size of the video
- `category`: object containing the following fields:
  - `category_id`
  - `category_name`
- `channel_id`: the channel of the video
- `tags`: an array of objects containing the following fields:
  - `tag_id`
  - `tag_name`

Additional headers for caching the result in the private cache can also be added for the same amount of time.

## Edit Video Details

An endpoint that edit video details. It does so by updating the entry in the database with the corresponding `video_id`.

**Endpoint**: `PUT /videos/<video_id>`

**Input**: The `video_id` parameter in the URL, the `access_token` in the Authorization header, and JSON object containing the following optional fields:           
- `title`: the new title of the newly added video
- `desription`: the new description of the newly added video
- `preview_link`: the URL of a preview of the newly added video
- `image_link`: the URL of a thumbnail of the newly added video
- `category_id`: the category of the newly added video
- `tags`: an array of strings representing the tags or keywords associated with the newly added video


**Output**: JSON object containing the following fields:
- `message`: A confirmation message indicating that the video details were edited successfully.

## Delete Video

An endpoint that removes the video. It does so by removing the entry in the database with the corresponding `video_id` and sends delete request to video hosting.

**Endpoint**: `DELETE /videos/<video_id>`

**Input**: The `video_id` parameter in the URL and the `access_token` in the `Authorization` header.

**Output**: JSON object containing the following field:
- `message`: A confirmation message indicating that the video was deleted successfully.

## Get Categories

Retrieve all categories.

**Endpoint**: `GET /categories`

**Output**: JSON array of `category` objects, each containing the following fields:
- `category_id`: the ID of the category
- `category_name`: the name of category

## Get List of Videos

An endpoint that allows to search videos by name, category and tags by using ElasticSearch. If optional parameters are not specified - request suggestions from Suggestions Microservice.

**Endpoint**: `GET /videos`

**Input**:
Optional parameters:
- `channel_id`: parameter to specify the channel ID to get the videos of a certain channel or search within the channel
- `search_query`: string with search query
- `category`: name of the category
- `tags`: video tags

**Output**: JSON object containing the following fields:
- `title`: the title of the video
- `preview_link`: the URL of a preview of the video
- `image_link`: the URL of a thumbnail of the video
- `duration`: the length of the video

## Get Video Views

An endpoint that return video views.

**Endpoint**: `GET /videos/<video_id>/views`

**Input**: The `video_id` parameter in the URL

**Output**: JSON object containing the following fields:
- `video_id`: unique identifier of the video.
- `views`: amount of views.
