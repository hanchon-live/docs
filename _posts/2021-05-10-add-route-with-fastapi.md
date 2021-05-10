---
title: "Add route to FastAPI (GetGloby example)"
date: 2021-05-10T08:10:30+01:00
categories:
  - tips
tags:
  - vscode
  - python
  - fastapi
last_modified_at: 2021-05-10T17:40:30+01:00
---

This is a basic example on how to add new routes to the **GetGloby** API:

{% include toc icon="cog" title="Content" %}

# Create the model
Make a folder inside `models` to organize your files, in this case we are going to add the **translation** settings.

``` sh
mkdir -p src/models/translation/settings
```

*NOTE: add an `__init__.py` file in each folder so python can import them.*

Let's create the `models.py`, `schema.py` and `__init__.py` files inside the `settings` folder to start coding.

``` sh
touch models.py schema.py __init__.py
```

## Models.py:
In this file we are going to define just the database model:
``` python 
from sqlalchemy import Boolean
from sqlalchemy import Column
from sqlalchemy import ForeignKey
from sqlalchemy import Integer

from src.database import Base


class UserTranslationSettings(Base):
    __tablename__ = 'user_translation_settings'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('user.id', ondelete='CASCADE'), nullable=False)
    title_case = Column(Boolean, nullable=False, default=True, server_default='true')
    punctuation_marks = Column(Boolean, nullable=False, default=True, server_default='true')
    replace_synonyms = Column(Boolean, nullable=False, default=True, server_default='true')
    shorten_sentences = Column(Boolean, nullable=False, default=True, server_default='true')
    use_known_translations = Column(Boolean, nullable=False, default=True, server_default='true')
    use_brand_terms = Column(Boolean, nullable=False, default=True, server_default='true')
    force_ad_lengths = Column(Boolean, nullable=False, default=False, server_default='false')

```

Let's import this model to the `models/__init__.py` so **alembic** can read it and create the migration.
``` python
...
from src.models.translation.settings import models  # noqa[F401]
...
```

- Create the revision:
``` sh
alembic revision --autogenerate
```

- Update your local database:
``` sh
alembic upgrade head
```

# Getter
## Create the schema for the API:

Edit the `schema.py` file and create the `requests` and `responses` that we want to use in the new endpoint.

*NOTE: using the `orm_mode = True` will get the data from the database when the object is created instead of requesting the data after accessing each attribute.*

*This change makes possible to just return the object in the endpoint and it will have all the information.*


``` python
from typing import Optional

from pydantic import BaseModel


class Settings(BaseModel):
    title_case: bool
    punctuation_marks: bool
    replace_synonyms: bool
    shorten_sentences: bool
    use_known_translations: bool
    use_brand_terms: bool
    force_ad_lengths: bool

    class Config:
        orm_mode = True


class GetSettingsResponse(BaseModel):
    result: bool
    settings: Optional[Settings]

```

## Create the database queries:

Let's create the `translation` folder inside `src/database`.

``` sh
mkdir -p src/database/translation/settings
touch src/database/translation/__init__.py
touch src/database/translation/settings/__init__.py
```

We are going to code our database functions inside the `__init__.py` file:

``` python
from typing import Optional

from sqlalchemy.orm import Session

from src.models.translation.settings.models import UserTranslationSettings
from src.models.translation.settings.schema import Settings


def get_translation_settings(db: Session, user_id: int) -> Optional[Settings]:
    res = db.query(UserTranslationSettings).filter_by(user_id=user_id).first()
    return res

```

## Create the endpoint:
Let's create the endpoint in the `endpoint` folder:

``` sh
mkdir -p src/endpoints/translation/settings
touch src/endpoints/translation/__init__.py
touch src/endpoints/translation/settings/__init__.py
```

Code the endpoint inside the `settings/__init__.py` file:

``` python
from fastapi import APIRouter
from fastapi import Depends
from sqlalchemy.orm import Session

from src.auth.tokens.validators import get_current_user
from src.database import session_for_request
from src.database.translations import settings
from src.endpoints.constants import TRANSLATION_TAG
from src.models.translation.settings.schema import GetSettingsResponse

router = APIRouter()


@router.get('/get_translation_settings', tags=[TRANSLATION_TAG], response_model=GetSettingsResponse)
async def get_translation_settings(
        user_id: str = Depends(get_current_user),
        db: Session = Depends(session_for_request),
):
    r = settings.get_translation_settings(db, user_id)
    if r:
        return {'result': True, 'settings': r}
    return {'result': False}

```

*NOTE: You may need to create the `TAG` in the `src/endpoints/constants.py` file, this is helpful to organize your endpoints when using `Swagger`*

## Add the new routes to the API:
In `src/endpoints/router_helper.py` add the new router:

``` python
from fastapi import FastAPI

from src.endpoints.translation.settings import router as settings_router


def add_routes(app: FastAPI):
    # Translation Routes
    app.include_router(settings_router)

```

*NOTE: make sure you are calling this function on your `main.py` file*

# Setter
## Validate attributes using the model:
We are going to add a `function` to update the `attribute` of our `model` after validating that the key exists.

This functions allow us to just send the request from the `frontend` directly to the `model`, and it will **validate** it and **execute** it.

Add to the `models/translation/settings/models.py` file the function `update_a_setting`:
``` python
class UserTranslationSettings(Base):
    __tablename__ = 'user_translation_settings'
    id = Column(Integer, primary_key=True)
    ...

    def update_a_setting(self, key, value):
        if hasattr(self, key):
            setattr(self, key, value)
            return True
        return False
```

## Create a schema for the incoming message:
Let's add a class to validate the parameters that the `frontend` is going to send us.

In this case our endpoint needs an `identifier` and a `value`.

Add to the `models/translation/settings/schema.py` file the class `Setting`:
``` python
class Setting(BaseModel):
    identifier: str
    value: bool
```

## Create the database function to update the setting:
Let's create a function that gets the `translation settings` from the current user and update (if possible) its value.

Add to the `database/translations/settings/__init__.py` file the function `set_translation_setting`:

``` python
def set_translation_setting(db: Session, user_id: int, setting: str, value: bool) -> bool:
    user_settings = get_translation_settings(db, user_id)
    updated = user_settings.update_a_setting(setting, value)
    if updated:
        db.commit()
        return True
    return False
```

## Create the endpoint calling this function:
Let's join everything together 
Add to the `endpoints/translation/settings/__init__.py` file the route `set_translation_setting`:
``` python
@router.post('/set_translation_setting', tags=[TRANSLATION_TAG], response_model=BoolResponse)
async def set_translation_setting(
        params: Setting,
        user_id: str = Depends(get_current_user),
        db: Session = Depends(session_for_request),
):
    r = settings.set_translation_setting(db, user_id, params.identifier, params.value)
    return {'result': r}
```