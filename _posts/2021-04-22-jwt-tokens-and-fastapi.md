---
title: "Blacklist and Refresh Tokens (JWT) and FastAPI"
date: 2021-04-19T12:52:30+01:00
categories:
  - guides
tags:
  - fastapi
  - security
  - oauth
  - JWT
---
This guide is a follow up to the [Use Google Login (OAuth) with FastAPI and JWT](/guides/google-login-with-fastapi-and-jwt/), in the previous guide the added to our `FastAPI` application `JWT` support.

Now we are going to improve this tokens and create a blacklist token to ban tokens in case is necessary or just to invalidate a token after an user logs out.
Also we are going to create refresh tokens to avoid requesting the user credentials after each simple token expires.

{% include figure image_path="/assets/posts/jwt-tokens/header.png" alt="JWT tokens" caption="" %}
{% include toc icon="cog" title="Content" %}

# Requirements
This guide assumes you already have installed in your system `python3.8` (or newer).

All the steps to create the Google Credentials and the `login` and `auth` endpoint are defined in [Use Google Login (OAuth) with FastAPI - Python](/guides/google-login-with-fastapi/).

All the steps to support `JWT` defined in the guide [Use Google Login (OAuth) with FastAPI and JWT](/guides/google-login-with-fastapi-and-jwt).

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

This endpoint is going to set the token as invalid. In order to do this, we are going to add the token a list of blacklisted tokens.

This means that we need to add the token to that list when the user logs out, but also means that we need to check in any other request that the token is NOT in that list for it to be valid.

## Create a new function to return the token on user validation:
Our function `get_current_user_email` returns just the user email, but we are going to need the token to add it to the blacklist database.

So we are going to create a new function `get_current_user_token` that returns the `token` instead of the `email`

Create the `get_current_user_token` function on `apps/jwt.py`:
``` python
async def get_current_user_token(token: str = Depends(oauth2_scheme)):
    _ = get_current_user_email(token)
    return token
```
Note: this works because if the credentials are invalid, the `get_current_user_email` function will raise an `Exception`.

## Create a blacklist tokens database
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

The file needs to be created (if needed) when the application starts, so we are going to add the init_backlist_file to the `./main.py` file:
``` python
from apps.db import init_blacklist_file
if __name__ == '__main__':
    init_blacklist_file()
    uvicorn.run(app, port=7000)

```

## Create the logout endpoint:
We are going to add the endpoint to the `main.py` file, we just need to `Depends` on the newly created `get_current_user_token` function. If the token is valid, we are going to just add it to the blacklist.

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
Add the `Logout` button to the end of the `/token` endpoint, the `JWT` must be sent in this message.

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

Note: we are going to delete from `localStorage` the `JWT` value if the logout function was successful, this is useful in case we want to render another elements when the user is no longer logged in.

## Use blacklist token on the validation:
Now that we support `blacklist`tokens, the `get_current_user_email` function needs to check if the token is NOT in the blacklist token list.

So let's modify the `apps/jwt.py` file and edit the `get_current_user_email` function to check if the token is blacklisted at the beginning:

``` python
from apps.db import is_token_blacklisted
async def get_current_user_email(token: str = Depends(oauth2_scheme)):
    if is_token_blacklisted(token):
        raise CREDENTIALS_EXCEPTION
    ...
```

## Test the blacklist:
Now we can test everything:
- Run the app and go to http://127.0.0.1:7000/ .
- Log in with your Google Credentials.
- Generate the `JWT` (this button store the token on `localStorage`).
- Call the protected API call and it should work.
- Press the Logout button.
  - A new line will be added to the `blacklist_db.txt` file.
  - The `JWT` variable is removed from your `localStorage`.
- Manually add the `JWT` var to your browser. 
  - Look for the token in your `blacklist_db.txt` file. (Ignore the `,`)
  - Press F12 in your browser, go to application and add the `jwt` variable with the value of your token.
- Try to use the protected API call and it should fail.

With all this changes we have the blacklist token functionality complete. So let's improve the application with refresh tokens.

# Refresh Tokens:
WIP


# Related Guides:
The [part 1](/guides/google-login-with-fastapi/) of this tutorial explains how to create a Google Application, and how to integrate the Google OAuth with our `FastAPI` project.

The [part 2](/guides/google-login-with-fastapi-and-jwt) of this tutorial just uses sessions cookies for `OAuth` and to create a `JWT` token. Every other endpoint is going to use `JWT` Bearer token authentication.