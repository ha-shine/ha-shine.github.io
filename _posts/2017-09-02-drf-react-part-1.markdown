---
layout: post
title:  "Django Rest TokenAuthentication with React Frontend Part 1"
date:   2017-09-02 22:56:38 +0800
categories: django django-rest reactjs tokenauthentication
---
As the title says I will show you how to write [Django-Rest-Framework](http://www.django-rest-framework.org) endpoint and [reactjs](https://facebook.github.io/react/) front-end with TokenAuthentication in mini series. I will show you how to do the user registration, logging in and out, and a way for registered users to see their profile. I will use my own custom user model for finer control. Without much further ado, let's dive right in!

## Setup

If you have pip installed, great. If not, install pip for python. You can read up on how to install pip [here](https://pip.pypa.io/en/stable/installing/). Let's install django and django-rest-framework.

```shell
pip install django djangorestframework
```

For react, I will use create-react-app, if you don't know what is this, it's [here](https://github.com/facebookincubator/create-react-app). It is the easiest way to start a react project without worrying about the dark arts of configuration that is webpack. Install create-react-app globally using npm.

```shell
npm install --global create-react-app
```

After getting installation out of our way, let's continue with setting up django and react project. Let's start with an empty directory. In that empty directory we will run these two commands.

```shell
django-admin startproject server # create a django project called server
create-react-app client # create a react project called client
```

You should be seeing something like below in your directory after this step.

```
.
├── client
│   ├── node_modules
│   ├── package.json
│   ├── package-lock.json
│   ├── public
│   ├── README.md
│   └── src
└── server
    ├── manage.py
    └── server
    
```

## Django Configuration

Let's cd into server folder and do some configuration so our djangorestframework works. The steps from now on assumes you are in the server directory. Before doing anything let's make sure Django recognizes our django-rest-framework. In `./server/settings.py` add `rest_framework` into `INSTALL_APPS`.

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
)
```

Since we will be using browsable API for testing and we need to login and logout, add this into `./server/urls.py`.

```python
from django.conf.urls import url, include

urlpatterns = [
    ...
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework')),
]
```

Alright, great. At the same time, let's add these lines into `./server/settings.py` to make sure django-rest is using`TokenAuthentication`.

```python
INSTALLED_APPS = (
    ...
    'rest_framework.authtoken'
)

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    )
}
```

We are doing great. Let's start a new app called `myapp` by running this command.

```shell
python manage.py startapp myapp
```

Awesome, now we got a new directory called `myapp` in the server directory.  Don't forget to add this into our `INSTALLED_APPS` in `./server/settings.py`.

```python
INSTALLED_APPS = (
    ...
    'myapp.apps.MyappConfig',
)
```

After this, we will need to create our own user model. Let's add some models into `./myapp/models.py`.

```python
from django.db import models
from django.contrib.auth.models import (
    BaseUserManager, AbstractBaseUser
)


class CustomUserManager(BaseUserManager):

    def create_user(self, email, password=None):
        if not email:
            raise ValueError('Users must have an email')

        user = self.model(
            email=self.normalize_email(email),
        )

        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, date_of_birth, password):
        user = self.create_user(
            email,
        )

        user.set_password(password)
        user.is_staff = True
        user.save(using=self._db)
        return user


class CustomUser(AbstractBaseUser):
    email = models.EmailField(
        verbose_name='email address',
        max_length=255,
        unique=True
    )
    first_name = models.CharField(max_length=255, default='')
    last_name = models.CharField(max_length=255, default='')
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'

    def __str__(self):
        return self.email

    def get_full_name(self):
        return '%s %s' % (self.first_name, self.last_name)

    def get_short_name(self):
        return self.email
```

Ok there is a lot going on in this snippet.

- `CustomUser` is our user model for handling user authentication. There is nothing much going on in this model. `USERNAME_FIELD` is used to identify `email` field is used as the username.
- `CustomUserManager` is our model manager for user model created above. You can read more about customizing authentication in this [link](https://docs.djangoproject.com/en/1.11/topics/auth/customizing/).

We still need to inform our root app that we will be using this `CustomUser` model in place of the default user account. Add this to `./server/settings.py`.

```python
AUTH_USER_MODEL = 'myapp.CustomUser'
```

Great job so far! Before moving onto next section, let's do the migration by running this.

```shell
python manage.py makemigrations myapp
python manage.py migrate
```

Let's test whether our models are working. Open a shell session by running `python manage.py shell`, and you will see the python shell for our project. Write these in that said shell.

```python
from myapp.models import CustomUser
CustomUser.objects.create_user(email='user1@test.com', password='password')
```
This will create a user with an email `user1@test.com`, if you try to create another user with the same email, you will be slapped in the face with `django.db.utils.IntegrityError`. Go create one if you don't believe me, go ahead, I dare you. When you look at the password of x after creation with `x.password`, you will see that the password is hashed by default. In the next part I will be showing on to use DRF serializers, and views plus how to test them using Browsable API.