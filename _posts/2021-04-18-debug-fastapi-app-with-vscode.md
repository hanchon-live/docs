---
title: "Debug FastAPI with VSCode"
date: 2021-04-18T22:59:30+01:00
categories:
  - tips
tags:
  - fastapi
  - vscode
last_modified_at: 2021-04-18T22:59:30+01:00
---
To debug a `FastAPI` app with `VSCode` we are going to go the debug section and create a `launch.json` file.
{% include figure image_path="/assets/posts/debug-fast-api/debug-location.png" alt="Location" caption="" %}

We are going to use the debug template for FastAPI, `VSCode` will ask for the name of our file, in my case is `run.py` that is located in my root folder.
{% include figure image_path="/assets/posts/debug-fast-api/debug-file-type.png" alt="FastAPI web application" caption="" %}

It will generate a `launch.json` file in the `.vscode` folder
``` json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: FastAPI",
            "type": "python",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "run:app"
            ],
            "jinja": true
        }
    ]
}
```

Make sure that you have your python interpreter set in `VSCode` and the libs `fastapi` and `uvicorn` are installed.


## Setting up environments variables in VSCode
To add env vars to the debugging, we need to create `env` in the configuration file and set our variables there:
``` json
{
    ...
    "configurations": [
        { 
            ...,
            "env": {
                "GOOGLE_CLIENT_ID": "id_from_google",
                "SECRET_KEY": "this_is_my_secret_key"
            }
        }
    ]
}
```

## Configure FastAPI port in VSCode
We may also want to run the FastAPI app in another port (default port is `8000`), we can set up the port in the `args` section:
``` json
{
    ...
    "configurations": [
        { 
            ...,
            "args": {
                "--port",
                "7000",
                "run:app"
            }
        }
    ]
}
```