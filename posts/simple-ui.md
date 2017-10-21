---
layout: post
date: 2017-09-26T01:40:15+00:00
title: Let's Build a Band App with LoopBack - UI and Wrap-Up (part 8 of 8)
author: Joe Sepi
permalink: /strongblog/lets-build-a-band-app-loopback-pt8/
categories:
  - How-To
  - LoopBack
---

In this series, we are working through building an application to support the needs of DIY bands. We'll start out solving some basic problems and move into more complex ground, eventually transforming the application into a platform that others can use and build upon.

Accompanying this series is a [corresponding workshop](https://github.com/StrongLoop-Evangelists/workshop-band-app) as well as a [code repository](https://github.com/StrongLoop-Evangelists/band-app).

## Previously on _Let's build a band app!_

In the [previous episode](/strongblog/lets-build-a-band-app-loopback-pt7/), we setup a toolchain to always build and deploy our code whenever we merge changes to the master branch of our upstream repository. It is a good practice to automate any repetitive tasks and this is a perfect example. We can now focus on continuing to build our band app.

## In this episode

In this episode, we are going to quickly build out another API endpoint, configure our LoopBack server to be more than just an API server so that we can serve our UI as well, and we will put together some simple HTML to show our APIs in action.

<!--more-->

<img src="https://strongloop.com/blog-assets/2017/band-app/bandapp8.jpg" alt="Let's make a band app part 8" style="width: 500px"/>

## Show our band's info

Our goal at the moment is to have a band site driven by our API endpoints. Currently, we have an `events` endpoint, but I would also like my band's information to be stored in the database. So I will create a `bands` endpoint. In thinking through what this endpoint should look like, I realized that it could hold multiple band entries, so I could also use it to collect information for my previous bands and on the site, I could eventually have a page that shows that information. But that is for a later date. For now, we will create the `bands` endpoint and add our current band and use the API to get the info and have the UI display it.

## `bands` endpoint

Just like we've done previously, we jump into the terminal and type `lb model`. Since this is old hat, let's look at the answers to the questions and not step through each prompt. They are as follows:

```
➜  band-app git:(master) ✗ lb model
? Enter the model name: band
? Select the datasource to attach band to: db (memory)
? Select model's base class PersistedModel
? Expose band via the REST API? Yes
? Custom plural form (used to build REST URL):
? Common model or server only? common
Let's add some band properties now.

Enter an empty property name when done.
? Property name: name
   invoke   loopback:property
? Property type: string
? Required? Yes
? Default value[leave blank for none]:

Let's add another band property.
Enter an empty property name when done.
? Property name: image
   invoke   loopback:property
? Property type: string
? Required? No
? Default value[leave blank for none]: ''

Let's add another band property.
Enter an empty property name when done.
? Property name: description
   invoke   loopback:property
? Property type: string
? Required? No
? Default value[leave blank for none]: ''

Let's add another band property.
Enter an empty property name when done.
? Property name: socialMedia
   invoke   loopback:property
? Property type: array
? The type of array items: object
? Required? No
? Default value[leave blank for none]:

Let's add another band property.
Enter an empty property name when done.
? Property name:
```

This will produce a `band.json` file that looks like so:

```
{
  "name": "band",
  "base": "PersistedModel",
  "idInjection": true,
  "options": {
    "validateUpsert": true
  },
  "properties": {
    "name": {
      "type": "string",
      "required": true
    },
    "image": {
      "type": "string",
      "default": "''"
    },
    "description": {
      "type": "string",
      "default": "''"
    },
    "socialMedia": {
      "type": [
        "object"
      ]
    }
  },
  "validations": [],
  "relations": {},
  "acls": [],
  "methods": {}
}
```

We can restart our node app (using `node .` or `npm start`), visit the explorer and add our band's information now!

## Serving static files

LoopBack requires a couple quick configuration changes to serve static files. There is already a client directory, so we need to tell the application server to look there for static files and we also need to disable the default root route.

To tell LoopBack to look in the client directory for static files, we find the `files` key in the `server/middleware.json` file and update it to look like this:

```
  "files": {
    "loopback#static": {
      "params": "$!../client"
    }
  },
```

Then we go into our `server/boot/root.js` file and find the line that handles the default route. We can disable it by simply commenting it out by adding two slashes at the beginning, like so:

```
  // router.get('/', server.loopback.status());
```

Now LoopBack is setup to serve static files from your client directory. Let's give it something to serve!

## Our index page

Navigate to your `client` directory and add a file named `index.html`. In this file, add the following code, which has our basic HTML structure, including two divs to inject our templated content built from APIs, as well as the JavaScript needed to do that fetch and template.

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8"/>
    <title>Band App</title>
  </head>
  <body>
    <div class="js-info"></div>
    <div class="js-events"></div>

    <script>
      // BAND INFO
      fetch('http://0.0.0.0:3000/api/bands/829f5215a7864a332e12db14fbd40a2e')
      .then(res => res.json())
      .then(res => {
        let tmpl = `
          <img src="${res.image}"" style="float:right; margin:50px 0; width:100%;">
          <h1>${res.name}</h1>
          <div>
            ${res.description}
          </div>
        `;

        tmpl += res.socialMedia.reduce((str, channel) => {
          return str + `<p><a href="${channel.url}">${channel.name}</a></p>`;
        }, '');

        document.getElementsByClassName('js-info')[0].innerHTML = tmpl;
      });

      // EVENTS
      fetch('http://0.0.0.0:3000/api/events')
      .then(res => res.json())
      .then(res => {
        tmpl = '<h2>Events</h2><ul>';

        tmpl += res.reduce((str, event) => {
          return str + `<li>${event.description}</li>`;
        }, '');

        tmpl += '</ul>';

        document.getElementsByClassName('js-events')[0].innerHTML = tmpl;
      });
    </script>
  </body>
</html>
```

*Note: we are using the Fetch API which isn't supported in all browsers. I chose this method because I didn't want to introduce any dependencies like jQuery or a Fetch shim. Also of note, I am very crudely templating my data by hand. This doesn't scale well for a full application -- typically we would choose a templating library like Mustache or perhaps even use a full featured client-side framework like Vue.js, Angular or that other one with the licensing issues.* ;)

And voila! We have a working band page that is the beginnings of our band app. We can see this page being served up from the app on Bluemix: https://band-app.mybluemix.net

## What's Next?

This is obviously still a very crude "Band App" but it gets us started in a number of ways:

1. We have a couple APIs to start with and can expand from there to include the band's recorded output (CDs and such), the band's merch, and more.
1. We have our infrastructure in place to build on this foundation: deployment source, data-source, continuous deployment/integration

This is also the seed of my inspiration. From here, we can imagine a number of great possibilities for this application, including:

- Events can have bands, which is now its own model and we can build a relationship between the two
- Having multiple bands means we could invite other users which turns our app into a platform
- Events have venues, which should be their own model. We could make the platform an avenue to connect bands and venues by having promoters join the platform
- The platform could be opened up to anyone to follow their favorite bands and be alerted of upcoming shows and such

I will begin Phase Two after I build out this application/platform further and get users on board. Then I will come back and share more about what I've built and how others can do so too. I'll be sure to document all the pains and the joys, so stay tuned and thanks for following along. Cheers!

