---
layout: post
Let's Build a Band App with LoopBack: exploring our api (part 3 of many)
title: Let's Build a Band App with LoopBack&#58; Exploring Our API (part 3 of 8)
date: 2017-07-25T08:05:36+00:00
author: Joe Sepi
permalink: /strongblog/lets-build-a-band-app-loopback-pt3/
categories:
  - How-To
  - LoopBack
---

In this series, we are working through building an application to support the needs of DIY bands. We'll start out solving some basic problems and move into more complex ground, eventually transforming the application into a platform that others can use and build upon.

Accompanying this series is a [corresponding workshop](https://github.com/StrongLoop-Evangelists/workshop-band-app) as well as a [code repository](https://github.com/StrongLoop-Evangelists/band-app).

## Previously on _Let's build a band app!_

In the [previous episode](/strongblog/lets-build-a-band-app-loopback-pt2/), we created our first model-driven API endpoint. We used the LoopBack CLI, which asked us a series of questions about our data and then built out our RESTful API endpoints based on the answers we provided. We took a moment to look at the files that were generated, but now we will actually see those endpoints in action.

## In this episode

In this episode, we will learn about the Swagger UI Explorer that comes with LoopBack and allows us to "explore" our APIs. We can see all of the endpoints that LoopBack has created for us as well as interact with them within the browser.
<!--more-->
## Let's get exploring!

We've initialized a LoopBack application and built our first model-driven RESTful API endpoint for `events`. Let's run the application using `node .` and take a look at the API Explorer UI.

### Start our application

```
➜  band-app git:(master) ✗ node .
Web server listening at: http://0.0.0.0:3000
Browse your REST API at http://0.0.0.0:3000/explorer
```

As we can see above, we have started our node.js application -- upon startup, it helpfully tells us what URL our application is using as well as the URL to the API Explorer. Let's click on the Explorer URL and engage with our endpoints.

_Pro tip: most terminal applications will launch a URL if you `cmd + click` on the text._

### API Explorer

![LoopBack API Explorer - List of endpoints](https://strongloop.com/blog-assets/2017/band-app/api-explorer-02.png)

The browser opens up to a page that lists our API endpoints. We'll see one for `event` and one for `User`.

The **User** endpoint is the base class that is provided by the `api server` option we chose when initializing our application. It is the class that we would extend from when making any sort of user-based role; `members`, for example.

_Pro tip: a programming convention is to name Base classes with a capital letter. Reminder: Base classes are the ones that you do not edit directly, but instead extend from._

The **event** endpoint is the one created in the previous step based on our answers to the prompts.

### Our first endpoint: `event`

![LoopBack API Explorer - product endpoint](/https://strongloop.com/blog-assets/2017/band-app/api-explorer-03.png)

This is where the magic of LoopBack shines through for those just getting started. Based on the questions related to our model, we are given this full CRUD RESTful API. Let's take this moment to appreciate LoopBack: we haven't written any code yet. ✨MAGIC✨ (And some super-smart generator actions.)

### A `GET` request

![LoopBack API Explorer - GET request](https://strongloop.com/blog-assets/2017/band-app/api-explorer-04.png)

If we open up the details of the `GET` request panel in our explorer, we will see some details of our model schema, a parameter section (we'll get to later), as well as a button to "Try it out!"

### A `GET` request - Try it out!

Clicking the "Try it out!" button makes a `GET` request to our `event` endpoint:

![LoopBack API Explorer - GET request response](https://strongloop.com/blog-assets/2017/band-app/api-explorer-05.png)

In addition to providing some request examples (Curl and URL), we also see that the `GET` request is successful with a `200` response, but the response array is empty. That's because we don't have any data! We've only described our data, but we haven't actually added any to our data-source. Let's try adding some data using the `POST` method.

*Pro tip: To learn more about HTTP responses, check out this [list of http status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes). One of my personal favorites is `418 - I'm a teapot`.*

### A POST call - add some data

![LoopBack API Explorer - POST method](https://strongloop.com/blog-assets/2017/band-app/api-explorer-06.png)

If we scroll the page down the list of HTTP verbs, we will see a few POST options. The first one should be `POST /events`. Let's open that panel by clicking on it and we should see a similar panel to the GET one we saw above.

### A `POST` request - Try it out!

We can click on the text field with the yellow background in the Parameters section. Doing so will populate the text field (with the white background) with example data. In the image above, I edited the example data like so:

```json
{
  "date": "June 9, 2017 20:00:00",
  "location": "2nd Nature Skatepark",
  "cost": 5,
  "city": "Peekskill",
  "state": "NY",
  "description": "A punk rock show at a punk rock place",
  "age-restriction": "All ages",
  "lineup": [
    "Jukebox Romantics",
    "Least Best Beast",
    "Kittyhawk"
  ]
}

```

There are a few things to notice in our data:

- it is trimmed: everything in this model is optional
- the `id` key/value in the data is removed: if we send an `id`, we may get an error because that field is managed by the data source
- the date is formatted differently than the sample data: LoopBack handles dates rather nicely

After entering this data into the text field, we can click the 'Try it out!' button to `POST` this data to the endpoint.

![LoopBack API Explorer - POST method response](https://strongloop.com/blog-assets/2017/band-app/api-explorer-07.png)

If the request is successful (as it should be), we get a `200` response and the response body includes the data we sent with an actual `id` returned. To verify the successful `POST`, go back to the `GET` request and "Try it out!"

![LoopBack API Explorer - GET method with data](https://strongloop.com/blog-assets/2017/band-app/api-explorer-08.png)

Yay! Again, we get the `200` response and this time, the `GET` request returns an array with the object we had submitted in the previous step. Exciting, right?

*Note: we have not hooked up a datasource yet. If you remember, we chose the "in memory data store" when creating this model. Therefore, every time you restart your node app, you will lose all the data you have created. We will hook up a datasource in an upcoming episode.*

Until next time, if we want to "explore" this a little further, we could try making these requests through our command line with cURL requests or by hitting the endpoints directly in the browser. We may even want to start implementing these requests in some HTML and make AJAX requests to get data. This should keep us busy for a few until next time!

## What's Next?

Now we have a handle on creating our first model within LoopBack and seeing it in action, let's dive a little deeper: in the next episode, we'll break this model down a little further and create a couple "related" models.


