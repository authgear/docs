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

Follow the doc of [Update user's standard attributes](./update-users-standard-attributes.md) to update the user's `picture` attribute. Replace the attribute with the new picture url (e.g. `authgearimages:///...`) obtained by the previous upload.
