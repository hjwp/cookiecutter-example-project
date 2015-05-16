project_name
==============================

A short description of the project.


LICENSE: BSD

Settings
------------

project_name relies extensively on environment settings which **will not work with Apache/mod_wsgi setups**. It has been deployed successfully with both Gunicorn/Nginx and even uWSGI/Nginx.

For configuration purposes, the following table maps the 'project_name' environment variables to their Django setting:

======================================= =========================== ============================================== ===========================================
Environment Variable                    Django Setting              Development Default                            Production Default
======================================= =========================== ============================================== ===========================================
DJANGO_AWS_ACCESS_KEY_ID                AWS_ACCESS_KEY_ID           n/a                                            raises error
DJANGO_AWS_SECRET_ACCESS_KEY            AWS_SECRET_ACCESS_KEY       n/a                                            raises error
DJANGO_AWS_STORAGE_BUCKET_NAME          AWS_STORAGE_BUCKET_NAME     n/a                                            raises error
DJANGO_CACHES                           CACHES (default)            locmem                                         memcached
DJANGO_DATABASES                        DATABASES (default)         See code                                       See code
DJANGO_DEBUG                            DEBUG                       True                                           False
DJANGO_EMAIL_BACKEND                    EMAIL_BACKEND               django.core.mail.backends.console.EmailBackend django.core.mail.backends.smtp.EmailBackend
DJANGO_SECRET_KEY                       SECRET_KEY                  CHANGEME!!!                                    raises error
DJANGO_SECURE_BROWSER_XSS_FILTER        SECURE_BROWSER_XSS_FILTER   n/a                                            True
DJANGO_SECURE_SSL_REDIRECT              SECURE_SSL_REDIRECT         n/a                                            True
DJANGO_SECURE_CONTENT_TYPE_NOSNIFF      SECURE_CONTENT_TYPE_NOSNIFF n/a                                            True
DJANGO_SECURE_FRAME_DENY                SECURE_FRAME_DENY           n/a                                            True
DJANGO_SECURE_HSTS_INCLUDE_SUBDOMAINS   HSTS_INCLUDE_SUBDOMAINS     n/a                                            True
DJANGO_SESSION_COOKIE_HTTPONLY          SESSION_COOKIE_HTTPONLY     n/a                                            True
DJANGO_SESSION_COOKIE_SECURE            SESSION_COOKIE_SECURE       n/a                                            False
======================================= =========================== ============================================== ===========================================

* TODO: Add vendor-added settings in another table

Getting up and running
----------------------

The steps below will get you up and running with a local development environment. We assume you have the following installed:

* pip
* virtualenv
* PostgreSQL

First make sure to create and activate a virtualenv_, then open a terminal at the project root and install the requirements for local development::

    $ pip install -r requirements/local.txt

.. _virtualenv: http://docs.python-guide.org/en/latest/dev/virtualenvs/

You can now run the ``runserver_plus`` command::

    $ python manage.py runserver_plus

The base app will run but you'll need to carry out a few steps to make the sign-up and login forms work. These are currently detailed in `issue #39`_.

.. _issue #39: https://github.com/pydanny/cookiecutter-django/issues/39

**Live reloading and Sass CSS compilation**

If you'd like to take advantage of live reloading and Sass / Compass CSS compilation you can do so with the included Grunt task.

Make sure that nodejs_ is installed. Then in the project root run::

    $ npm install grunt

.. _nodejs: http://nodejs.org/download/

Now you just need::

    $ grunt serve

The base app will now run as it would with the usual ``manage.py runserver`` but with live reloading and Sass compilation enabled.

To get live reloading to work you'll probably need to install an `appropriate browser extension`_

.. _appropriate browser extension: http://feedback.livereload.com/knowledgebase/articles/86242-how-do-i-install-and-use-the-browser-extensions-

It's time to write the code!!!


Deployment
------------

It is possible to deploy to Heroku or to your own server by using Dokku, an open source Heroku clone.

Heroku
^^^^^^

Run these commands to deploy the project to Heroku:

