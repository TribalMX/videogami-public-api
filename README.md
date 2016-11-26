
# videogami-public-api
Videogami Public API

# Overview

The Videogami API allows a developer to automatically create clips
that capture important moments in a video or live stream. You will
need to set up the basics, e.g. to tell Videogami which video or live
stream to analyse, and our platform will do the rest.

# Architecture

The architecture is pretty straight forward: each user of the API
corresponds to a user object. Each user object has many stream
objects, each stream has many videos, and each video has many
clips. Streams are similar to channels on YouTube or Twitch, videos
are the main class and contain the videos or live stream assets we'd
like to analyze, and clips are short snippets of their parent
video. Users of the API are not aware of each other.

# Workflow

The main workflow is as follows: as a user you will do a one-time set
up by creating a user object, which will give you credentials to
perform further actions on streams, videos, and clips. You may create
one stream and put all your videos there, or you may create multiple
streams each containing related videos.

To create a video, you'll need to tell Videogami the source of the
video, which can either be a video online, or a live stream that you
stream to our servers (docs coming soon). Once the video is ingested,
Videogami will start analysing it and generate clips.

NOTE. A user of the API should create only one user object.

# API End Points

All end points have root URL `http://api.videogami.co`.

```
POST /v6/user
GET  /v6/user/:username

GET  /v6/user/:username/streams
POST /v6/user/:username/stream
GET  /v6/user/:username/stream/:streamname

GET  /v6/user/:username/stream/:streamname/videos
POST /v6/user/:username/stream/:streamname/video
GET  /v6/user/:username/stream/:streamname/video/:id
GET  /v6/user/:username/stream/:streamname/video/:id/clips
```

## `POST /v6/user`

Create a user. Returns a user object containing the password required
for all future requests.

### Parameters

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| username      | String        | YES      | Username               |

### Example request

```
curl -X POST --data "username=bob" http://api.videogami.co/v6/user
```

### Example response

You'll need to save this password:

```
{
  "user": {
    "username": "bob",
    "password": "XXXXXXXXXX",
    "created": "2016-10-28T15:50:50.468Z",
    "_id": "581373da7bbf246e19a2b522"
  }
}
```

## `GET /v6/user/:username`

Get user. Returns a user object.

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |

### Example request

```
curl -X GET --data "password=XXXXXXXX" http://api.videogami.co/v6/user/bob
```

## `GET /v6/user/:username/streams`

Get all streams belonging to user, most recent first. Results limited
to 10 per page.

### Parameters

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |
| page          | Integer       | NO       | Pagination             |

### Example request

```
curl -X GET --data "password=XXXXXXXXXX&page=0" http://api.videogami.co/v6/user/bob/streams
```

### Example response

```
{
  "streams": [
    {
      "_id": "58109c4a57e1f85307f09547",
      "user": "bob",
      "name": "bobchannel17",
      "password": "XXXXXXXXXX",
      "created": "2016-10-26T12:06:34.983Z",
      "live": null,
    },
    ...
  ],
  "page": "0",
  "limit": 10,
  "total": 18
}
```

## `POST /v6/user/:username/stream`

Create a stream belonging to user. Returns a stream object containing
a password required for live streaming to this channel.

### Parameters

| Name          | Type          | Required | Description                  |
|---------------|---------------|----------|------------------------------|
| password      | String        | YES      | Password                     |
| streamname    | String        | YES      | Alphanumeric, <30 characters |

### Example request

```
curl -X POST --data "password=XXXXXXXXXX&streamname=bobchannel17" http://api.videogami.co/v6/user/bob/stream
```

### Example response

```
{
  "stream": {
    "user": "bob",
    "name": "bobchannel17",
    "password": "XXXXXXXXXX",
    "created": "2016-10-28T16:19:36.344Z",
    "live": null,
    "_id": "58137a987bbf246e19a2b60c"
  }
}
```

## `GET /v6/user/:username/stream/:streamname`

Get stream. Returns a stream object.

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |

### Example request

```
curl -X GET --data "password=XXXXXXXX" http://api.videogami.co/v6/user/bob/stream/bobchannel17
```

## `GET /v6/user/:username/stream/:streamname/videos`

Get all videos belonging to user, most recent first. Results limited
to 10 per page.

### Parameters

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |
| page          | Integer       | NO       | Pagination             |

### Example request

```
curl -X GET --data "password=XXXXXXXXXX&page=0" http://api.videogami.co/v6/user/bob/stream/bobchannel17/videos
```

### Example response

```
{
  "videos": [
    {
      "_id": "581362047bbf246e19a2b2c9",
      "user": "bob",
      "stream": "bobchannel17",
      "title": "Untitled",
      "created": "2016-10-28T14:34:44.777Z",
      "modified": "2016-10-28T15:18:41.179Z",
      "ready": true,
      "status": "transcoded",
      "autoclip": true,
      "screenshots": {
        "small": "https://www.vgtv.media/581362047bbf246e19a2b2c9.small.jpg",
        "medium": "https://www.vgtv.media/581362047bbf246e19a2b2c9.medium.jpg",
        "big": "https://www.vgtv.media/581362047bbf246e19a2b2c9.big.jpg"
      },
      "formats": {
        "original": "https://www.vgtv.media/581362047bbf246e19a2b2c9.original.flv",
        "mp4": "https://www.vgtv.media/581362047bbf246e19a2b2c9.mp4"
      },
      "playlist": "https://www.vgtv.media/581362047bbf246e19a2b2c9.m3u8",
      "src": "http://example.com/asdf.mp4",
      "length": 1961.645,
    },
    ...
  ],
  "page": 0,
  "limit": 10,
  "total": 9
}
```

