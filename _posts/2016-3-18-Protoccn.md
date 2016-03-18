---
layout: post
title: Protoccn -- A Python Framework for CCN Applications
---

Most CCN applications I've seen or written so far go something like this:

1. Producers listen for interests with a particular prefix. They invoke
some callback function which generates or obtains the data when they receive
an interest with that particular prefix. The data is then put in the payload
of a content object and sent back to the network.
2. Consumers issue interests to the network for specific prefixes to get
data. Most of these calls are blocking.

This is a fairly common pattern so I decided to write a little framework
to help hide the mundane details that go into structuring these applications.
The result -- [protoccn](https://github.com/chris-wood/protoccn) -- is a Flask-like [1]
framework for creating producer and consumer CCN applications. In this post I'll
outline some basic examples of the framework to build a simple file server.

# Producer Framework

Producers in CCN applications can be differentiated by how they respond to
interests with a specific name. The rest of the mechanics -- listening for
incoming interests, extracting the name and payload information to act on it,
and invoking a particular callback -- are common among many applications.
Consider the following code snippet taken from the PARC standard file transfer
application written in C.

{% gist fee428e4345537aa3a90 %}

The code should be quite readable. The meat of this infinite loop is a call to
the function ```_createInterestResponse```. This function generates a content
object response for the incoming interest.

Even though this loop is simple, it would be nice to hide it from the application
developer. That's what protoccn does. Here's the producer code that provides
this functionality.

{% gist 6d5e56f6b132eb9c6b05 %}

The Producer class has two methods: ```handle``` and ```run```. The ```handle```
method is a decorator that takes a string and some other options. The string is an
interest name prefix. This decorator has the effect of registering the decorated
function as a callback whenever an interest with the specified prefix arrives at
the producer. The ```run``` function implements the infinite loop to listen for
interests and invoke the specified callback. The payload of the content object
response is then set to the output of the callback. Pretty simple.

# Consumer Framework

The goal of the consumer part of the protoccn framework is to allow application
developers to configure functions or methods that can trigger interests
and return their responses. It provides two ways of doing this:

1. A consumer can register a custom function or method which maps to an
interest request. For example, an application can create a method called
```get_data``` that, when invoked, issues an interest for, say, "ccnx:/random/data,"
and returns the response.

2. A consumer can create a function "sink" that points to a data location
and serves to generate the payload of the interests sent to that location. For
example, one could define a method called ```send_data``` to accept a string
parameter. A sink can that refers to the location to where the data is sent
can decorate this method. Upon invoking the ```send_data``` method, the
string parameter is inserted into the interest payload and then sent in
an interest carrying the locator name.

The following gist summarizes the main parts of this simple consumer framework.

{% gist 39decbbd16f4c0a86e5b %}

# A Simple File Server

Now that I've discussed the basics of the ```Producer``` and ```Consumer``` classes,
it's now time to build the simple file server. It will work as follows:

- Consumers will download files by issuing interests for them. The interest name will
contain some routable prefix that identifies the server, a string command to indicate
what should be done with the request, and the remainder of the suffix carries the
parameter of said command, e.g., the file to download. We will only support two commands:
```list``` and ```fetch```.

- Producers will listen for interests and respond to them with the respective content.
A list request will generate a recursive listing of the directory rooted at the
server's "mount point." A fetch request will generate the content of the indicated file.
In both cases, the producer returns the result to the consumer in the payload of a content
object.

Let's start with the ```Producer``` code.

{% gist 3b20bcd42b02c1bad938 %}

The code should be pretty simple to read. The producer registers two methods to
handle the ```fetch``` and ```list``` commands separately. The contents of those
methods are also pretty simple to understand: ```fetch``` reads the specified
file and returns the payload whereas ```list``` generates the contents of the
server directory as a list.

The consumer code is almost just as simple. It's shown below.

{% gist f8c7c93fa48b63b7b3ff %}

Consider the nature of the ```list``` and ```fetch``` commands. The ```list``` command
doesn't take any parameters, so we it's registered as a method in the application
using the ```register``` function. Conversely, the ```fetch``` command does require
an argument. As mentioned above, it inserts the name of the file to
fetch as the suffix of the interest name. Therefore, we use the ```install_sink```
decorator to create a function that, when invoked with a file name, appends
that file name to the suffix of the locator.

A sample run of the consumer is below.

{% gist c0f3395ab9f35ba67713 %}

The ```list``` command shows that there are four files to be downloaded. The
```fetch``` command then retrieves and prints the file contents to stdout.

And that's it. I'll try to write more applications using this framework. But
even if I don't, it was still fun to put together and play with the API.

# References

- [1] [http://flask.pocoo.org/](http://flask.pocoo.org/)
