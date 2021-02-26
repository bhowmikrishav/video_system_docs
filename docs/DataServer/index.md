# Data Server

[(Github Repo)](https://github.com/rishavbhowmik/video_system_data_server)

## Introduction

This microservice is a rest API-based system. The DataServer's role is to provide essential data navigating essential data from the system.

Data Server shares data with the client to provide video data, video manifest, and other data sets from the system that can be used for analytical purposes.

The Data server also performs tokenization of data to ensure secured access resources. This feature is essential for accessing video chunks via FSS(File Store System). FSS has mandatory use of `file_token`, for ensuring the safety of private data as an additional layer of security.



## Data Shared by Data Server


#### `my_video_list` : List of all the video of the logged User

This is a private endpoint, which can be accessed by a user after logging into the video_system. This endpoint returns the list of videos along with their processing status.

**Request Format**

```js
method: 'POST',

url: '/my_video_list',

body: {
	type: 'json',
	properties: {
		user_token: string, 	// JWT token of the user post login
		limit: number		// Maxmimum number of documents in result
	}
}
```

**Response Format**

```js
type: "application/json"
body: {
	title: string,  		// Video's title
	upload_time: number,		// Unix Apx. time in milli seconds
	duration: number | null		// Video duration in milli seconds
}
```

#### `get_manifest` : Video Manifest for the Video player to play a video

This endpoint is for obtaining video manifest of all public & permitted videos and keep the data accessible for a limited duration. 

This endpoint performs the following steps:-
- checks user's access to the video 
- finds the video_manifest of video from the `videos` collection from the database
- wrap `object_id` of video chunks in web-tokens with a time limit for token expiry
- Then return the tokenized document to the client

**Request Format**

```js
method: 'POST',
url: '/get_manifest',
body: {
  type: 'json',
  properties: {
    video_id: string,
    tokenify: string
  }
}
```

**Response Format**

```js
type: "application/json"
body: {
_id: mongodb.ObjectId,    // Uuid of the video
title: string,
upload_time: number,
upload_id: mongodb.ObjectId,
user_id: string,          // user_id of the video owner
stream_manifest: {
    '144': {
        user_id: string,  // user_id of the video owner
        video_id: string, // Uuid of the video
        duration: number, // Duration of the video in ms
        chunks: [{
            "object_id": string,    // object_id of the video chunk
            "object_token": string, // web token to access the video chunk via file store
            "height": 144,
            "width": number,
            "start_time": number,
            "end_time": number,
            "byte_length": number
        }]
    },
    '360': {
        user_id: string,
        video_id: string,
        duration: number,
        chunks: [{
            "object_id": string,
            "object_token": string,
            "height": 144,
            "width": number,
            "start_time": number,
            "end_time": number,
            "byte_length": number
        }]
    },
    '720': {
        user_id: string,
        video_id: string,
        duration: number,
        chunks: [{
            "object_id": string,
            "object_token": string,
            "height": 144,
            "width": number,
            "start_time": number,
            "end_time": number,
            "byte_length": number
        }]
    }
}
     
}
```

