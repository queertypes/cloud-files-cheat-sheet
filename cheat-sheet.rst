***********************
Cloud Files Cheat Sheet
***********************

**Table of Contents**

.. contents::
    :local:
    :depth: 3
    :backlinks: none


============
Authenticate
============

.. code-block:: bash

    http POST https://identity.api.rackspacecloud.com/v2.0/tokens auth:='{"passwordCredentials": {"username": "myuser", "password": "mypass"}}' content-type:application/json
    # extract response.access.token.id for auth_token
    # extract response.access.serviceCatalog.X.endpoints[0].publicURL 
    #   for storageURL where X is the entry in the serviceCatalog
    #   pertaining to cloud files

==========
Containers
==========

-------
Listing
-------

**Request**

.. code-block:: bash

    http GET https://$storageURL?format=json x-auth-token:$auth_token


**Response**

.. code-block:: http

    HTTP/1.1 200 OK
    Accept-Ranges: bytes
    Content-Length: 93
    Content-Type: application/json; charset=utf-8
    Date: Mon, 22 Apr 2013 16:18:24 GMT
    X-Account-Bytes-Used: 1096
    X-Account-Container-Count: 2
    X-Account-Meta-Temp-Url-Key: 1cf501017a96ff62b8056da007ebb3f1
    X-Account-Object-Count: 1
    X-Timestamp: 1346948684.64672
    X-Trans-Id: txba808b7831114e77a88cd30c3d7e1ff5

    [
        {
            "bytes": 1096, 
            "count": 1, 
            "name": "Test"
        }, 
        {
            "bytes": 0, 
            "count": 0, 
            "name": "millennium"
        }
    ]


--------
Creating
--------

**Request**

.. code-block:: bash

    http PUT https://$storageURL/$container x-auth-token:$auth_token


**Response**

.. code-block:: http

    HTTP/1.1 201 Created
    Content-Length: 0
    Content-Type: text/html; charset=UTF-8
    Date: Mon, 22 Apr 2013 16:21:42 GMT
    X-Trans-Id: tx5a6a0f97a0334dd0b93933b4ec23dde7

--------
Deleting
--------

**Request**

.. code-block:: bash

    http DELETE https://$storageURL/$container x-auth-token:$auth_token


**Response**

.. code-block:: http

    HTTP/1.1 204 No Content
    Content-Length: 0
    Content-Type: text/html; charset=UTF-8
    Date: Mon, 22 Apr 2013 16:22:20 GMT
    X-Trans-Id: tx289e332d72694c86828c39e45b34f3b6

=======
Objects
=======

-------
Listing
-------

**Request**

.. code-block:: bash

    http GET https://$storageURL/$container?format=json x-auth-token:$auth_token


**Response**

.. code-block:: http

    HTTP/1.1 200 OK
    Accept-Ranges: bytes
    Content-Length: 170
    Content-Type: application/json; charset=utf-8
    Date: Mon, 22 Apr 2013 16:32:01 GMT
    X-Container-Bytes-Used: 1096
    X-Container-Object-Count: 1
    X-Timestamp: 1347036637.53485
    X-Trans-Id: tx5300c0bc908e4e21a911b35e9b54d757

    [
        {
            "bytes": 1096, 
            "content_type": "application/octet-stream", 
            "hash": "e4950045ea277a5428c4c9ede564891e", 
            "last_modified": "2012-09-07T16:50:59.222800", 
            "name": "README"
        }
    ]


--------
Fetching
--------

RFC 2616 conditional fetching is supported. Take note of headers: If-Match, If-None-Match, If-Modified-Since, If-Unmodified-Since. Range fetch is also supported.

**Request**

.. code-block:: bash

    http GET https://$storageURL/$container/$object x-auth-token:$auth_token


**Response**

.. code-block:: http

    HTTP/1.1 200 OK
    Accept-Ranges: bytes
    Content-Length: 1096
    Content-Type: application/octet-stream
    Date: Mon, 22 Apr 2013 16:35:53 GMT
    Etag: e4950045ea277a5428c4c9ede564891e
    Last-Modified: Fri, 07 Sep 2012 16:50:59 GMT
    X-Timestamp: 1347036659.22280
    X-Trans-Id: tx9cd061cf3f17421bb03a9002842278ab
    
    This is the source code repository for the Go programming language.
    [....]


--------
Creating
--------

You can ensure end-to-end data integrity by including an MD5 checksum in the object's ETag header.

Automatic deletion can be programmed in by setting the X-Delete-At or X-Delete-After headers. X-Delete-At takes a UNIX timestamp. X-Delete-After takes an integer representing a number of seconds.

**Request**

.. code-block:: bash

    http put $STORAGE/Test/tacos x-auth-token:$AUTH_TOKEN content-length:64 x-delete-after:10 < tacos


**Response**

.. code-block:: http

    HTTP/1.1 201 Created
    Content-Length: 0
    Content-Type: text/html; charset=UTF-8
    Date: Mon, 22 Apr 2013 16:42:29 GMT
    Etag: 68f55613d95a0316a994abc41edda7c9
    Last-Modified: Mon, 22 Apr 2013 16:42:28 GMT
    X-Trans-Id: tx468010bbbf434131afe79ab95bdfd5ec


----------------------
Large Objects Creation
----------------------

For objects larger than 5GB, do as follows:

1. Put the object segments in the same container.
2. Make sure they share a common prefix, e.g., obj-1, obj-2
3. Put the object manifest in that container with metadata X-Object-Manifest: {container}/obj

Now you can either fetch the segments individually or fetch the entire object by fetching the manifest.

----------------
Chunked Transfer
----------------

1. Set HTTP header Transfer-Encoding: chunked
2. Do not specify a Content-Length header.
3. You are responsible for pre-splitting if the stream would be larger than 5 GB.

--------------------
Enabling Compression
--------------------

1. Compress your data.
2. Set the header Content-Encoding: gzip when you create the object.

Cloud Files does not compress your data. This serves as metadata for future requests.

----------------
Server-side Copy
----------------

**Request**

.. code-block:: bash

    http put $STORAGE/Test/tacos-copy x-auth-token:$AUTH_TOKEN content-length:64 x-copy-from:/Test/tacos


**Response**

.. code-block:: http

    HTTP/1.1 201 Created
    Content-Length: 0
    Content-Type: text/html; charset=UTF-8
    Date: Mon, 22 Apr 2013 17:00:48 GMT
    Etag: 68f55613d95a0316a994abc41edda7c9
    Last-Modified: Mon, 22 Apr 2013 17:00:47 GMT
    X-Copied-From: Test/tacos
    X-Copied-From-Last-Modified: Mon, 22 Apr 2013 17:00:32 GMT
    X-Trans-Id: tx61fd4975147d4a29a5ce9dfd14e12b66


