# Suggestions Microservice

This service implements suggestions of videos.

As any other service, it requires Redis instance that can be used as a fast cache and that is synced with the Redis instance of the User Management Service (all authentication tokens are synced).

# Endpoints

## Make suggestions

This endpoint retrieves a video that is considered to be a good suggestion to user, using some algorithm. Criteria for suggestion are based on categories and tags of viewed videos, user comments and likes. Requesting is paginated and the suggestions are retrieved from the database and are ordered in accordance to the suggestion algorithm.

**Endpoint**: `GET /suggestions`

**Input**: the `access_token` in the `Authorization` header.

**Output**: JSON array of `suggestion` objects, each containing the following field:

- `video_id`: The ID of a suggested video.
