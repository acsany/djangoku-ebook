# Djangoku - Heroku Setup

After the local basics are set, it is a good idea to deploy as soon as possible. This way you can spot and fic configuration errors without a big codebase and lot of dependencies.

## Presets
* [Heroku](https://www.heroku.com) account
* [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)


## Basic setup on heroku.com
You can do some of those steps on the [Heroku dashboard](https://dashboard.heroku.com/apps); this might be even more convenient to begin with. For this readme however, we try to stick to the terminal.

Before you start, is a good idea to see if the heroku CLI is working by running `heroku --version` and to update the heroku CLI client with `heroku update`

* Login with `heroku login` (or check which user is logged in: `heroku auth:whoami`)
* Create your heroku app: `heroku apps:create djangoku-1 --region eu` (Note: Change the app name or leave it completely to let heroku give it a random name)
* Check if heroku was automatically added as git remote: `git remote -v`
* Install gunicorn: `pip install gunicorn`
* Install whitenoise: `pip install whitenoise`
* Create settings file for heroku: `touch djangoku/settings/production.py`
* Create a requirements file: `pip freeze > requirements.txt`
* Create a Procfile: `touch Procfile`
* Create a runtime file: `echo python-3.7.2 > runtime.txt` (Run `python --version` for reference)
* Set environment variables for heroku (see `.env` for reference):
    * `heroku config:set DJANGO_SETTINGS_MODULE=djangoku.settings.production`
    * `heroku config:set DJANGO_SECRET_KEY=NEWLY%G3GENERATEDsecret_KEY!`

Add content:
**djangoku/settings/base.py**
Add whitenoise middleware right after secutiry middleware:
```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    #...
]
```

**djangoku/settings/production.py**
(Note: Change the URL to match your heroku app name)
```
from .base import *
DEBUG = False

ALLOWED_HOSTS = ['djangoku-1.herokuapp.com']

STATIC_URL = '/static/'
STATICFILES_DIRS = [
    'static/',
]
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

import dj_database_url
db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)

```

**Procfile**
```
release: python manage.py migrate --no-input
web: gunicorn djangoku.wsgi --log-file - --log-level debug
```

* Test configuration locally: `heroku local web` (just to see if the server starts properly)
* ***Important:*** Add all created files to git
* Push files to heroku: `git push heroku master`
* Check if app works: `heroku open`