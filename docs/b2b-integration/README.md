# Unforced Pickleball Business to Business (B2B) Partnership Documentation

## Introduction

Unforced Pickleball provides users with automated pickleball statistics,
primarily via automated video analysis. While most of our services are designed
for application users to interact with directly via on-device video upload, we
now offer Business to Business (B2B) capabilities that allow other
video-providers to integrate with our services to upload videos on behalf of
users.

As a video-provider business, this document will walk you through how you can
receive access to upload your videos to the UPB system, and how you can provide
your users the ability to access the uploaded video and submit it for
automated analysis.

## High-Level: How does this work?

Unforced Pickleball uses Amazon's [AWS S3](https://docs.aws.amazon.com/s3/)
service to store videos. For security and privacy reasons, our S3 bucket is
private - but we've provided a REST API endpoint that enables video-providers
authorized access to upload videos by
using [presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html).

This REST endpoint uses a simple shared API key to authenticate the
video-provider, and returns a pre-signed URL that the video-provider can use to
upload the video to our S3 bucket.

Once the video has been successfully uploaded to S3 bucket, the video-provider
will then send an email to the user containing a deep-link that allows the user
to continue the submission process. This deep-link will contain a unique video
identifier that is present in the pre-signed URL.

Here's a sequence diagram showing the whole process. Please see details below
about each step

![upb-b2b-sequence-diagram.png](upb-b2b-sequence-diagram.png)

## Obtaining an API Key

To obtain an API key, please contact Unforced Pickleball at
admin@unforcedpb.com, and we will generate and share an API key with you that
will be privileged to allow video uploads to our S3 bucket. This API key should
be kept secure and not shared with anyone else. If you suspect that your API
key has been compromised, please contact us immediately, and we will deactivate
the key and generate a new one for you.

## Using the API Key to generate an upload URL

One you've received an authorized API key, you can then use it in your
application to generate a pre-signed URL that can be used to upload videos to
our S3 bucket. The following is an example of how you can use the API key to
generate a pre-signed URL:

```curl
curl -X POST -H "X-API-Key: <API_KEY>" https://api.unforcedpickleball.com/video-upload-presigned-url
```

If authenticated correctly, you can expect to receive back a JSON response that
looks like this:

```json
{
  "upload_url": "https://upb-match-videos.s3.amazonaws.com/fdc6ba74-8f57-4460-b41a-d6e6318c5f83.mp4?AWSAccessKeyId=ASIAVHYEK2L63X5J7ZCR&Signature=gAQdN0BGneX2rSqCVDkr1sFsbNc%3D&x-amz-security-token=IQoJb3JpZ2luX2VjEHQaCXVzLXdlc3QtMiJGMEQCIFQ4s3XkqqotQUngCp3xSZgPDp09JPulF8CYU4J%2FNC2dAiBNxr5jeN%2Bk8EV4%2BsB97928hg2dFrghjnaRtcT1y06zBSrxAgid%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F8BEAIaDDM2MDI0OTQ3OTkzMyIMGPYzvIB8%2FFefrUo1KsUCjaurIikqJvN6LkMndfnOrIo%2BZUGaVIzgurTfjriRktghlEXFS7Gz4PV9NiHKr4FEzaMak2CpkbCo%2Bu2bS7YO3QRcVvst%2BVLgWznYBu4RAF4TMp3oAJpPGxAG7MjBS9WSnsXRJFZvHqwefndI8uszrRtxQarvjwo27WWJGVXaN4R8k%2FzHQcHnO%2Fiit9kOa5AIihNdzDo3HNzw%2BrlutbIv%2F0gkqqBR53Se%2BS8EH0Nb06gqftaA6cKsCUUZhuyg6rM7sw33vex5zDD8vEXNod47udd2ATrHPi8v1lFTf1ffNWpBapyWI%2BvxHyh%2BP4YuWTh7akxLUwIksVk8hFyKOCs%2F2B21MMWsgqmAikCMqIujDE7bTxPIsmrcvTiUqMZm%2B%2F3v%2Fqd%2FZyNs717%2BKENkv23ifm%2BlbLOHTOUDtLRcNSKemTjAc9AiMTC3xom3BjqfAXbVp8lk983D8XpRnVKytzWMunZeO99tIezUOzLZTG7B4HBOMuYg25eyPExMWVfGzLlOut8%2FHDpiFN03T6k3TEUvJYPm9o4A2MsEaAr5BlkrXNO%2Bz9LuJIRoZQ2o5KP%2BakMcw5OnZ43CgMbr%2BwjIBv4T6q4YVwuZyqkOsDpWz%2B2hdckZeAzos1EVLDtB0hfovxZfy%2FCe%2FnxCxfWYw65uxA%3D%3D&Expires=1726112617",
  "object_key": "fdc6ba74-8f57-4460-b41a-d6e6318c5f83.mp4"
}
```

The `upload_url` field contains a temporary pre-signed URL that allows you to
upload the video to our S3 bucket. The `object_key` field contains the unique
identifier for the video that you can use to reference the video in the email
to the user.

## Sending the email to the user

Once you've successfully uploaded the video to our S3 bucket, you can then send
an email to the user containing a deep-link formatted like this:

```
https://unforcedpickleball.com/app-redirect?deep_link=unforcedpb%3A//new-match/video-analysis/<OBJECT_KEY>
```

where `<OBJECT_KEY>` is replaced with the object_key that you received in the
response from the pre-signed URL generation.

Your email should instruct the user that tapping this link will open the
Unforced Pickleball app and allow them to submit the uploaded video for
automated analysis. Here's an example of how the email could look:

```
... Heading, business icons, styling, etc ...
To receive stats and analysis from Unforced Pickleball, tap <a href="https://unforcedpickleball.com/app-redirect?deep_link=unforcedpb%3A//new-match/video-analysis/abc-123.mp4">this link</a> to submit for processing.
```

Once this email has been sent, your job is done! The user will initiate the
video upload request process, and receive the results in their app.