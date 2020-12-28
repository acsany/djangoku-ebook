# djangoku
A boilerplate Django app with a GitLab remote repository that auto-deploys to heroku.

## Prerequisites
* Python 3.8+
* Django 2.2+
* Postgres 11+
* GitLab Account
* Heroku Account
 
## Setup
* Create working directory ("djangoku") with a git repo: `git init djangoku`
* Create remote Gitlab repo and add it as remote. E. g.: `git remote add origin https://gitlab.com/axani/djangoku.git`
* Create a virtual environment: `python3 -m venv virtualenv_mac`
* Activate the virtual environment: `source virtualenv_mac/bin/activate`