.. code-block:: bash

    heroku create --buildpack https://github.com/heroku/heroku-buildpack-python

    heroku addons:add heroku-postgresql:dev
    heroku pg:backups schedule DATABASE_URL
    heroku pg:promote DATABASE_URL

    heroku addons:add sendgrid:starter
    heroku addons:add memcachier:dev

    heroku config:set DJANGO_SECRET_KEY=RANDOM_SECRET_KEY_HERE
    heroku config:set DJANGO_SETTINGS_MODULE='config.settings.production'

    heroku config:set DJANGO_AWS_ACCESS_KEY_ID=YOUR_AWS_ID_HERE
    heroku config:set DJANGO_AWS_SECRET_ACCESS_KEY=YOUR_AWS_SECRET_ACCESS_KEY_HERE
    heroku config:set DJANGO_AWS_STORAGE_BUCKET_NAME=YOUR_AWS_S3_BUCKET_NAME_HERE

    git push heroku master
    heroku run python manage.py migrate
    heroku run python manage.py check --deploy
    heroku run python manage.py createsuperuser
    heroku open

PythonAnywhere
^^^^^^^^^^^^^^


Make sure your project is fully commited and pushed up to Bitbucket or Github or wherever it may be.  Then, log into your PythonAnywhere account, open up a **Bash** console, clone your repo, and create a virtualenv:

.. code-block:: bash

    git clone <my-repo-url>
    cd my-project-name
    mkvirtualenv --python=/usr/bin/python3.4 my-project-name # or python2.7, etc
    pip install -r requirements/production.txt  # may take a few minutes

Generate a secret key for yourself, eg like this:

.. code-block:: bash

    python -c 'import random; print("".join(random.SystemRandom().choice("abcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*(-_=+)") for _ in range(50)))'

Make a note of it, since we'll need it here in the console and later on in the web app config tab.

Set environment variables via the virtualenv "postactivate" script (this will set them every time you use the virtualenv in a console):

.. code-block:: bash

    vi $VIRTUAL_ENV/bin/postactivate
    # You can also edit this file via the PythonAnywhere "Files" menu;
    # look in the .virtualenvs folder.

Add these exports

.. code-block:: bash

    export DJANGO_SETTINGS_MODULE='config.settings.production'
    export DJANGO_SECRET_KEY='<secret key goes here>'
    export SENDGRID_USERNAME='<sendgrid username>'
    export SENDGRID_PASSWORD='<sendgrid password>'
    export DJANGO_AWS_ACCESS_KEY_ID=
    export DJANGO_AWS_SECRET_ACCESS_KEY=
    export DJANGO_AWS_STORAGE_BUCKET_NAME=

* The AWS details are not required if you're using whitenoise or the built-in pythonanywhere static files service, but you do need to set them to blank, as above

Database setup:
"""""""""""""""

Go to the PythonAnywhere databases tab and configure your database.

* For Postgres, setup your superuser password, then open a Postgres console and run a `CREATE DATABASE my-db-name`.  You should probably also set up a specific role and permissions for your app, rather than using the superuser credentials.  Make a note of the address and port of your postgres server.

* For MySQL, set the password and create a database.  Be aware that Django's support for MySQL under Python 3 is still patchy.

* Alternatively, you can just use sqlite!

Now go back to the `postactivate` script and set the `DATABASE_URL` environment variable:

.. code-block:: bash

    vi $VIRTUAL_ENV/bin/postactivate

.. code-block:: bash

    export DATABASE_URL='postgres://<postgres-username>:<postgres-password>@<postgres-address>:<postgres-port>/<database-name>'
    # or
    export DATABASE_URL='mysql://<pythonanywhere-username>:<mysql-password>@mysql.server/<database-name>'
    # or
    export DATABASE_URL='sqlite:////absolute/path/to/db.sqlite'

If you're using MySQL, you'll probably need to run a `pip install MySQLdb`, and maybe add `MySQLdb` to `requirements/production.txt` too.

Now run the migration, and collectstatic:

.. code-block:: bash

    source $VIRTUAL_ENV/bin/postactivate
    python manage.py migrate
    python manage.py collectstatic
    # and, optionally
    python manage.py createsuperuser


Web App config
""""""""""""""

Go to the PythonAnywhere **Web** tab, hit **Add new web app**, and choose **Manual Config**, and then the version of Python you used for your virtualenv.

When you're redirected back to the web app config screen, set the path to your virtualenv.  If you used virtualenvwrapper as above, you can just enter its name.

Click through to the **WSGI configuration file** link (near the top) and edit the wsgi file. Make it look something like this, repeating the environment variables you used earlier:


.. code-block:: python

    import os
    import sys
    path = '/home/<your-username>/<your-project-directory>'
    if path not in sys.path:
        sys.path.append(path)

    os.environ['DATABASE_URL'] = '<database url as above>'
    os.environ['DJANGO_SETTINGS_MODULE'] = 'config.settings.production'
    os.environ['DJANGO_SECRET_KEY'] = '<secret key as above>'
    os.environ['SENDGRID_PASSWORD'] = ''
    os.environ['SENDGRID_USERNAME'] = ''
    os.environ['DJANGO_AWS_ACCESS_KEY_ID'] = ''
    os.environ['DJANGO_AWS_SECRET_ACCESS_KEY'] = ''
    os.environ['DJANGO_AWS_STORAGE_BUCKET_NAME'] = ''

    from django.core.wsgi import get_wsgi_application
    application = get_wsgi_application()

Back on the Web tab, hit **Reload**, and your app should be live!

NB - you will see security warnings until you set up your SSL certificates. If you
want to supress them temporarily, set `DJANGO_SECURE_SSL_REDIRECT` to blank.  Follow
the instructions here to get SSL set up: https://www.pythonanywhere.com/wiki/SSLOwnDomains 

Optional: static files
""""""""""""""""""""""

