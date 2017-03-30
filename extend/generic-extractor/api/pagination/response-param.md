---
title: Response Parameter Scroller
permalink: /extend/generic-extractor/api/pagination/response-param/
---

* TOC
{:toc}

The Response Parameter Scroller can be used with an API which provides some kind
of value in the response which must be used in the next request.

{% highlight json %}
{
    "api": {
        "pagination": {
            "method": "response.url",
            "urlKey": "links.next"
        },
        ...
    },
    ...
}
{% endhighlight %}

## Configuration Parameters
The following configuration parameters are supported for the `response.param` method of pagination:

- `responseParam` (required, string) -- A path to key which contains a value used for scrolling
- `queryParam` (required, string) -- Name of the [query string](/extend/generic-extractor/tutorial/rest/#url) parameter in which the above value should be sent to the API.
- `includeParams` (optional, boolean) -- When true, the [job parameters](/extend/generic-extractor/jobs/#request-parameters) are added to the provided URL. The `queryParam` overrides values from the job parameters (see [example](#overriding-parameters). Default value is `false`.
- `scrollRequest` (optional, object) -- A [job-like](/extend/generic-extractor/jobs/) (supported fields are `endpoint`, `method`, `params`) object which allows to sent an initial scrolling request (see [example](#using-scroll-request).

### Stopping Condition
The pagination ends when the value of `responseParam` parameters is empty -- the key is not present at all, is null,
is an empty string or is `false`. Take care when configuring the `responseParam` parameter. If you e.g. misspell the
name of the key, the extraction will not go beyond the first page. 
[Common stopping conditions](/extend/generic-extractor/api/pagination/#stopping-strategy) also apply.

## Examples

### Basic Configuration
Assume you have an API which returns e.g. the next page number inside the response:

{% highlight json %}
{
    "items": [
        {
            "id": 123,
            "name": "John Doe"
        },
        {
            "id": 234,
            "name": "Jane Doe"
        }
    ],
    "scrolling": {
        "next_page": 2
    }
}
{% endhighlight %}

The following configuration can handle such situation:

{% highlight json %}
{
    "parameters": {
        "api": {
            "baseUrl": "http://example.com/",
            "pagination": {
                "method": "response.param",
                "responseParam": "scrolling.next_page",
                "queryParam": "page"
            }
        },
        "config": {
            "outputBucket": "mock-server",
            "jobs": [
                {
                    "endpoint": "users",
                    "dataField": "items"
                }
            ]
        }
    }
}
{% endhighlight %}

The first request will be sent to `/users`. For the second request, the value found in the response 
in the property `scrolling.next_page` will be sent as the `page` parameter. Therefore the request 
will be sent to `/users?page=2`.

See [Full Example](todo:057-pagination-response-param-basic)

### Overriding Parameters
The following configuration passes the parameter `orderBy` to every request:

{
    "parameters": {
        "api": {
            "baseUrl": "http://example.com/",
            "pagination": {
                "method": "response.param",
                "responseParam": "scrolling.next_page",
                "includeParams": true,
                "queryParam": "page"
            }
        },
        "config": {
            "debug": true,
            "outputBucket": "mock-server",
            "jobs": [
                {
                    "endpoint": "users",
                    "dataField": "items",
                    "params": {
                        "page": "start",
                        "orderBy": "id"
                    }
                }
            ]
        }
    }
}

The `includeParams` configuration set to true causes the parameters from the `job.params` settings to 
be sent with every request. If you set `includeParams` to false, then they will be sent only with
the first request. Also notice that the `page` parameter from `job.params` is overridden by the 
`page` parameter specified in the `pagination.queryParam`. Therefore the first request will be
sent to `/users?page=start&orderBy=id` and the second request will be sent to `/users?page=2&orderBy=id`.

See [Full Example](todo:058-pagination-response-param-override)

### Using Scroll Request
The response param scroller supports sending of an initial scrolling request. This can be used
in situations where the API requires some kind of special initialization of a scrolling endpoint.
For example the [Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/search-request-scroll.html)
Another example is an API which has something like a search endpoint, which needs a initial request and
then allows you to scroll through the results (this is not exactly [RESTful](todo) though).

Let's consider an API which -- to list users -- requires that you send a POST request to
`/search` endpoint with the configuration:

{% highlight json %}
{
    "object": "users",
    "orderBy": "id"
}
{% endhighlight %}

It will then respond with a *search token* which represents an internal cursor:

{% highlight json %}
{
    "scroll": {
        "token": "b97d814f1a715d939f3f96bc574445de",
        "totalCount": 4
    }
}
{% endhighlight %}

To obtain the actual result, you need to send a request to endpoint `/results` with parameter
`scrollToken=b97d814f1a715d939f3f96bc574445de`. The response looks like this:

{% highlight json %}
{
    "items": [
        {
            "id": 123,
            "name": "John Doe"
        },
        {
            "id": 234,
            "name": "Jane Doe"
        }
    ],
    "scroll": {
        "token": "4015e9ce43edfb0668ddaa973ebc7e87"
    }
}
{% endhighlight %}

The following configuration is able to handle the situation:

{% highlight json %}
{
    "parameters": {
        "api": {
            "baseUrl": "http://example.com/",
            "pagination": {
                "method": "response.param",
                "responseParam": "scroll.token",
                "queryParam": "scrollToken"
                "scrollRequest": {
                    "endpoint": "results",
                    "method": "GET"
                }
            }
        },
        "config": {
            "debug": true,
            "outputBucket": "mock-server",
            "jobs": [
                {
                    "endpoint": "search",
                    "method": "POST",
                    "dataField": "items",
                    "dataType": "users",
                    "params": {
                        "object": "users",
                        "orderBy": "id"
                    }
                }
            ]
        }
    }
}
{% endhighlight %}

The configuration is actually turned upside-down. The `jobs` section defines the initial search request
(`POST` to `/search` with the required parameters `object` and `orderBy`). The first request sent to the API 
is therefore

    POST /search

    {"object":"users","orderBy":"id"}

When the response contains a `scroll.token` field, the scroller starts to act and overrides the above 
configuration with the one provided in `scrollRequest` configuration. The next request is therefore 
a `GET` to `/results?scrollToken=b97d814f1a715d939f3f96bc574445de`. The `queryParam` configuration
causes the `scrollToken` request parameter. This will then repeat until the `scroll.token` field in the response is empty.

See [Full Example](todo:059-pagination-response-param-scroll-request)
