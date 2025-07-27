---
title: "Building a Simple API with Flask, Demo with Ngrok"
description: "Last week, my team lead gave a seminar about Flask. Oh, I found this framework builds APIs quickly and simply, true to the spirit of Python."
date: 2021-01-14T01:03:00+07:00
slug: flask
image: flask.webp
categories: [Technical]
tags: [Python, Flask, Ngrok]
---

# Introduction

Recently, my team has been running a program called "one lesson per week," where each week, a team member gives a seminar on a particular technique or technology. We just finished several weeks on Kotlin. Last week, my team lead gave a seminar about Flask. Oh, I found this framework builds APIs quickly and simply, true to the spirit of Python. I’ve always worked on Front-end and Mobile, never really touched Back-end (well, I did some PHP in college, but I’ve forgotten it all :joy::joy:). So I decided to give it a try and see how it goes.

# Setup

## Basic

For IDE, PyCharm is probably the best, but it’s a bit heavy, so I just used Visual Studio Code with the Python extension, which is more than enough.

First, install Python.

* If you’re on Windows, download it here: [Download Python](https://www.python.org/downloads/).

* On Ubuntu, just run:

```
$ sudo apt-get update
$ sudo apt-get install python3.9
```

## Option

You can set up **Virtual environments** or skip this step. Virtual environments help manage your project’s dependencies. For example, you can install different versions of a library for different projects, or even use different Python versions. Each project will have its own set of Python libraries, isolated from others.

Python 3 uses the `venv` module to create virtual environments. You can run the following commands to create a project folder and a venv folder:

```
$ mkdir myproject
$ cd myproject
$ python3 -m venv venv
```

On Windows:

```
$ py -3 -m venv venv
```

Activate your environment:

```
$ . venv/bin/activate
```

## Flask

Next, install Flask:

```
$ pip install Flask
```

# Let’s Code

## Hello World

Let’s start by coding a simple Flask app. Create a file called `hello.py` (any name is fine, just avoid `flask.py` to prevent conflicts) and write the following:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

A quick explanation:

* `Flask(__name__)` creates an instance of the Flask class.
* `route()` defines the URL for the API.

Run the app right away:

```
$ export FLASK_APP=hello.py
$ flask run
```

Check http://127.0.0.1:5000/ to see if your app is running.

But now, every time you change the code and refresh the browser, you don’t see the changes applied. Why? That’s because you need to enable Debug mode.

```
$ export FLASK_ENV=development
$ flask run
```

Now try editing the code and see your results.

You can also view your results on another device (I wanted to check on my Android phone) by changing the host (set the host address to your laptop/PC’s IP) and connecting both devices to the same Wi-Fi network:

```
$ flask run --host=192.168.xxx.xxx
```

## JSON

JSON is the data format I often use for APIs. Let’s do a quick JSON demo. First, import `jsonify`:

```python
from flask import Flask, jsonify
```

Instead of returning `Hello, World!`, let’s return a JSON response:

```python
@app.route('/')
def hello_world():
    return jsonify([
        {
            "id": 1,
            "title": "First Memory",
            "description": "This is first Memory"
        },
        {
            "id": 2,
            "title": "Second Memory",
            "description": "This is second Memory"
        },
        {
            "id": 3,
            "title": "Third Memory",
            "description": "This is third Memory"
        }
    ])
```

But I noticed the fields are all jumbled up, so let’s add a bit of config:

```python
app.config['JSON_SORT_KEYS'] = False
```

Okay, that looks better!

## Ngrok - Demo Your App Without Deploying

I’m an Android developer, so I wanted to see if Flask could be used as an API for Retrofit in Android. With the above config, my device could receive the API, but when I tried it with Retrofit, it still didn’t work. Not sure if it’s because of HTTP? If anyone knows, please help clarify this part.

So, let me introduce you to [**Ngrok**](https://ngrok.com/), a tool that helps you quickly demo your Flask app without deploying it to a server.

Install ngrok:

```
$ pip install pyngrok
```

Run `$ ngrok --help` to make sure it’s installed successfully.
Modify your code a bit to create a Ngrok Tunnel:

```python
from pyngrok import ngrok
...
url = ngrok.connect(5000).public_url
print('Henzy Tunnel URL:', url)
```

Continue by running `$ flask run` to see the result. Check the printed URL and access it on your Android device. Try using it as the Retrofit URL (remember to use https), and it works pretty well.

# Conclusion

This is just a Quickstart guide. If you’re interested, I’ll write more articles in the future.

Thank you for reading!

References:

* https://flask.palletsprojects.com/en/1.1.x/quickstart/#quickstart
* https://blog.miguelgrinberg.com/post/access-localhost-from-your-phone-or-from-anywhere-in-the-world
