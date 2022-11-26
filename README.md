# How to deploy a django project with staticfiles on vercel
 In this post I am going to show you how you can deploy your django or django rest framework project on vercel in 8 easy steps.

<b>Please note</b>
<br>
For this post I am assuming that you have setup your django project.
This process works on django framework and django restframework projects.
And also for my project databse I am using postgres_db.
Vercel does not provide support for dbsqlite3 currently, if you try to deploy your project with django's default dbsqlite3 database then its going to lead to an error. Ensure to comment out your dbsqlite database configuration or switch to a different database (e.g postgreSQL).
Alright lets begin;

<h3>Step 1</h3>
In your projects root directory create a new file and name it build_files.sh then paste the code below inside it

```shell
# build_files.sh
pip install -r requirements.txt
python3.9 manage.py collectstatic
```

<h3>Step 2</h3>
In your projects root directory create a new json file and name it vercel.json then paste the code below inside it

```javascript
{
  "version": 2,
  "builds": [
    {
      "src": "projectname/wsgi.py",
      "use": "@vercel/python",
      "config": { "maxLambdaSize": "15mb", "runtime": "python3.9" }
    },
    {
      "src": "build_files.sh",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "staticfiles_build"
      }
    }
  ],
  "routes": [
    {
      "src": "/static/(.*)",
      "dest": "/static/$1"
    },
    {
      "src": "/(.*)",
      "dest": "projectname/wsgi.py"
    }
  ]
}
```

<h3>Step 3</h3>
Go to your projects setting.py and paste in the code below

```python
STATICFILES_DIRS = os.path.join(BASE_DIR, 'static'),
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles_build', 'static')
```

<h3>Step 4</h3>
If you are using dbsqlite3 then comment it out like so. Since this is going to be a production enviroment I highly recommend that you should switch to another database such as postgreSQL.

```python
DATABASES = {
    # 'default': {
    #     'ENGINE': 'django.db.backends.sqlite3',
    #     'NAME': BASE_DIR / 'db.sqlite3',
    # }
}
```

<h3>Step 5</h3>
If you commented out your database as shown above then skip this step.
But if you took my advice and switched to a diffrent database then run the following commands in your terminal


```shell
python manage.py makemigrations
python manage.py migrate
```

<h3>Step 6</h3>
Next step is to go to your settings.py file and update your ALLOWED_HOST to permit vercel.app to be a valid hostname for your django app like so


```python
ALLOWED_HOSTS = ['.vercel.app', '.now.sh']
```

<h3>Step 7</h3>
Go to your wsgi.py file created for you by django in your peoject directory and paste in the line of code below

```python
app = application
```

<h3>Step 8</h3>
Finally upload your code to a github repo and connect your github account with said repo to your vercel dashboard.

Import the project.

You can change the default project name to something else if you wish but ensure to leave all other settings as default.

Then click on deploy and after a couple of seconds your project will be deployed successfully on vercel.

Finally go to this link to see the example project on my github page.

Ps. Checkout the link below to learn how you can add a free postgreSQL database for your project.

Hosting PostgreSQL database on Supabase for free (devmaesters.com)

If you are migrating your django site from one platform(e.g heroku) to another platform (e.g vercel) then checkout the link below to learn how you can migrate your postgreSQL database to supabase for free.

Migrating your PostgreSQL database from one hosting platform to another (e.g Heroku to Supabase) (devmaesters.com)


