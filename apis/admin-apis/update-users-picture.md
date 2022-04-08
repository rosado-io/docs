# Update user's picture through Admin API

Authgear allows your server to update the user's picture via the Admin APIs.

## Overview

To update the user's picture:

1. Make a `GET` request to `/_api/admin/images/upload` to obtain the pre-signed upload url.
1. Make a `POST` request to the pre-signed upload url to upload the image file. This call returns the result url once the upload is finished.
1. Call the Admin GraphQL API `/_api/admin/graphql` to update the user's standard attributes. Update the `picture` attribute with the result url returned by the previous call.

## API Details

### Obtain the per-signed upload url

Make a `GET` request to `/_api/admin/images/upload` to obtain the pre-signed upload url.

The call will look like this:

```http
GET /_api/admin/images/upload HTTP/1.1
Host: <YOUR_APP>.authgearapps.com
Authorization: Bearer <JWT>
```

The call returns the pre-signed upload url:

```json
{
  "result": {
    "upload_url": "<PRESIGNED_UPLOAD_URL>"
  }
}
```

### Upload the picture

Upload the picture to the pre-signed upload url. We recommend you to pre-process the image before uploading. The image should be cropped into square and should be less than 10 MB.

Upload the file with FormData and the field name is `file`. You can construct the `multipart/form-data` request by most HTTP libraries.

The call will look like this:

```http
POST <PRESIGNED_UPLOAD_URL_PATH> HTTP/1.1
Host: <YOUR_APP>.authgearapps.com
Content-Length: <CONTENT_LENGTH>
Content-Type: multipart/form-data; boundary=----boundary

----boundary
Content-Disposition: form-data; name="file"; filename="image.png"
Content-Type: image/png

<CONTENT_OF_IMAGE_FILE>
----boundary
```

The call returns the result url once the upload completed, the url is in format of `authgearimages:///APP_ID/OBJECT_ID`:

```json
{
  "result": {
    "url": "authgearimages:///..."
  }
}
```

### Update user's picture in the standard attributes

In this part, we use the Admin GraphQL API to update the standard attributes.

Before updating the picture attribute, you need to obtain the original standard attributes of the user first.

#### Get the user's standard attributes by query

The query:

```graphql
query ($userID: ID!) {
  node(id: $userID) {
    ... on User {
      standardAttributes
    }
  }
}
```

The variables:

```json
{
  "userID": "<BASE64_ENCODED_USER_ID>"
}
```

The sample response:

```json
{
  "data": {
    "node": {
      "standardAttributes": {
        "given_name": "John",
        "name": "User",
        "picture": "https://<THE_PICTURE_URL>",
        "preferred_username": "user01",
        "updated_at": 1649363483
      }
    }
  }
}
```

#### Update the user's standard attributes by mutation

Copy the original standard attributes, replace the picture with the new picture url obtained by the previous upload.

The query:

```graphql
mutation ($userID: ID!, $standardAttributes: UserStandardAttributes!) {
  updateUser(input: {userID: $userID, standardAttributes: $standardAttributes}) {
    user {
      id
      standardAttributes
      updatedAt
    }
  }
}
```

The variables:

```json5
{
  "userID": "<BASE64_ENCODED_USER_ID>",
  "standardAttributes": {
    /* Include all the original standard attributes from the previous response */
    /* and replace the picture url only */
    "picture": "authgearimages:///...."
  }
}
```

The sample variables:

```json
{
  "userID": "<BASE64_ENCODED_USER_ID>",
  "standardAttributes": {
    "given_name": "John",
    "name": "User",
    "preferred_username": "user01",
    "picture": "authgearimages:///..."
  }
}
```

The sample response:

```json
{
  "data": {
    "updateUser": {
      "user": {
        "id": "<BASE64_ENCODED_USER_ID>",
        "standardAttributes": {
          "given_name": "John",
          "name": "User",
          "picture": "https://<THE_PICTURE_URL>",
          "preferred_username": "user01",
          "updated_at": 1649368033
        }
      }
    }
  }
}
```