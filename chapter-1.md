---
title: Getting Started
seoTitle: Django Project Setup
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Hello, Django

In this section, we can begin to create a Django micro-API framework. First, we would like to consider a few key questions:

1. What are the benefits of using Django and Python?
  - In Django, lots of things are handled for you, with the full suite of customization options available. 
  - Python is amongst the most [popular](https://github.com/trending/python) coding languages to date. This means that there are plenty of open-source libraries and packages to choose from. 

2. How can I integrate Django with non-python projects?
  - In short, we will use Django to create an API interface.
  - We will cover two types of API interfaces: RESTful and GraphQL:
    - Query for your data using Django Rest Framework 
        - Models, Viewsets, Serializers (RESTful API)
    - Query for your data using Graphene 
        - Schema, Queries, Resolvers, Mutations (GraphQL)

## Tech Overview 

Here is an overview of the tech we will be using throughout the project.

### Backend (Python)
- [Django](https://www.djangoproject.com/)
- [Graphene](https://docs.graphene-python.org/projects/django/en/latest/tutorial-plain/)
- [Django-GraphQL-JWT](https://django-graphql-jwt.domake.io/en/latest/quickstart.html)

## Getting Started. 

### Django App Structure

<iframe frameborder="2" style="width:100%;height:400px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Django3.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1V4_FPPGXnL5xICFkiX5cuwA6909PqvXn%26export%3Ddownload"></iframe>

As pictured above, we will develop the app in three stages: 

1) Set up a Python virtual environment
2) Create a Django API interface
3) Create a Django GraphQL interface with Graphene

## Virtual Env and project structure. 

Let's kick things off with a new directory, called `libby`:

```bash
mkdir libby
cd libby
pipenv install django==3.0.5
pipenv run python3 -m django startproject library
cd library
pipenv run python3 manage.py startapp catalog
```

So far, we created a new `pipenv`, which acts as a container for our application. We use the `pipenv` to keep our application dependencies in check using `pip`. After installing Django, we save our list of dependencies in a file called `Pipfile`.

Django handles the rest, by creating a new project and application definition: `library`.

Our application structure is the following:

<iframe frameborder="0" style="width:100%;height:393px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Django%20Libby.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1skuFAhafoNmd7ZZxIgtRHJHLpx6tQA7I%26export%3Ddownload"></iframe>

Notice that by creating a project named `library`, Django also creates an application definition of `library`. This is where we would configure the root project settings, urls and templates.

## Django Models

Imagine you were designing an abstract definition of a `book` model. In fact, you were simply given a blob of JSON data to work with:

```json
{
  "data": {
    "books": [
      {
        "title": "The Girl With the Dragon Tattoo",
        "summary": "Forty years after the disappearance of Harriet Vanger from the secluded island owned and inhabited by the powerful Vanger family, her octogenarian uncle hires journalist Mikael Blomqvist and Lisbeth Salander, an unconventional young hacker, to investigate.",
        "author": {
          "firstName": "Stieg",
          "lastName": "Larsson"
        },
        "genre": [
          {
            "name": "Fiction"
          }
        ]
      },
      {
        "title": "The Hunger Games",
        "summary": "In a future North America, where the rulers of Panem maintain control through an annual televised survival competition pitting young people from each of the twelve districts against one another, sixteen-year-old Katniss's skills are put to the test when she voluntarily takes her younger sister's place.",
        "author": {
          "firstName": "Suzanne",
          "lastName": "Collins"
        },
        "genre": [
          {
            "name": "Fiction"
          }
        ]
      }
    ]
  }
}
```

You may be caught with your pants down, having no clue where to begin in designing these `Book`, `Author`, or `Genre` models.

While managing a database, we use models to write an abstract representation of our data nodes. Models contain sub-selections of [model fields](https://docs.djangoproject.com/en/3.0/ref/models/fields/).

The list of options here is extensive, which allows for more [model relationships](https://docs.djangoproject.com/en/3.0/topics/db/examples/many_to_one/). 

We can begin by writing our first model definitions for two new classes: `Author` and `Book`. We will create `Genre` later on.

### Django Models

Our database schema can be represented as follows:

- Author
    - `first_name`
    - `last_name`

- Book
    - `title`
    - `author`
    - `summary`

<iframe frameborder="0" style="width:100%;height:255px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Libby_ERD#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1BJ1aSXpxzJXWqg0AsJH9EXK-jVGP4fPn%26export%3Ddownload"></iframe>

Above, you can visualize the abstract relationship between `Author` and `Book` as a One to Many relationship. For now, we will allow one author to own many books.

Let's create the `Book` and `Author` models inside our application:

Insert the following code inside `library/catalog/models.py`:

```python
from django.db import models

class Book(models.Model):
	title = models.CharField(max_length=200)
	# Foreign Key used because book can only have one author, but authors can have multiple books
	# Author as a string rather than object because it hasn't been declared yet in the file
	author = models.ForeignKey('Author', on_delete=models.SET_NULL, null=True)
	summary = models.TextField(max_length=1000, help_text='Enter a brief description of the book')

	class Meta:
		ordering = ['-id']

	def __str__(self):
		"""String to represent the model object"""
		return self.title

class Author(models.Model):
	"""Model representing an author."""
	first_name = models.CharField(max_length=100)
	last_name = models.CharField(max_length=100)

	class Meta:
		ordering = ['last_name', 'first_name']

	def __str__(self):
		"""String for representing the Model object."""
		return f'{self.last_name}, {self.first_name}'
```

After creating the models, we can manage the database using Django's `admin` panel. To view the `admin` panel, we will need to register our models, app and create a superuser.

### Register Models

Insert the following inside `library/catalog/admin.py`:

```python
from django.contrib import admin

# Register your models here.
from .models import Author, Book

classes = [Author, Book]

for model in classes:
	admin.site.register(model)
```

### Register Catalog

Insert the following inside `library/library/settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'catalog',
]
```

### Admin Panel

Now we can manage the application models from Django's admin panel. Let's run the following commands:

1) Migrate our new `Book` and `Author` models to the database.
2) Create a super user without inputting sensitive credentials.

```bash
pipenv run python3 manage.py makemigrations
pipenv run python3 manage.py migrate
pipenv run python3 manage.py createsuperuser --username USERNAME --email EMAIL
pipenv run python3 manage.py runserver
```

- [Source: Django Docs](https://docs.djangoproject.com/en/3.0/ref/django-admin/#createsuperuser)

Try creating your new models with the Django admin panel at [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/). You should be able to login using the credentials from the previous command.

### Collect Static Assets

While testing out the admin panel we can collect static assets and stylesheets. This is useful for creating a stylish landing page for our website. You may notice how the admin panel uses [bootstrap](https://getbootstrap.com/) and is mobile responsive.

At the bottom of `settings.py`, add the following:

```python
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.0/howto/static-files/

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = os.environ.get('STATIC_URL', '/staticfiles/')
```

Then collect static assets:

```bash
pipenv run python3 manage.py collectstatic --noinput
```

- For production, Django websites typically use [Whitenoise](http://whitenoise.evans.io/en/stable/) for serving static files.

Congratulations, today we made a sample Django library app!

In the next section, we introduce Django Rest Framework and our first serializers.

## References
- [Coding for Entrepenuers](https://www.codingforentrepreneurs.com/)
- [docs.graphene-python.org](https://docs.graphene-python.org/projects/django/en/latest/tutorial-plain/)