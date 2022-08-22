[<img width="250" alt="ImageKit.io" src="https://raw.githubusercontent.com/imagekit-developer/imagekit-javascript/master/assets/imagekit-light-logo.svg"/>](https://imagekit.io)
# ImageKit.io Python SDK

[![Python CI](<https://github.com/imagekit-developer/imagekit-python/workflows/Python%20CI/badge.svg>)](https://github.com/imagekit-developer/imagekit-python/)
[![imagekitio](<https://img.shields.io/pypi/v/imagekitio.svg>)](https://pypi.org/project/imagekitio)
[![codecov](https://codecov.io/gh/imagekit-developer/imagekit-python/branch/master/graph/badge.svg?token=CwKWqBIlCu)](https://codecov.io/gh/imagekit-developer/imagekit-python)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Twitter Follow](https://img.shields.io/twitter/follow/imagekitio?label=Follow&style=social)](https://twitter.com/ImagekitIo)

ImageKit Python SDK allows us to use [image resizing](https://docs.imagekit.io/features/image-transformations), [optimization](https://docs.imagekit.io/features/image-optimization), [file uploading](https://docs.imagekit.io/api-reference/upload-file-api), and other [ImageKit APIs](https://docs.imagekit.io/api-reference/api-introduction) from applications written in the Python language.

Supported Python Versions: >=3.6

Table of contents -
 * [Installation](#Installation)
 * [Initialization](#Initialization)
 * [URL Generation](#URL-generation)
 * [File Upload](#File-Upload)
 * [File Management](#File-Management)
 * [Utility Functions](#Utility-functions)
 * [Support](#Support)
 * [Links](#Links)


 ## Installation
 Go to a terminal and type the following command,
```bash
pip install imagekitio
```

## Initialization
```python
from imagekitio import ImageKit
imagekit = ImageKit(
    private_key='your private_key',
    public_key='your public_key',
    url_endpoint='your url_endpoint'
)
```

## Usage

One can use this Python SDK for three different methods - URL Generation, File Upload, and File Management.

## URL Generation

**1. Using Image path and image hostname or endpoint**

This method allows us to create a URL using the path where the image exists and the URL
endpoint(url_endpoint) you want to use to access the image. One can refer to the documentation
[here](https://docs.imagekit.io/integration/url-endpoints) to read more about URL endpoints
in ImageKit and the section about [image origins](https://docs.imagekit.io/integration/configure-origin) to understand paths with different kinds of origins.


```python
imagekit_url = imagekit.url({
    "path": "/default-image.jpg",
    "url_endpoint": "https://ik.imagekit.io/your_imagekit_id/endpoint/",
    "transformation": [{
        "height": "300", 
        "width": "400"
    }]
})
```

The result is in a URL like
```
https://ik.imagekit.io/your_imagekit_id/endpoint/tr:h-300,w-400/default-image.jpg
```

**2. Using full image URL**

This method allows us to add transformation parameters to an absolute URL using `src` parameter. One can use this method if the complete image URL is in a database.

```python
image_url = imagekit.url({
    "src": "https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg",
    "transformation" : [{
        "height": "300",
        "width": "400"
    }]
})
```

The result is in a URL like
```
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=h-300%2Cw-400
```


The ```.url()``` method accepts the following parameters.

| Option                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| url_endpoint            | Optional. The prepended base URL before the path of the image. If not specified, the URL Endpoint specified during SDK initialization gets used. For example, https://ik.imagekit.io/your_imagekit_id/endpoint/                                                                                                                                                                                                                                                                                                                                                                |
| path                    | Conditional. A path at which the image exists. For example, `/path/to/image.jpg`. Specify a `path` or `src` parameter for URL generation.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| src                     | Conditional. Complete URL of an image already mapped to ImageKit. For example, `https://ik.imagekit.io/your_imagekit_id/endpoint/path/to/image.jpg`. Specify a `path` or `src` parameter for URL generation.                                                                                                                                                                                                                                                                                                                                                                    |
| transformation          | Optional. Specify an array of objects with name and the value in key-value pair to apply transformation params in the URL. Append different steps of a [chained transformation](https://docs.imagekit.io/features/image-transformations/chained-transformations) as different objects of the array. This document includes a complete list of supported transformations in the SDK with some examples. If one uses an unspecified transformation name, it gets applied as it is in the URL. |
| transformation_position | Optional. The default value is `path`, which places the transformation string as a path parameter in the URL. One can also specify it as a `query`, which adds the transformation string as the query parameter `tr` in the URL. Suppose one uses the `src` parameter to create the URL. In that case, the transformation string is always a query parameter.                                                                                                                                                                                                                                                  |
| query_parameters        | Optional. These are the other query parameters that one wants to add to the final URL. These can be any query parameters and are not necessarily related to ImageKit. Especially useful if one wants to add some versioning parameter to their URLs.                                                                                                                                                                                                                                                                                                                                            |
| signed                  | Optional. Boolean. Default is `false`. If set to `true`, the SDK generates a signed image URL adding the image signature to the image URL. One can only use this if they create the URL with the `url_endpoint` and `path` parameters and not the `src` parameter.                                                                                                                                                                                                                                                                                                             |
| expire_seconds          | Optional. Integer. Used along with the `signed` parameter to specify the time in seconds from `now` when the URL should expire. If specified, the URL contains the expiry timestamp, and the image signature is modified accordingly.                                                                                                                                                                                                                                                                                                                                 |


## Examples of generating URLs
**1. Chained Transformations as a query parameter**

```python
image_url = imagekit.url({
    "path": "/default-image.jpg",
    "url_endpoint": "https://ik.imagekit.io/your_imagekit_id/endpoint/",
    "transformation": [{
        "height": "300",
        "width": "400"
    },
    {
        "rotation": 90
    }],
    "transformation_position ": "query"
})
```

The result is in a URL like
```
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=h-300%2Cw-400%3Art-90
```

**2. Sharpening and contrast transform a progressive JPG image**

Add transformations like [Sharpening](https://docs.imagekit.io/features/image-transformations/image-enhancement-and-color-manipulation)
to the URL with or without any other value. To use such transforms without specifying a value, specify
the value as "-" in the transformation object. Otherwise, specify the value that one wants to add to this transformation.


```python
image_url = imagekit.url({
    "src": "https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg",
    "transformation": [{
        "format": "jpg",
        "progressive": "true",
        "effect_sharpen": "-",
        "effect_contrast": "1"
    }]
})
```

The result is in a URL like
```python
# Note that because `src` parameter is in effect, the transformation string gets added as a query parameter `tr`
https://ik.imagekit.io/your_imagekit_id/endpoint/default-image.jpg?tr=f-jpg%2Cpr-true%2Ce-sharpen%2Ce-contrast-1
```

**3. Signed URL that expires in 300 seconds with the default URL endpoint and other query parameters**

```python
image_url = imagekit.url({
    "path": "/default-image",
    "query_parameters": {
        "p1": "123",
        "p2": "345"
    },
    "transformation": [{
        "height": "300",
        "width": "400"
    }],
    "signed": True,
    "expire_seconds": 300
})
```

**Sample Result URL**
```
https://ik.imagekit.io/your_imagekit_id/tr:h-300,w-400/default-image.jpg?v=123&ik-t=1567358667&ik-s=f2c7cdacbe7707b71a83d49cf1c6110e3d701054
```

**List of transformations**

The complete list of transformations supported and their usage in ImageKit is available [here](https://docs.imagekit.io/features/image-transformations/resize-crop-and-other-transformations).
The SDK gives a name to each transformation parameter, making the code simpler, more straightforward, and readable.
If a transformation is supported in ImageKit, though it cannot be found in the table below, then use the
transformation code from ImageKit docs as the name when using the `url` function.

| Supported Transformation Name | Translates to parameter |
| ----------------------------- | ----------------------- |
| height |  h|
| width |  w|
| aspect_ratio |  ar|
| quality |  q|
| crop |  c|
| crop_mode |  cm|
| x |  x|
| y |  y|
| focus |  fo|
| format |  f|
| radius |  r|
| background |  bg|
| border |  b|
| rotation |  rt|
| blur |  bl|
| named |  n|
| overlay_image |  oi|
| overlay_image_aspect_ratio |  oiar|
| overlay_image_background |  oibg|
| overlay_image_border |  oib|
| overlay_image_dpr |  oidpr|
| overlay_image_quality |  oiq|
| overlay_image_cropping |  oic|
| overlay_image_trim |  oit|
| overlay_x |  ox|
| overlay_y |  oy|
| overlay_focus |  ofo|
| overlay_height |  oh|
| overlay_width |  ow|
| overlay_text |  ot|
| overlay_text_font_size |  ots|
| overlay_text_font_family |  otf|
| overlay_text_color |  otc|
| overlay_text_transparency |  oa|
| overlay_alpha |  oa|
| overlay_text_typography |  ott|
| overlay_background |  obg|
| overlay_image_trim |  oit|
| overlay_text_encoded |  ote|
| overlay_text_width |  otw|
| overlay_text_background |  otbg|
| overlay_text_padding |  otp|
| overlay_text_inner_alignment |  otia|
| overlay_radius |  or|
| progressive |  pr|
| lossless |  lo|
| trim |  t|
| metadata |  md|
| color_profile |  cp|
| default_image |  di|
| dpr |  dpr|
| effect_sharpen |  e-sharpen|
| effect_usm |  e-usm|
| effect_contrast |  e-contrast|
| effect_gray |  e-grayscale|
| original |  orig|

## File Upload

The SDK provides a simple interface using the `.upload_file()` method to upload files to the ImageKit Media library. It
accepts all the parameters supported by the [ImageKit Upload API](https://docs.imagekit.io/api-reference/upload-file-api/server-side-file-upload).

The `upload_file()` method requires at least the `file` and the `file_name` parameter to upload a file and returns a Dict with error or success data. Use `options` parameter to pass other parameters supported by the [ImageKit Upload API](https://docs.imagekit.io/api-reference/upload-file-api/server-side-file-upload). Use the same parameter name as specified in the upload API documentation.

Simple usage

```python
imagekit.upload_file(
    file= "<url|base_64|binary>", # required
    file_name= "my_file_name.jpg", # required
    options= {
        "folder" : "/example-folder/",
        "tags": ["sample-tag"],
        "is_private_file": False,
        "use_unique_file_name": True,
        "response_fields": ["is_private_file", "tags"],
    }
)

```

If the upload succeeds, `error` will be `None,` and the `response` is the same as the response received from ImageKit's servers.
If the upload fails, `error` will be the same as the error response from ImageKit's servers, and the `response` will be `None`. Learn more from the sample app in this repository.

## File Management

The SDK provides a simple interface for all the [media APIs mentioned here](https://docs.imagekit.io/api-reference/media-api)
to manage one's files. 

If the request succeeds, `error` will be `None,` and the `response` is the same as the response received from ImageKit's servers.
If the upload fails, `error` will be the same as the error response from ImageKit's servers, and the `response` will be `None`. Learn more from the sample app in this repository.

**1. List & Search Files**

Accepts an object specifying the parameters used to list and search files. 
Pass parameters as specified in the [documentation here](https://docs.imagekit.io/api-reference/media-api/list-and-search-files#list-and-search-file-api) with the correct values to get the results.

```python
imagekit.list_files({
    "skip": 10,
    "limit": 10,
})
```

**2. Get File Details**
Accepts the file ID and fetches the details as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/get-file-details)

```python
imagekit.get_file_details(file_id)
```

**3. Get File Metadata**
Accepts the file ID and fetches the metadata as per the [API documentation here](https://docs.imagekit.io/api-reference/metadata-api/get-image-metadata-for-uploaded-media-files)
```python
imagekit.get_file_metadata(file_id)
``

**3. Get File Metadata from remote url**
Accepts the remote file URL and fetches the metadata as per the [API documentation here](https://docs.imagekit.io/api-reference/metadata-api/get-image-metadata-from-remote-url)

```python
imagekit.get_remote_file_url_metadata(remote_file_url)
```

**4. Update File Details**
Update parameters associated with the file as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/update-file-details).
The first argument to the `update_field_details` method is the file ID, and a second argument is an object with the
parameters to be updated.

```python
imagekit.update_file_details("file_id", {
    "tags": ["image_tag"],
    "custom_coordinates": "10,10,100, 100"
})
```

**5. Delete file**
Delete a file per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/delete-file). The method accepts the file ID of the file that has to be
deleted.

```python
imagekit.delete_file(file_id)
```

**6. Bulk File Delete by IDs**
Delete a file as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/delete-files-bulk). The method accepts a list of file IDs of files.

```python
imagekit.bulk_file_delete(["file_id1", "file_id2"])
```

**7. Purge Cache**
Programmatically issue an explicit, `clear cache` request as per the [API documentation here](https://docs.imagekit.io/api-reference/media-api/purge-cache). It accepts the full URL of the file.

```python
imagekit.purge_file_cache(full_url)
```

**7. Purge Cache Status**

Get the purge cache request status using the request ID returned when a purge cache request gets submitted as pet the
[API documentation here](https://docs.imagekit.io/api-reference/media-api/purge-cache-status)

```python
imagekit.get_purge_file_cache_status(cache_request_id)
```

## Utility functions

We have included the following commonly used utility functions in this package.

**Authentication parameter generation**

Suppose one wants to implement client-side file upload. In that case, one will need a token, expiry timestamp, and a valid signature for that upload. The SDK provides a simple method that one can use in their code to generate these authentication parameters.

<em>Note: Any client-side code should never expose The Private API Key. One must always generate these authentications parameters on the server-side</em>

authentication

`authentication_parameters = imagekit.get_authentication_parameters(token, expire)`

Returns

```python
{
    "token": "unique_token",
    "expire": "valid_expiry_timestamp",
    "signature": "generated_signature"
}
```

Both the `token` and `expire` parameters are optional. If not specified, the SDK uses the UUID to generate a random token and internally generates a valid expiry timestamp. The `token` and `expire` used to generate `signature` is part of a response returned by the server.

**Distance calculation between two pHash values**

Perceptual hashing allows one to construct a hash value that uniquely identifies an input image based on the contents
of an image. [imagekit.io metadata API](https://docs.imagekit.io/api-reference/metadata-api) returns the `pHash`
value of an image in the response. One can use this value to [find a duplicate or similar image](https://docs.imagekit.io/api-reference/metadata-api#using-phash-to-find-similar-or-duplicate-images) by calculating the distance between the two images.


This SDK exposes `phash_distance` function to calculate the distance between two `pHash` value. It accepts two `pHas`h hexadecimal
strings and returns a numeric value indicative of the difference between the two images.

```python
def calculate_distance():
    # fetch metadata of two uploaded image files
    ...
    # extract pHash strings from both: say 'first_hash' and 'second_hash'
    ...
    # calculate the distance between them:

    distance = imagekit.phash_distance(first_hash, second_hash)
    return distance
```

**Distance calculation examples**
```python
imagekit.phash_distance('f06830ca9f1e3e90', 'f06830ca9f1e3e90')
# output: 0 (ame image)

imagekit.phash_distance('2d5ad3936d2e015b', '2d6ed293db36a4fb')
# output: 17 (similar images)

imagekit.phash_distance('a4a65595ac94518b', '7838873e791f8400')
# output: 37 (dissimilar images)
```

### Sample Code Instruction
To run `sample` code, go to the sample directory and run,

```python
python sample.py
```

## Support
For any feedback or to report any issues or general implementation support, please reach out to [support@imagekit.io]()

## Links

* [Documentation](https://docs.imagekit.io/)

* [Main Website](https://imagekit.io/)

## License
Released under the MIT license.
