---
title: "Blacklist and Refresh Tokens (JWT) and FastAPI (Part 3)"
date: 2021-04-19T12:52:30+01:00
categories:
  - guides
tags:
  - fastapi
  - security
  - oauth
  - JWT
---
This guide is a follow up to [Use Google Login (OAuth) with FastAPI and JWT](/guides/google-login-with-fastapi-and-jwt/), in the previous guide the added to our `FastAPI` application `JWT` support.

Now we are going to improve this tokens and create a blacklist token to ban tokens in case is necessary or just to invalidate a token after a user logs out.
Also we are going to create refresh tokens to avoid requesting the user credentials after each access token expires.

{% include figure image_path="/assets/posts/jwt-tokens/header.png" alt="JWT tokens" caption="" %}
{% include toc icon="cog" title="Content" %}

# Requirements
This guide assumes you already have installed in your system `python3.8` (or newer).

- Yoy have your the Google Credentials created and the `login` and `auth` endpoint defined. Following the guide: [Use Google Login (OAuth) with FastAPI - Python](/guides/google-login-with-fastapi/).

- You have support for `JWT`, defined in the guide [Use Google Login (OAuth) with FastAPI and JWT](/guides/google-login-with-fastapi-and-jwt).

Note: if you don't want to read the last two guides, you can just clone the `guide-2` branch to follow along.
```sh
git clone -b guide-2 https://github.com/hanchon-live/tutorial-fastapi-oauth.git
cd tutorial-fastapi-oauth
# Create the virtualenv, activate it, and install the requirements
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

# Blacklist tokens
Let's start with a `logout` endpoint to allow the user to leave our application without the fear of anyone else in the same device using its token.

This endpoint is going to set the token as invalid. In order to do this, we are going to add the token to a list of blacklisted tokens.

This change make us also validate, for each request, if the token is not in the list of blacklisted tokens.

## Validate the token and return it:
Our function `get_current_user_email` returns just the user email, but we are going to need the token to add it to the blacklist database (on the `/logout` endpoint).

So we are going to create a new function `get_current_user_token` that returns the `token` instead of the `email`.

Create the `get_current_user_token` function on `apps/jwt.py`:
``` python
async def get_current_user_token(token: str = Depends(oauth2_scheme)):
    _ = get_current_user_email(token)
    return token
```
Note: this works because if the credentials are invalid, the `get_current_user_email` function will raise an `Exception`.

## Create a blacklist tokens database:
To avoid setting up a database for this example, we are just going to just use a text file.

Let's create a new file `apps/db.py` file and add the fake database.
``` python
def init_blacklist_file():
    open('blacklist_db.txt', 'a').close()
    return True


def add_blacklist_token(token):
    with open('blacklist_db.txt', 'a') as file:
        file.write(f'{token},')
    return True


def is_token_blacklisted(token):
    with open('blacklist_db.txt') as file:
        content = file.read()
        array = content[:-1].split(',')
        for value in array:
            if value == token:
                return True

    return False

```

The file should to be created (if needed) when the application starts, so we are going to add the `init_backlist_file` to the `./main.py` file:
``` python
from apps.db import init_blacklist_file
if __name__ == '__main__':
    init_blacklist_file()
    uvicorn.run(app, port=7000)

```

## Create the logout endpoint:
We are going to add the endpoint to the `main.py` file, we just need to call `Depends` on the newly created `get_current_user_token` function. 

If the token is valid, we are going to just add it to the blacklist.

``` python
from fastapi import Depends
from apps.jwt import get_current_user_token
from apps.jwt import CREDENTIALS_EXCEPTION
from apps.db import add_blacklist_token
from fastapi.responses import JSONResponse


@app.get('/logout')
def logout(token: str = Depends(get_current_user_token)):
    if add_blacklist_token(token):
        return JSONResponse({'result': True})
    raise CREDENTIALS_EXCEPTION

```

## Add the Logout button:
Add the `Logout` button to the end of the `/token` endpoint (in the `main.py` file), the `JWT` must be sent in this message.

``` python
    <button onClick='fetch("http://127.0.0.1:7000/logout",{
        headers:{
            "Authorization": "Bearer " + window.localStorage.getItem("jwt")
        },
    }).then((r)=>r.json()).then((msg)=>{
        console.log(msg);
        if (msg["result"] === true) {
            window.localStorage.removeItem("jwt");
        }
        });'>
    Logout
    </button>
```

Note: we are going to delete from `localStorage` the `JWT` value if the logout function was successful, this is useful in case we want to render another element when the user is no longer logged in.

## Use blacklist token list on the validation:
Now that we support `blacklist` tokens, the `get_current_user_email` function needs to check if the token is NOT in the blacklist token list.

So let's modify the `apps/jwt.py` file and edit the `get_current_user_email` function to check if the token is blacklisted:

``` python
from apps.db import is_token_blacklisted
async def get_current_user_email(token: str = Depends(oauth2_scheme)):
    if is_token_blacklisted(token):
        raise CREDENTIALS_EXCEPTION
    ...
