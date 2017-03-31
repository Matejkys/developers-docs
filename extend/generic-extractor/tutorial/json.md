---
title: JSON Introduction
permalink: /extend/generic-extractor/tutorial/json/
---

* TOC
{:toc}

[JSON (JavaScript Object Notation)](http://www.json.org/) is an easy-to-work-with format for describing structured 
data. Before you start working with JSON, familiarize yourself with basic programming jargon.

## Object Representation
To describe structured data, JSON uses **objects** and **arrays**. 

### Objects
Objects consist of **properties** and their **values**. Because the values in an object are identified by names (property names), they are not kept in a particular order. 

The following object describes *John Doe* using two properties : `firstName` and `lastName`.

{% highlight json %}
{
    "firstName": "John",
    "lastName": "Doe"
}
{% endhighlight %}

*Notice that the object is enclosed in `{}`. The properties and values are both in double quotes and are separated 
by a colon. The individual properties are separated from each other using commas.*

### Arrays
As objects collect named values, **arrays** are collections/simple lists of values that do not have a property 
name. They are identified by their numeric position that cannot change. The order of array items stays the same.   

Let's go on to describing John Doe's family using an **array** (marked by `[]`) of three **objects**:

{% highlight json %}
[
    {
        "firstName": "John",
        "lastName": "Doe",
        "role": "father"
    },
    {
        "firstName": "Jenny",
        "lastName": "Doe",
        "role": "mother"
    },
    {
        "firstName": "Jimmy",
        "lastName": "Doe",
        "role": "son"
    }    
]
{% endhighlight %}

*Objects are also separated from each other by commas. Notice that the last item (property or object) is not 
followed by a comma.*

### Terminology
The terminology varies a lot and other expressions are also commonly used: 

- Object --- also a record / structure / dictionary / hash table / keyed list / key value / associative array
- Property --- also a field / key / index
- Array --- also a collection / list / vector / ordinal array / sequence

## Data Values
Each property value always has one of the following data types:

- String --- text
- Number --- number
- Integer --- whole number (without decimal part)
- Boolean --- value which is either `true` or `false`
- Array --- collection of values
- Object --- collection of named values

The types `string`, `number`, `integer` and `boolean` represent **scalar values**. The types `array` and `object` 
represent **structured values** (they are composed of other values). For example:

{% highlight json %}
{
    "stringProperty": "someText",
    "numberProperty": 12.45,
    "integerProperty": 42,
    "booleanProperty": false,
    "arrayProperty": ["first", "second"],
    "objectProperty": {
        "name": "John",
        "surname": "Doe"
    }
}
{% endhighlight %}

*Notice that the boolean value is `false` without quotes. `false`, `true` and `null` (no or an 
unknown value) are **keywords** which must be written without quotes. Otherwise, they would become strings.*

## References
There are multiple ways to refer to particular properties in a JSON document (for instance, [JSONPath](http://jsonpath.com/). 
For the purpose of this documentation, we will use simple *dot notation*. Let's consider this JSON describing the
Doe's family:

{% highlight json %}
{
    "address": {
        "city": "Fresno",
        "street": "Main Street"        
    },
    "members": [
        {
            "firstName": "John",
            "age": 42,
            "shoeSize": 42.5,
            "lastName": "Doe",
            "interests": ["cars", "girls", "lego"],
            "adult": true
        },
        {
            "firstName": "Jenny",
            "adult": true,
            "shoeSize": "24.5",
            "lastName": "Doe",
            "age": 42,
            "interests": ["cars", "boys", "painting"]
        },
        {
            "adult": false,
            "firstName": "Jimmy",
            "lastName": "Doe",
            "shoeSize": null,
            "age": 1,
            "interests": ["cars", "lego", "painting"]
        }
    ]
}
{% endhighlight %}

To refer to John's city, we would write `address.city`. To refer to little Jimmy's shoe size, we
would write `members[2].shoeSize`. Array items indexes are *zero-based*, so the third item has 
index `2`. 

The order of items in an object is not important. It is also worth noting that `[]` represents an empty array and 
`{}` represents an empty object.

## Summary
This page contains a little introduction to JSON documents. We intentionally avoided many details, 
but you should now understand what JSON is, and how to write some stuff in it.