## `POST /v6/user/:username/stream/:streamname/video`

Create a video belonging to a stream. Returns a video object. Use this
end point to ingest an online video into the system, and optionally
analyse it.

### Parameters

| Name          | Type          | Required | Description                  |
|---------------|---------------|----------|------------------------------|
| password      | String        | YES      | Password                     |
| src           | String        | YES      | URL of the video to ingest   |
| title         | String        | NO       | <30 characters               |
| autoclip      | Boolean       | NO       | Whether to generate automatic clips or not |

### Example request

```
curl -X POST --data "password=XXXXXXXXXX&autoclip=true" --data-urlencode "src=http://example.com/asdf.mp4" http://api.videogami.co/v6/user/bob/stream/bobchannel17/video
```

### Example response

```
{
  "video": {
    "_id": "581362047bbf246e19a2b2c9",
    "user": "bob",
    "stream": "bobchannel17",
    "title": "Untitled",
    "created": "2016-10-28T14:34:44.777Z",
    "modified": "2016-10-28T15:18:41.179Z",
    "ready": true,
    "status": "transcoded",
    "autoclip": true,
    "screenshots": {
      "small": "https://www.vgtv.media/581362047bbf246e19a2b2c9.small.jpg",
      "medium": "https://www.vgtv.media/581362047bbf246e19a2b2c9.medium.jpg",
      "big": "https://www.vgtv.media/581362047bbf246e19a2b2c9.big.jpg"
    },
    "formats": {
      "original": "https://www.vgtv.media/581362047bbf246e19a2b2c9.original.flv",
      "mp4": "https://www.vgtv.media/581362047bbf246e19a2b2c9.mp4"
    },
    "playlist": "https://www.vgtv.media/581362047bbf246e19a2b2c9.m3u8",
    "src": "http://example.com/asdf.mp4",
    "length": 1961.645,
  }
}
```

## `GET /v6/user/:username/stream/:streamname/video/:id`

Get video. Returns a video object.

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |

### Example request

```
curl -X GET --data "password=XXXXXXXX" http://api.videogami.co/v6/user/bob/stream/bobchannel17/video/581362047bbf246e19a2b2c9
```


## `GET /v6/user/:username/stream/:streamname/video/:id/clip`

Clip a video.

### Parameters

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |
| start         | Integer       | YES      | Start time of the clip in seconds |
| duration      | Integer       | NO       | Duration of the clip in seconds, defaults to 30, max 40. |
| title         | String        | NO       | Title of the clip      |

### Example request

```
curl -X GET --data "password=XXXXXXXXXX&start=256&duration=30" http://api.videogami.co/v6/user/bob/stream/bobchannel17/581362047bbf246e19a2b2c9/clip
```

### Example response

```
{
  "clip": {
    "_id": "58136dd77bbf246e19a2b44c",
    "user": "bob",
    "title": "Untitled clip",
    "created": "2016-10-28T15:25:11.578Z",
    "parent": {
      "pID": "581362047bbf246e19a2b2c9",
      "stream": "bobchannel17",
      "start": 256,
      "end": 286
    },
    "ready": true,
    "formats": {
      "flv": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.original.mp4",
      "mp4": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.mp4",
      "smp4": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.small.mp4"
    },
    "screenshots": {
      "small": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.small.jpg",
      "medium": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.medium.jpg",
      "big": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.big.jpg"
    },
    "length": 30,
  }
}
```

## `GET /v6/user/:username/stream/:streamname/video/:id/clips`

Get all videos belonging to user, most recent first. Results limited
to 10 per page.

### Parameters

| Name          | Type          | Required | Description            |
|---------------|---------------|----------|------------------------|
| password      | String        | YES      | Password               |
| page          | Integer       | NO       | Pagination             |

### Example request

```
curl -X GET --data "password=XXXXXXXXXX&page=0" http://api.videogami.co/v6/user/bob/stream/bobchannel17
```

### Example response

```
{
  "clips": [
    {
      "_id": "58136dd77bbf246e19a2b44c",
      "user": "bob",
      "title": "Autoclip",
      "created": "2016-10-28T15:25:11.578Z",
      "parent": {
        "pID": "581362047bbf246e19a2b2c9",
        "stream": "bobchannel17",
        "start": 256,
        "end": 286
      },
      "ready": true,
      "formats": {
        "flv": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.original.mp4",
        "mp4": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.mp4",
        "smp4": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.small.mp4"
      },
      "screenshots": {
        "small": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.small.jpg",
        "medium": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.medium.jpg",
        "big": "https://www.vgtv.media/581362047bbf246e19a2b2c9.256.30.big.jpg"
      },
      "length": 30,
    },
    ...
  ],
  "page": 0,
  "limit": 10,
  "total": 9
}
```