```

## Test the blacklisted tokens:
Now we can test everything:
- Run the app and go to [http://127.0.0.1:7000/](http://127.0.0.1:7000/).
- Log in with your Google Credentials.
- Generate the `JWT` (this button stores the token on `localStorage`).
- Call the protected API call and it should work.
- Press the Logout button.
  - A new line will be added to the `blacklist_db.txt` file.
  - The `JWT` variable is removed from your `localStorage`.
- Manually add the `JWT` var to your browser. 
  - Look for the token in your `blacklist_db.txt` file. (Ignore the `,`)
  - Press F12 in your browser, go to `application` and add the `jwt` variable with the value of your token.
- Try to use the protected API call and it should fail.

With all this changes we have the blacklist token functionality complete. So let's improve the application with refresh tokens.

# Refresh Tokens
A refresh token is an special token that will allow the users to create new tokens when the one that they are using expires.

It works the same way as the `access token` that we are creating on the login function, but this token expiration time is a much longer time, for example, 30 days.

In the frontend the user will try to make a request using the `JWT` token that is stored in the browser's `localStorage` and if the server returns a credentials error, it will try to generate a new `JWT` token using the `refresh token` and make the request again.

The refresh token can be sent in the request header or as a post body, we are going to sent it as post body to follow the [OAuth2 documentation](https://tools.ietf.org/html/rfc6749#section-6).

## Create a refresh token:
We are going to add a function to create a token with an expiration time of 30 days.

Add to the file `apps/jwt.py` the `create_refresh_token` function:

``` python
REFRESH_TOKEN_EXPIRE_MINUTES = 60 * 24 * 30


def create_refresh_token(email):
    expires = timedelta(minutes=REFRESH_TOKEN_EXPIRE_MINUTES)
    return create_access_token(data={'sub': email}, expires_delta=expires)
```


Add to the `/auth/token` route the refresh token (`apps/auth.py`):
``` python
from apps.jwt import create_refresh_token

@auth_app.route('/token')
async def auth(request: Request):
...
    return JSONResponse({
        'result': True,
        'access_token': create_token(user_data['email']),
        'refresh_token': create_refresh_token(user_data['email']),
    })
```

Save the `refresh_token` on the browser's `localStorage` when the `/token` endpoint (in the `main.py` file) is called:

``` python
@app.get('/token')
async def token(request: Request):
    return HTMLResponse(''' 
...
window.localStorage.setItem('jwt', req.response["access_token"]);
window.localStorage.setItem('refresh', req.response["refresh_token"]);
...
```

## Refactor the token decoder:
In the `apps/jwt.py` file let's create the function `decode_token`:
``` python
def decode_token(token):
    return jwt.decode(token, API_SECRET_KEY, algorithms=[API_ALGORITHM])
```
And let's call it inside the `get_current_user_email` function:
``` python
# Change this line:
# jwt.decode(token, API_SECRET_KEY, algorithms=[API_ALGORITHM])
# to this:
payload = decode_token(token)
```

Now we have the decode token function ready to be called when we want to validate the refresh token.

## Create the refresh endpoint:
This endpoint will read the POST data, check if it's a refresh token request, validate the token and get its payload.

With this information we will check if the token is expired and if the email is in our database. After this validations, we create a new `access_token` and return it to the user.

If there is an error in any step, we just return a credentials exception.

Let's add the endpoint to the `apps/auth.py` file:
``` python
from apps.jwt import decode_token
from datetime import datetime


@auth_app.post('/refresh')
async def refresh(request: Request):
    try:
        # Only accept post requests
        if request.method == 'POST':
            form = await request.json()
            if form.get('grant_type') == 'refresh_token':
                token = form.get('refresh_token')
                payload = decode_token(token)
                # Check if token is not expired
                if datetime.utcfromtimestamp(payload.get('exp')) > datetime.utcnow():
                    email = payload.get('sub')
                    # Validate email
                    if valid_email_from_db(email):
                        # Create and return token
                        return JSONResponse({'result': True, 'access_token': create_token(email)})

    except Exception:
        raise CREDENTIALS_EXCEPTION
    raise CREDENTIALS_EXCEPTION
```

## Add a button to refresh the JWT:
To test the new functionality let's add a refresh token call on the `/token` endpoint in the file `main.py`

``` python
<button onClick='fetch("http://127.0.0.1:7000/auth/refresh",{
    method: "POST",
    headers:{
        "Authorization": "Bearer " + window.localStorage.getItem("jwt")
    },
    body:JSON.stringify({
        grant_type:\"refresh_token\",
        refresh_token:window.localStorage.getItem(\"refresh\")
        })
}).then((r)=>r.json()).then((msg)=>{
    console.log(msg);
    if (msg["result"] === true) {
        window.localStorage.setItem("jwt", msg["access_token"]);
    }
    });'>
Refresh
</button>
```
This function will send the token to the backend and will store the new `JWT` value on the `localStorage`.

# Link to the code
This app is uploaded to github, you can view the repository using this [link](https://github.com/hanchon-live/tutorial-fastapi-oauth/tree/guide-3), this tutorial is the branch `guide-3`

# End Notes
This tutorial will end here, but I'm going to make another tutorial to use this API with a frontend written with `React`.

This frontend is going to automatically call the redirect from the Google OAuth and it'll also automatically refresh the token after the `access_tokens` expires.

# Related Guides
The [part 1](/guides/google-login-with-fastapi/) of this tutorial explains how to create a Google Application, and how to integrate the Google OAuth with our `FastAPI` project.

The [part 2](/guides/google-login-with-fastapi-and-jwt) of this tutorial just uses sessions cookies for `OAuth` and to create a `JWT` token. Every other endpoint is going to use `JWT` Bearer token authentication.