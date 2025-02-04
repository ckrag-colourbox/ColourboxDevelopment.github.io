---
layout: page
title: "CDN"
permalink: /cdn/
layout: default
nav_order: 3
---

# CDN

## Getting started
CDN needs to be configured be it can be used. Please have a look at this [Getting Started page](https://www.skyfish.com/help/cdn-links/set-up-domain)

## Using CDN from the API
The CDN feature is actually just a "magic" folder. When files are added to it, a CDN link will automatically be generated (and removed again if the file is deleted from the folder). 

The first step is to figure out the id of the CDN folder. Note that a user can be in more than one company, so remember to specify the correct `company-id`. If in doubt what your company ID is, you can reach out to us on api@colourbox.com
```
curl -H "Authorization: CBX-SIMPLE-TOKEN Token=<token>" https://api.colourbox.com/cdn/<company-id>/company/aliases
```

It will give you something like:
```json
{
  "count": 1,
  "limit": 1,
  "aliases": [
    {
      "folder_id": 1337,
      "alias": "cdn-tutorial",
      "folder_name": "CDN"
    }
  ]
}
```

In this case the ID of the folder is `1337`. The `alias` is part of the URL of any links generated. In this case URLS will be on the format: `https://cdn.skyfish.com/cdn-tutorial/<id>`

### Generating a CDN link
Generating a CDN link for a file is as easy as to simply add it to the CDN folder found in the previos section. Assume you want to generate a link for the media with `unique_media_id` of `42424242`:

```
curl -H "Authorization: CBX-SIMPLE-TOKEN Token=<token>" -XPOST https://api.colourbox.com/media/42424242/folder/1337
```

Once the file is placed in the folder, the file will be queued for CDN link creation. Often the link will be ready within 20 seconds. 

**NOTE** You need full access to both the CDN folder and to at least one of the folders the file is placed in,. 

### Fetching the CDN link
All search endpoints supports asking for `cdn_urls` as a return value. To get the CDN link for a specific file, specify the `unique_media_id` as a search paraemter and request `cdn_urls` as a return values:

```
curl -H "Authorization: CBX-SIMPLE-TOKEN Token=<token>" api.colourbox.com/search?unique_media_id=42424242&return_values=cdn_urls
```

Response:

```json
{
  "response": {
    "media": [
      {
        "cdn_urls": [
          "https://cdn.skyfish.com/cdn-tutorial/<id>.<extension>"
        ]
      }
    ],
    "hits": 1
  },
  "media_count": 1,
  "media_offset": 0
}
```
The format of CDN links is: `https://cdn.skyfish.com/<alias>/<id>.<extension>`

The `id` is automatically generated by our system. The `.extension` part is optional, meaning that our API will always include it when asking for cdn urls, but the links work even if you ommit it. So assuming `12345` is a valid ID for a jpeg, the following URLS will both work:

- https://cdn.skyfish.com/cdn-tutorial/12345.jpeg
- https://cdn.skyfish.com/cdn-tutorial/12345


**NOTE** Because creating CDN links goes through our internal queuing system there can be some delay. In case `cdn_urls` is empty even though the file was placed in the CDN folder, wait a few seconds and try again.

### Changing the underlying file behind a CDN link
A great feature of the Skyfish CDN is that you can change the underlying file behind a CDN link. This means that after you have generated a link for a file, you can update the link to point to another file inside your Skyfish. One use case could be to use Skyfish CDN to host the header image for your homepage. In case you want to update it, you just update the file behind the link instead of changing the homepage it self. 

Following our example from above, to update the link to point to a new file with unique media id of `101` 

**Note** that this is a `PATCH` call
```
curl -H "Authorization: CBX-SIMPLE-TOKEN Token=<token>" -XPATCH api.colourbox.com/cdn/<company-id>/alias/cdn-tutorial/<id>/unique_media/101
```

### Deleting CDN links
To delete a CDN link simply remove the file from the folder. Continuing with our example, removing the media with `unique_media_id` of `42424242`
**Note** that this is a `DELETE` call. `1337` is the ID of our CDN folder. 
```
curl -H "Authorization: CBX-SIMPLE-TOKEN Token=<token>" -XDELETE api.colourbox.com/media/42424242/folder/1337
```

### Image manipulation on the fly
Our CDN supports image manipulation on the fly. It works by appending query parameters to the generated CDN link. 
We support:

| Parameter        | Description         
| ------------- |-------------
| width    | Width of the image
| height    | Height of the image
| grayscale    | Make the image grayscale
| output_format    | Format of the image

Our system automatically maintains aspect ratio, so you should **either** specify `width` or `height`.
AS for `output_format` we support

- webp
- jpeg

**Example**

https://cdn.skyfish.com/cdn-tutorial/12345.jpeg?width=800&output_format=webp

This will give you the file resized to be 800 in the width and the format is webp. 


