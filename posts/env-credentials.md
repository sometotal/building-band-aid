---
layout: post
title: Let's Build a Band App with LoopBack&#58; Working with Credentials (part 6 of 8)
date: 2017-08-24T01:00:00+00:00
author: Joe Sepi
permalink: /strongblog/lets-build-a-band-app-loopback-pt6/
categories:
  - How-To
  - LoopBack
---

In this series, we are working through building an application to support the needs of DIY bands. We'll start out solving some basic problems and move into more complex ground, eventually transforming the application into a platform that others can use and build upon.

Accompanying this series is a [corresponding workshop](https://github.com/StrongLoop-Evangelists/workshop-band-app) as well as a [code repository](https://github.com/StrongLoop-Evangelists/band-app).

🖥 To see the code for this step, here is [commit/diff](https://github.com/StrongLoop-Evangelists/band-app/commit/a2c5667cd21ed92f6e5c1f95e3c5e0da43a12dda) in the code repository. This will show you the changes from the last episode to this one.

## Previously on _Let's build a band app!_

In the [previous episode](/strongblog/lets-build-a-band-app-loopback-pt5/), we connected a data-source to our application so that our data can be persisted to the cloud. We can start to interact with our APIs and know that our data will not be lost and we can start to build meaningful interfaces. However, we discovered a potential security issue that we need to deal with, which will do in this episode.

## In this episode

In this episode, we will deal with this very important issue of managing your application's credentials. Having our username and password in our code (and therefore, likely to be committed to a public repository) makes our application vulnerable. Let's look at how we can alleviate this issue.

<!--more-->

<img src="https://strongloop.com/blog-assets/2017/band-app/bandapp6.jpg" alt="Let's make a band app part 6" style="width: 500px"/>

## Our approach

It is good practice to have your environments as consistent as possible. The reason being: when you promote your application from your local development environment to a staging environment and eventually to a production environment, if all of the environments along that path are very similar and consistent, we shouldn't have any surprises or unexpected behavior introduced through our deployments. Therefore, we will use the same Cloudant DB instance for both local and production environments.

*Note: As we get further along in our development, we would create a database just for production data to keep development changes from introducing instability; but for now, we will just use the same data-source.*

### Overview

- Connect our Cloudant DB to our production environment
- Copy the production credentials (from VCAP_SERVICES) to a local file
- Add the local credentials file to our ignore lists (do not commit to public repo)
- Configure our application to use the production credentials in prod and the local credentials during development

## Production

Cloud Foundry apps can pass credentials and other environment variables to the application by way of VCAP_SERVICES. Bluemix provides this functionality which we will integrate it into our application.

The first thing we are going to do is connect our Cloudant data-source to our application in Bluemix. We will then be able to take the "production" settings and use them in our local development environment.

### Connect our DB

Let's go into the dashboard of our application on Bluemix and scroll down to the second section where we will be able to connect new or existing services:

![Bluemix - App dashboard](https://strongloop.com/blog-assets/2017/band-app/app-dashboard.png)

We will click the "Connect existing" button to connect our existing Cloudant DB service. From the resulting page (below) we can choose our data-source and click the Connect button:

![Bluemix - App connect service](https://strongloop.com/blog-assets/2017/band-app/app-connect-cloudant.png)

Upon connecting, we will need to restage our application to make that connection "live" on Bluemix.

*Note: there was an option to connect the Cloudant DB to our app when we created the service, but it seemed like a good idea to wait until now to introduce the concept.*

### Environment Variables

Now that our app and data-source are connected in Bluemix, we can take advantage of VCAP_SERVICES for our hosted code and we will replicate the service locally. First, let's go to the "Runtime" section of our application and then click on the "Environment variables" tab:

![Bluemix - App credentials](https://strongloop.com/blog-assets/2017/band-app/app-credentials.png)

At a minimum, we should see our recently connected CloudantDB credentials here. We could also store other environment specific variables here as well, if we needed them. Let's copy that entire code block with all of our credentials.

## Local Development

Now that production is set up and we've copied the information for our CloudantDB, we will need to store this information locally. Let's create a file called `env.json` and paste our credential data into it.


`env.json` should look something like:

```json
{
  "cloudantNoSQLDB": [
    {
      "credentials": {
        "username": "xxxxxxxxxxxxx-bluemix",
        "password": "xxxxxxxxxxxxx",
        "host": "xxxxxxxxxxxxxx-bluemix.cloudant.com",
        "port": 443,
        "url": "https://xxxxxxxxxxxxxx-bluemix:xxxxxxxx@xxxxxxxxxxxxxxxx-bluemix.cloudant.com"
      },
      "syslog_drain_url": null,
      "label": "cloudantNoSQLDB",
      "provider": null,
      "plan": "Lite",
      "name": "Band App CloudantDB",
      "tags": [
        "data_management",
        "ibm_created",
        "lite",
        "ibm_dedicated_public"
      ]
    }
  ]
}
```

Before we go any further, let's add `env.json` to our `.gitignore` and our `.cfignore`. Open each of those "ignore" files and add "env.json" to each.

Now let's write some logic to do the following:

- Have our production application use VCAP_SERVICES on the server to supply our credentials
- Have our local application use the credentials from the local `env.json` file for development.

We'll create a file called `env.js` (*JS not JSON*) and add the following code to it:

```js
let VCAP;

// PRODUCTION CREDENTIALS
// Check for VCAP_SERVICES on the server
if (process.env.VCAP_SERVICES) {
  VCAP = JSON.parse(process.env.VCAP_SERVICES);
}
// LOCAL DEVELOPMENT
// Check for local credentials; log error if it fails
else {
  try {
    VCAP = require('./env.json');
  } catch (e) {
    console.error(e);
  }
};

// Export our data-sources for convenience
let DATASOURCES = {
  "db": {
    "url": VCAP.cloudantNoSQLDB[0].credentials.url,
    "database": "band-app",
    "name": "db",
    "connector": "cloudant"
  }
};

module.exports = { VCAP, DATASOURCES };
```

As you can see in the comments added to the code above, we do a couple things:

- Check if VCAP_SERVICES is provided which would be in production
- Else we try to read the local `env.json` file which contains the credentials we copied from the server
- Log the error if we cannot read the local credentials
- Export our data-sources for convenience. We do this here because then we can use the same code in both the local and production files and have abstracted the assemblage of the data-sources object to one place rather than two. We'll look at that next
- We export both `VCAP` and `DATASOURCES` so we can use any of the credentials we may have (`VCAP`) or access our data-sources directly (`DATASOURCES`)

### Same code for prod and dev

Now we create two files to load our data-sources, one for local and one for production: `datasources.local.js` and `datasources.production.js`. Each will have the following code:

```javascript
let {DATASOURCES} = require('../env.js');

module.exports = DATASOURCES;
```

Above, we are simply getting our data-sources object from the `env.js` file and it works for both local and production. Huzzah!

### Future tweak

Once `loopback-boot` (one of LoopBack's dependencies) v3 is released, we could just use `datasources.js` and not need two different environment-based files, which are both identical in our case.

We would need to:

- check if `loopback-boot` v3 is released: [check here](https://www.npmjs.com/package/loopback-boot)
- if so, we can install it directly: `npm install loopback-boot@3.0.0 --save`
- eventually LoopBack will expose this functionality to all apps without having to manually update `loopback-boot`

Feel free to tweet at me to check the above and update the workshop if there have been developments: [@joe_sepi](https://twitter.com/joe_sepi)

## What's Next?

Okay, now that we have our secrets hidden, let's get back on track and onto an aspect to our application that I find highly valuable: Continuous Integration & Deployment. In the meantime, we can start to play with a UI interface to our data and APIs. The pieces are there and the data will be persisted. Integrating CI/CD will allow us to commit changes to our application which will trigger a new build so we can quickly test those changes in a real way.


