---
title: "Using the logging lib on python"
date: 2021-05-16T22:56:30+02:00
categories:
  - tips
tags:
  - python
  - log
last_modified_at: 2021-05-16T22:56:30+02:00
---

In this post we are going to use the `logging lib` to write messages, in **production** we are going to write to a `Rotating File` and in **development** we are going to write to the `console`.

# Create a logger.py file

The first step is to create un logger file, in my case i'm going to create it in the `src` folder:

``` sh
mkdir src
cd src
touch logger.py
```

And let's create a `main.py` in the parent folder to test the **logger**.

``` sh
cd ..
touch main.py
```


# Code the logger

Let's read the **environment** variables to determinate if we are in `development` and if the user wants to use an specific `folder` to store the **logs**.

If we are working on `development` we are going to use a logging handler that writes to the **console** and we want to log everything above `DEBUG` level.

In production we just want to log the messages above `INFO` and directly to a **file**.

In the `src/logger.py` file add this:

``` python
import logging
import os
from logging.handlers import RotatingFileHandler

ENV = os.getenv('ENV', 'DEV')
LOG_FOLDER = os.getenv('LOG_FOLDER', '/tmp')

logger = logging.getLogger()

if ENV == 'DEV':
    # Console logger
    handler = logging.StreamHandler()
    formatter = logging.Formatter('[%(asctime)s][%(levelname)s]: %(message)s', '%H:%M:%S')
    level = logging.DEBUG
else:
    # Rotating file handler
    handler = RotatingFileHandler(f'{LOG_FOLDER}/my_app.log', maxBytes=2000, backupCount=10)
    formatter = logging.Formatter('%(asctime)s,%(msecs)03d;%(levelname)s;%(message)s', '%Y-%m-%d %H:%M:%S')
    level = logging.INFO

logger.setLevel(level)
handler.setFormatter(formatter)
logger.addHandler(handler)
```

In the `main.py`file add this:

```python
import time

from src.logger import logger

if __name__ == '__main__':
    logger.info('Info message')
    time.sleep(0.1)
    logger.debug('Debug message')
```

*NOTE: you can use the logger anywhere just importing `logger` from `src.logger`*

# Test the logger:
Run the `main.py` file without any option and the output should be like this:

``` sh
$ python main.py 
[13:11:51][INFO]: Info message
[13:11:51][DEBUG]: Debug message
```

Run the `main.py` file setting the environment as `production`, the log will be in a `file` and without the `debug` info.

``` sh
$ ENV=PROD python main.py 
$ cat /tmp/my_app.log 
2021-05-16 13:13:20,353;INFO;Info message
```



