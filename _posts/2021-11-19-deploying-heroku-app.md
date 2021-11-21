---
layout: distill
title: Deploying a containerized Heroku app with an Apple M1 machine
description: the M1 is fast and furious, but bound to cause headaches
date: 2021-11-19

authors:
  - name: A.L.
---

It's common to want to deploy an app on the web. A plethora of services and platforms now make this easy, compared to what it would have taken 10 or 20 years ago.

In this article, I'd like to outline one way to do so. Particularly, building a Python app, using Flask as a back-end server, bundling it all together with Docker, and deploying on Heroku. To add a twist, we're going to do this from a MacBook Pro with an Apple Silicon M1 chip.

## 1. Build your app

We're going to build a simple site that lands the user on a page, allows them fill out a form, and then view their submission.

To route traffic, handle requests, and serve static content, we'll be using Flask. A popular alternative is Django.

We'll start with the HTML.

Then we'll build our server.

## 2. Test locally

To test our web app locally, we can just run,

```
python app.py
```

This launches our Flask app as a local server on a localhost port, specifically port 5000.

## 3. Containerize your app

Now that we see our app working locally, we can containerize the app. By creating a container for our app, Heroku will be faster at deploying (since it won't have to rebuild the entire app every deployment), and it will ensure dependencies and architectures are platform-agnostic.

Or, almost platform-agnostic. I learned the hard way that Docker is particular in certain ways about the host build machine and its architecture. In particular, the architecture of an M1 Mac requires Docker to build apps differently than what Heroku wants to deploy them.

In any case, we need to start with a Dockerfile, so we'll do that.

First, we'll be starting from a Python 3.8 image as the base layer.

We need to install our Python dependencies. This could also be done with a requirements file, but here we just write them out.

We need to copy our app source files into the image.

Flask expects to host the server through an open port, so we'll expose a port just for Flask.

Lastly, the launch command when we run the image as a container is to launch the server.

Now we need to build the image. Typically, we'd see,

```
docker build . -t tag
```

but because we're working on a special architecture, we need,

```
docker buildx build . -t tag --platform linux/amd64
```

`buildx` does __

Now that we have it setup with the right architecture, we can test the image locally by building a container. For example,

```
docker run --rm tag
```

To break down the arguments here, 

Now we can visit the exposed port and see our app live, locally.

## 4. Push to Heroku

First, we need to create a Heroku account. After that's been setup, create an app with any name you want.

We'll be deploying to the Heroku registry manually, but we still will use the Heroku CLI for some parts, so make sure that's installed.

Now, we want to retag our image with the location of our Heroku app registry. For example,

```
docker tag xxx xxx
```

Next we want to push the image to this registry, like,

```
docker push xxx
```

Now our app is pushed up to Heroku, and we just need to tell Heroku to take the image live!

```
heroku container:release web -a xxx
```

To open your app, try `heroku open -a xxx`.

## 5. Share

You should be seeing your site live.

# Troubleshooting

## 1. Heroku H10 error

Notorious!
