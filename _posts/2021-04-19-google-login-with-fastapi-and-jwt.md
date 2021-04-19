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

From that code we need to `/login` and the `/auth` endpoints. Let's rename `/auth` to `/validate_token` so it's not confusing with our sub-application base route.

## Modify your Google Cloud domains:
We need to modify the authorized domains because now our endpoint has a `/auth` before the route.
- Access to the Google Cloud Console with your Google account: [GoogleCloud](https://console.cloud.google.com/apis/dashboard)
- Go to Credentials -> OAuth client ID -> Click on edit your application
- Change `http://127.0.0.1:7000/auth` to `http://127.0.0.1:7000/token`

NOTE: this `/token` url is the redirect_uri, it's a frontend route. We are going to redirect after entering the google credentials to the frontend, and then pass it to the server to validate the response. The frontend can be hosted on any domain, we just need to change the url in this section and make the host available on a `CORSMiddleware`.


## Move the run.py code to app/auth.py:
We are going to move the `auth` route to the newly created `validateToken` route, this endpoint will validate the token sent by google and create and send a `JWT` Token to the frontend.
We are going to set the `redirect_uri` to our frontend, so it can have the date to later request a `JWT` token to the server.

So the flow will be:
- Enter the `auth_app/login` endpoint.
- It will take us to the Google Credentials pages.
- It will redirect us to our `frontend`.
- The frontend will request the server to generate a valid `JWT` after validating the google credentials response.


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

# Frontend URL:
FRONTEND_URL = os.environ.get('FRONTEND_URL') or 'http://127.0.0.1:7000/token'

@auth_app.route('/login')
async def login(request: Request):
    redirect_uri = FRONTEND_URL  # This creates the url for our /auth endpoint
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
    jwt = f'valid-jwt-token-for-{user_data["email"]}'
    # TODO: return the JWT token to the user so it can make requests to our /api endpoint
    return JSONResponse({'result': True, 'access_token': jwt})

```

## Create a frontend to test the authentication:
We are going to write the front end in our main app.


The route `/` will only have a `Log In` button, to call the `/auth/login` endpoint. 
The route `/token` will have a button to request the server to generate a `JWT` token with the google response. (This process ideally will be automatically, but let's make a button to see it works).

File: `main.py`
``` python
import uvicorn
from fastapi import FastAPI
from fastapi import Request
from fastapi.responses import HTMLResponse

from apps.api import api_app
from apps.auth import auth_app

app = FastAPI()
app.mount('/auth', auth_app)
app.mount('/api', api_app)


@app.get('/')
async def root():
    return HTMLResponse('<body><a href="/auth/login">Log In</button></body>')


@app.get('/token')
async def token(request: Request):
    return HTMLResponse('''
                <script>
                function send(){
                    var req = new XMLHttpRequest();
                    req.onreadystatechange = function() {
                        if (req.readyState === 4) {
                            console.log(req.response);
                            if (req.response["result"] === true) {
                                window.localStorage.setItem('jwt', req.response["access_token"]);
                            }
                        }
                    }
                    req.withCredentials = true;
                    req.responseType = 'json';
                    req.open("get", "/auth/token?"+window.location.search.substr(1), true);
                    req.send("");

                }
                </script>
                <button onClick="send()">Get FastAPI JTW Token</button>
            ''')



if __name__ == '__main__':
    uvicorn.run(app, port=7000)
```

After running the application we can check that everything is working as intended, to check if it's really validating the token, we can just change a letter on your browser's url bar when the `/token` page is shown and then press generate `JWT Token`. It will return an error 401.

# JWT:
We are going the create a `jwt.py` file to code all the functions related to the token.

## Install PyJWT lib:

We are going to add to the `virtualenv` the `pyjwt` lib, so we can safely create and decode tokens.
``` sh
# With the virtualenv already activated
pip install pyjwt
```

## Create and decode tokens:

We are going to need a `secret key` for our tokens, we can create one using the same method for the secret key using in the `Starlette`'s Middleware in de previous guide.
To avoid creating a database just for this example, we are going to use a dictionary that only has one registered user.

`Create_Token` will be the function that encodes the token and `get_current_user_email` will receive a token and returns us the email in case the token is valid.

Let's create the file `apps/jtw.py`:
``` python
import os
from datetime import datetime
from datetime import timedelta

import jwt
from fastapi import Depends
from fastapi import HTTPException
from fastapi import status
from fastapi.security import OAuth2PasswordBearer

# Create a fake db:
FAKE_DB = {'guillermo.paoletti@gmail.com': {'name': 'Guillermo Paoletti'}}


# Helper to read numbers using var envs
def cast_to_number(id):
    temp = os.environ.get(id)
    if temp is not None:
        try:
            return float(temp)
        except ValueError:
            return None
    return None


# Configuration
API_SECRET_KEY = os.environ.get('API_SECRET_KEY') or None
if API_SECRET_KEY is None:
    raise BaseException('Missing API_SECRET_KEY env var.')
API_ALGORITHM = os.environ.get('API_ALGORITHM') or 'HS256'
API_ACCESS_TOKEN_EXPIRE_MINUTES = cast_to_number('API_ACCESS_TOKEN_EXPIRE_MINUTES') or 15

# Token url (We should later create a token url that accepts just a user and a password to use swagger)
oauth2_scheme = OAuth2PasswordBearer(tokenUrl='/auth/token')

# Error
credentials_exception = HTTPException(
    status_code=status.HTTP_401_UNAUTHORIZED,
    detail='Could not validate credentials',
    headers={'WWW-Authenticate': 'Bearer'},
)


# Create token internal function
def create_access_token(*, data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({'exp': expire})
    encoded_jwt = jwt.encode(to_encode, API_SECRET_KEY, algorithm=API_ALGORITHM)
    return encoded_jwt


# Create token for an email
def create_token(email):
    access_token_expires = timedelta(minutes=API_ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(data={'sub': email}, expires_delta=access_token_expires)
    return access_token


def valid_email_from_db(email):
    return email in FAKE_DB


async def get_current_user_email(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, API_SECRET_KEY, algorithms=[API_ALGORITHM])
        email: str = payload.get('sub')
        if email is None:
            raise credentials_exception
    except jwt.PyJWTError:
        raise credentials_exception

    if valid_email_from_db(email):
        return email

    raise credentials_exception

```

Make sure you add your `API_SECRET_KEY` to your environment so your application can run withour errors.

## Return the JWT on auth/token endoing
Lets import the necessaries functions and the exception to rewrite the `/token` route:
Add to the file `apps/auth.py`:

``` python
from apps.jwt import create_token
from apps.jwt import valid_email_from_db
from apps.jwt import CREDENTIALS_EXCEPTION
```

And let's replace the `/token` route in `apps/auth.py`:
``` python
@auth_app.route('/token')
async def auth(request: Request):
    try:
        access_token = await oauth.google.authorize_access_token(request)
    except OAuthError:
        raise CREDENTIALS_EXCEPTION
    user_data = await oauth.google.parse_id_token(request, access_token)
    if valid_email_from_db(user_data['email']):
        return JSONResponse({'result': True, 'access_token': create_token(user_data['email'])})
    raise CREDENTIALS_EXCEPTION

```

If we test the application, the `Get FastAPI JWT Token` button will print on the browser console something similar to this:
``` json
{"result":true,"access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJndWlsbGVybW8ucGFvbGV0dGlAZ21haWwuY29tIiwiZXhwIjoxNjE4ODczMjgyfQ.9dbl4ylFp2BWLNAVuOwixm0IrV2lr3t7xl2YKOHgC90"}
```

## Make the api endpoints protected
Let's import the `JTW` function that we need to the file `apps/api.py`:

```python
from apps.jwt import get_current_user_email
```

Let's create two routes, one that requires the `JWT` Token and another that doesn't need it.
``` python
from fastapi import Depends

@api_app.get('/')
def test():
    return {'message': 'unprotected api_app endpoint'}


@api_app.get('/protected')
def test2(current_email: str = Depends(get_current_user_email)):
    return {'message': 'protected api_app endpoint'}

```
## Test the new routes:
Using the `terminal` we can check that they are working as intended:
``` sh
$ curl 127.0.0.1:7000/api/
{"message":"unprotected api_app endpoint"}
$ curl 127.0.0.1:7000/api/protected
{"detail":"Not authenticated"}
```

Let's call the protected endpoint using the `JWT` that we have stored in the `localstore` in our frontend.
Update the `main.py` file to add a 3 new buttons to the route `/token` route to test the funcionality:

NOTE: I'm going to use javascript's `fetch` to make the request simpler. Add this lines inside the HTMLResponse of the `/token` route.

``` python
<button onClick='fetch("http://127.0.0.1:7000/api/").then(
    (r)=>r.json()).then((msg)=>{console.log(msg)});'>
Call Unprotected API
</button>
<button onClick='fetch("http://127.0.0.1:7000/api/protected").then(
    (r)=>r.json()).then((msg)=>{console.log(msg)});'>
Call Protected API without JWT
</button>
<button onClick='fetch("http://127.0.0.1:7000/api/protected",{
    headers:{
        "Authorization": "Bearer " + window.localStorage.getItem("jwt")
    },
}).then((r)=>r.json()).then((msg)=>{console.log(msg)});'>
Call Protected API wit JWT
</button>
```

We can check that everything is working fine looking at the browser console:
``` 
# After generating the JWT token
{result: true, access_token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJnd…zM2fQ.oTOHkvqvIYfAUwAxyg6w7z5IGAMjC2jP-kAoajHF7_4"}
# After calling /api/
{message: "unprotected api_app endpoint"}
# After calling /api/protected without JWT
GET http://127.0.0.1:7000/api/protected 401 (Unauthorized)
{detail: "Not authenticated"}
# After calling /api/protected with JWT
{message: "protected api_app endpoint"}
```

# Link to the code:
This app is uploaded to github, you can view the repository using this [link](https://github.com/hanchon-live/tutorial-fastapi-oauth/tree/guide-2), this tutorial is the branch `guide-2`


# Next Steps, improve tokens:
I'm going to make a third and last part of this guide with improvements to the tokens.
- Blacklist tokens
- Refresh tokens to avoid requesting the user and password every 15min
- Security tokens, ask for user and password for important actions in case the JWT was stolen.