If you want to use the PythonAnywhere static files service instead of using whitenoise or AWS, you'll find its configuration section on the Web tab.  Essentially you'll need an entry to match your `STATIC_URL` and `STATIC_ROOT` settings.  There's more info here: https://www.pythonanywhere.com/wiki/DjangoStaticFiles 


Future deployments
""""""""""""""""""

For subsequent deployments, the procedure is much simpler.  In a Bash console:

.. code-block:: bash

    workon my-virtualenv-name
    cd project-directory
    git pull
    python manage.py migrate
    python manage.py collectstatic

And then go to the Web tab and hit "Reload"



Dokku
^^^^^

You need to make sure you have a server running Dokku with at least 1GB of RAM. Backing services are
added just like in Heroku however you must ensure you have the relevant Dokku plugins installed.

.. code-block:: bash

    cd /var/lib/dokku/plugins
    git clone https://github.com/rlaneve/dokku-link.git link
    git clone https://github.com/jezdez/dokku-memcached-plugin memcached
    git clone https://github.com/jezdez/dokku-postgres-plugin postgres
    dokku plugins-install

You can specify the buildpack you wish to use by creating a file name .env containing the following.

.. code-block:: bash

    export BUILDPACK_URL=<repository>

You can then deploy by running the following commands.

..  code-block:: bash

    git remote add dokku dokku@yourservername.com:project_name
    git push dokku master
    ssh -t dokku@yourservername.com dokku memcached:create project_name-memcached
    ssh -t dokku@yourservername.com dokku memcached:link project_name-memcached project_name
    ssh -t dokku@yourservername.com dokku postgres:create project_name-postgres
    ssh -t dokku@yourservername.com dokku postgres:link project_name-postgres project_name
    ssh -t dokku@yourservername.com dokku config:set project_name DJANGO_SECRET_KEY=RANDOM_SECRET_KEY_HERE
    ssh -t dokku@yourservername.com dokku config:set project_name DJANGO_SETTINGS_MODULE='config.settings.production'
    ssh -t dokku@yourservername.com dokku config:set project_name DJANGO_AWS_ACCESS_KEY_ID=YOUR_AWS_ID_HERE
    ssh -t dokku@yourservername.com dokku config:set project_name DJANGO_AWS_SECRET_ACCESS_KEY=YOUR_AWS_SECRET_ACCESS_KEY_HERE
    ssh -t dokku@yourservername.com dokku config:set project_name DJANGO_AWS_STORAGE_BUCKET_NAME=YOUR_AWS_S3_BUCKET_NAME_HERE
    ssh -t dokku@yourservername.com dokku config:set project_name SENDGRID_USERNAME=YOUR_SENDGRID_USERNAME
    ssh -t dokku@yourservername.com dokku config:set project_name SENDGRID_PASSWORD=YOUR_SENDGRID_PASSWORD
    ssh -t dokku@yourservername.com dokku run project_name python manage.py migrate
    ssh -t dokku@yourservername.com dokku run project_name python manage.py createsuperuser

When deploying via Dokku make sure you backup your database in some fashion as it is NOT done automatically.
