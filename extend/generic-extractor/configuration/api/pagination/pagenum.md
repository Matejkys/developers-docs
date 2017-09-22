---
title: Page Number Scroller
permalink: /extend/generic-extractor/configuration/api/pagination/pagenum/
---

* TOC
{:toc}

The Page Number Scroller handles a pagination strategy in which the API splits the results into pages
of the same size (limit parameter) and navigates through them using the **page offset** parameter. 
If you need to use the item offset, use the [Offset Scroller](/extend/generic-extractor/configuration/api/pagination/offset/).

{% highlight json %}
{
    "api": {
        "pagination": {
            "method": "pagenum",
            "limit": 100,
            "limitParam": "count",
            ...
        },
        ...
    }
}
{% endhighlight %}

## Configuration Parameters
The following configuration parameters are supported for the `pagenum` method of pagination:

- `limit` (optional, integer) --- page size
- `limitParam`(optional, string) --- name of the parameter in which the API expects the page size; the default value is `limit`.
- `pageParam` (optional, string) --- name of the parameter in which the API expects the page number; the default value is `page`.
- `firstPageParams` (optional, boolean) --- when `false`, the first page will be retrieved without the page parameters; the default 
value is `true`.
- `firstPage` (optional, integer) --- index of the first page; the default value is `1`.

### Stopping Condition
The `pagenum` scroller uses similar stopping condition as the [`offset` scroller](/extend/generic-extractor/configuration/api/pagination/offset/#stopping-condition). 
Scrolling is stopped in case of an underflow --- when the result contains **less items than requested** (including zero). However, 
in the `pagenum` scroller, the **`limit` parameter is not required** and has **no default value**. This means that if you omit it, 
the scrolling will stop only if an empty page is encountered.

## Examples
This section contains three API pagination examples where the Page Number Scroller is used.

### Basic Scrolling
The most simple scrolling setup is the following:

{% highlight json %}
"pagination": {
    "method": "pagenum"
}
{% endhighlight %}

The first request is sent with the parameter `page=1`, for example `/users?page=1`.
The next request will have `page=2`, for example `/users?page=2`.

See [example [EX051]](https://github.com/keboola/generic-extractor/tree/master/doc/examples/051-pagination-pagenum-basic).

### Renaming Parameters
The `limitParam` and `pageParam` configuration options allow you to rename the limit and 
offset for the needs of a specific API:

{% highlight json %}
"pagination": {
    "method": "pagenum",
    "limit": 20,
    "limitParam": "count",
    "pageParam": "set"
}
{% endhighlight %}

Here the API expects the parameters `count` and `set`. The first request will be sent with the parameters `count=20` 
and `set=1`; for example, `/users?set=1&count=20`. 

**Important:** Without setting a value for the `limit` option, the `limitParam` will not be sent at all 
(no matter how you name it).

See [example [EX052]](https://github.com/keboola/generic-extractor/tree/master/doc/examples/052-pagination-pagenum-rename).

### Overriding Parameters
It is possible to override the limit parameter of a specific API job. 
This is useful when you want to use different limits for different API endpoints.

In the following configuration, the first request is sent to `/users?count=2` because the 
`limit` parameter was renamed to `count`. Then the default value of `count` was overridden for the 
`users` API endpoint in `jobs.params.count`. 

The `firstPageParams` is set to false, which means that
the page parameter (named `count`) is **not** sent in the first request. The second API 
request is sent to `/users?count=2&set=1`. Because the `firstPage` option is set to `0`, the 
second page index is `1`.

{% highlight json %}
{
    "parameters": {
        "api": {
            "baseUrl": "http://example.com/",
            "pagination": {
                "method": "pagenum",
                "limit": 200,
                "limitParam": "count",
                "pageParam": "set",
                "firstPage": 0,
                "firstPageParams": false
            }
        },
        "config": {
            "jobs": [
                {
                    "endpoint": "users",
                    "dataField": "items",
                    "params": {
                        "count": 2
                    }
                }
            ]
        }
    }
}
{% endhighlight %}

See [example [EX053]](https://github.com/keboola/generic-extractor/tree/master/doc/examples/053-pagination-pagenum-override).
