---
title: "Use Google Login (OAuth) with FastAPI and JTW"
date: 2021-04-19T12:52:30+01:00
categories:
  - guides
tags:
  - fastapi
  - security
  - oauth
  - JWT
---
This guide is a follow up to the [Use Google Login (OAuth) with FastAPI - Python](/guides/google-login-with-fastapi/), in the previous guide We allowed the user to login using its Google Credentials via OAuth in our `FastAPI` project. 

{% include figure image_path="/assets/posts/google-login-jtw/header.png" alt="Header image" caption="" %}

In this guide we are going to create a `JWT` when the user is logged in and use the `JWT Bearer token authentication` for the private endpoints.

The `FastAPI` application created in the previous guide uses a `Starlette` middleware to read the session cookies, we are going to limit the use of this middleware to just the `login` y `auth` endpoints. The rest of the application is going to run without middlewares.

{% include toc icon="cog" title="Content" %}


# Requirements
This guide assumes you already have installed in your system `python3.8` (or newer).

All the steps to create the Google Credentials and the `login` and `auth` endpoint are defined in [Use Google Login (OAuth) with FastAPI - Python](/guides/google-login-with-fastapi/).

# Support for endpoints with different middlewares
With `FastAPI` we can not set just an endpoint to have the middleware, it applies to all the routes. There is a way to resolve this is to create two apps and then mount both of them in our main app.

We are going to start developing on top of the last guide.
```sh
git clone -b guide-1 https://github.com/hanchon-live/tutorial-fastapi-oauth.git
cd tutorial-fastapi-oauth
# Create the virtualenv, activate it, and install the requirements
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Let's create two simple apps and mount them in our main file:

Create the empty file: `apps/__init__.py`

Create the file: `apps/auth.py`
``` python
import os

from fastapi import FastAPI
from starlette.middleware.sessions import SessionMiddleware
from starlette.responses import JSONResponse
# Create the auth app
auth_app = FastAPI()

# Set up the middleware to read the request session
SECRET_KEY = os.environ.get('SECRET_KEY') or None
if SECRET_KEY is None:
    raise 'Missing SECRET_KEY'
auth_app.add_middleware(SessionMiddleware, secret_key=SECRET_KEY)


@auth_app.get('/')
def test():
    return JSONResponse({'message': 'auth_app'})

```


Create the file: `apps/api.py`
``` python
from fastapi import FastAPI

api_app = FastAPI()


@api_app.get('/')
def test():
    return {'message': 'api_app'}

```

Create the file `/main.py`
``` python
import uvicorn
from fastapi import FastAPI

from apps.api import api_app
from apps.auth import auth_app

app = FastAPI()
app.mount('/auth', auth_app)
app.mount('/api', api_app)


@app.get('/')
async def root():
    return {'message': 'main_app'}


if __name__ == '__main__':
    uvicorn.run(app, port=7000)

```

We can test it running the `main.py` file. Make sure to set the SECRET_KEY variable on your environment or create an script to run the file.

Example `run.sh`
``` sh
source .venv/bin/activate
# export GOOGLE_CLIENT_ID=...
# export GOOGLE_CLIENT_SECRET=...
export SECRET_KEY=...
python3 main.py
```

Test the endpoints (using a `terminal` or a `browser`), `/` is our main app, `/auth` uses the `Starlette` middleware and `/api` has no midlleware.

```sh
$ curl 127.0.0.1:7000
{"message":"alive"}
$ curl 127.0.0.1:7000/api/
{"message":"api_app"}
$ curl 127.0.0.1:7000/auth/
{"message":"auth_app"}
```

# Move the auth code to the auth app
Now that we have a sub-application for everything google login related, we are going to move the code in the `run.py` file, created in the previous guide to the `app/auth.py` file.

From that code we need to `/login` and the `/auth` endpoints. Let's rename `/auth` to `/token` so it's not confusing with our sub-application base route.

## Modify your Google Cloud domains:
We need to modify the authorized domains because now our endpoint has a `/auth` before the route.
- Access to the Google Cloud Console with your Google account: [GoogleCloud](https://console.cloud.google.com/apis/dashboard)
- Go to Credentials -> OAuth client ID -> Click on edit your application
- Change `http://127.0.0.1:7000/auth` to `http://127.0.0.1:7000/auth/token`


## Move the run.py code to app/auth.py
We are going to change the `auth` route return so we can read the status when the user is successfully logged in or when then credentials are invalid.

After renaming the `auth` route it should result in something like this. 

File: `app/auth.py`
``` python
import os

from authlib.integrations.starlette_client import OAuth
from authlib.integrations.starlette_client import OAuthError
from fastapi import FastAPI
from fastapi import HTTPException
from fastapi import Request
from fastapi import status
from starlette.config import Config
from starlette.middleware.sessions import SessionMiddleware
from starlette.responses import JSONResponse

# Create the auth app
auth_app = FastAPI()

# OAuth settings
GOOGLE_CLIENT_ID = os.environ.get('GOOGLE_CLIENT_ID') or None
GOOGLE_CLIENT_SECRET = os.environ.get('GOOGLE_CLIENT_SECRET') or None
if GOOGLE_CLIENT_ID is None or GOOGLE_CLIENT_SECRET is None:
    raise BaseException('Missing env variables')

# Set up OAuth
config_data = {'GOOGLE_CLIENT_ID': GOOGLE_CLIENT_ID, 'GOOGLE_CLIENT_SECRET': GOOGLE_CLIENT_SECRET}
starlette_config = Config(environ=config_data)
oauth = OAuth(starlette_config)
oauth.register(
    name='google',
    server_metadata_url='https://accounts.google.com/.well-known/openid-configuration',
    client_kwargs={'scope': 'openid email profile'},
)

# Set up the middleware to read the request session
SECRET_KEY = os.environ.get('SECRET_KEY') or None
if SECRET_KEY is None:
    raise 'Missing SECRET_KEY'
auth_app.add_middleware(SessionMiddleware, secret_key=SECRET_KEY)


@auth_app.route('/login')
async def login(request: Request):
    redirect_uri = request.url_for('auth')  # This creates the url for our /auth endpoint
    return await oauth.google.authorize_redirect(request, redirect_uri)


@auth_app.route('/token')
async def auth(request: Request):
    try:
        access_token = await oauth.google.authorize_access_token(request)
    except OAuthError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail='Could not validate credentials',
            headers={'WWW-Authenticate': 'Bearer'},
        )
    user_data = await oauth.google.parse_id_token(request, access_token)
    # TODO: validate email in our database and generate JWT token
    # TODO: return the JWT token to the user so it can make requests to our /api endpoint
    return JSONResponse(dict(user_data))

```

# WIP: Create fake user db
# WIP: Create JWT Token