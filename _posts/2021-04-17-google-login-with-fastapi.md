---
title: "Use Google Login (OAuth) with FastAPI - Python"
date: 2021-04-17T12:59:30-04:00
categories:
  - guides
tags:
  - fastapi
  - security
---
{% include toc icon="cog" title="Content" %}


{% include figure image_path="/assets/posts/google-login/header.png" alt="Header image" caption="" %}

# Introduction
We are going to allow the user to register and generate a JWT token to operate with our API (`FastAPI`) using Google LogIn.

# Requirements
This guide assumes you already have installed in your system `python3.8` (or newer).

## Setting up the environment
We need to install [FastAPI](https://fastapi.tiangolo.com/), so we are going to create a virtual environment, activate it and use `pip` to install the lib.
We are going to use [Uvicorn](https://www.uvicorn.org/) to run our `FastAPI` app.

```
mkdir fastapi-googlelogin
cd fastapi-googlelogin
python3.8 -m venv .venv
source .venv/bin/activate
pip install fastapi
pip install uvicorn
```

&nbsp;
&nbsp;

# Development:
After we have all the libs installed in our virtualenv, we can start to code.

## Create a basic App
The first step is to create a simple FastAPI app with a public and a private endpoint. Let's create a `run.py` file in the root folder:

File: `fastapi-googlelogin/run.py`

``` python
import uvicorn
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def public():
    return {"result": "This is a public endpoint."}

@app.get("/private")
def private():
    return {"result": "This is a private endpoint."}

if __name__ == '__main__':
    uvicorn.run(app, port=7000)
```

We can run the app with `python ./run.py` (Make sure you have the virtualenv activated).

We can check if everything is working entering `http://127.0.0.1:7000/` and `http://127.0.0.1:7000/private` in any explorer, or just using the `terminal`:

``` sh
$ curl 127.0.0.1:7000
{"result":"This is a public endpoint."}

$ curl 127.0.0.1:7000/private
{"result":"This is a private endpoint."}
```

## Create Google Credentials

Screenshots creating the google auth

## Set up OAuth on our project
We are going to use libs al ready created for [Starlette]() because FastAPI is build on top of that.
We need to install [authlib](https://github.com/lepture/authlib) in our virtualenv and we are going to use this lib to handle the Google Login.

<!-- Currently (04-18-2021) the `authlib` doesn't support the last version of httpx, so we are going to install an older release. -->

### Install the dependency

``` sh
# Make sure the virtualenv is activated
# source .venv/bin/activate
pip install authlib
```

<!-- pip install httpx==0.12.0 -->

### Create the OAuth client

We need the `client_id` and the `client_secret`. To avoid pushing our credentials to the server we are going to pass the values using the environment variables `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`. 
To avoid running the code without this variables, we just ask if they are set and if they aren't we just stop the process.

After getting the credentials in our code, we are going to register the `OAuth` client, we are going to just ask google for `openid email profile`.

This code is added to the file: `fastapi-googlelogin/run.py`, after the `app=FastAPI()` line.

``` python
import os
from starlette.config import Config
from authlib.integrations.starlette_client import OAuth

# OAuth settings
GOOGLE_CLIENT_ID = os.environ.get('API_SECRET_KEY') or None
GOOGLE_CLIENT_SECRET = os.environ.get('API_SECRET_KEY') or None
if GOOGLE_CLIENT_ID is None or GOOGLE_CLIENT_SECRET is None:
    raise BaseException("Missing env variables")

# Set up oauth
config_data = {
    "GOOGLE_CLIENT_ID": GOOGLE_CLIENT_ID,
    "GOOGLE_CLIENT_SECRET":GOOGLE_CLIENT_SECRET
}
starlette_config = Config(environ=config_data) 
oauth = OAuth(starlette_config)
oauth.register(
    name='google',
    server_metadata_url='https://accounts.google.com/.well-known/openid-configuration',
    client_kwargs={
        'scope': 'openid email profile'
    }
)
```

# WIP
Work in Progress

