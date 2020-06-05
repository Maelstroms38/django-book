---
title: Django Rest Framework
seoTitle: Django API Setup
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django Rest Framework

### By the end of this lesson, developers will be able to:

- Create serializers to format data attributes
- Create viewsets to create, read, update and destroy data 
- Create an API router for handling endpoint urls

### Introduction

In this section, we take a deep dive into Django Rest Framework. 

In a traditional data-driven website, a web application waits for **HTTP requests** from the web browser (or other client). When a **request** is received the application works out what is needed based on the URL and possibly information in POST data or GET data. Depending on what is required it may then read or write information from a database or perform other tasks required to satisfy the request. The application will then return a **response** to the web browser, often dynamically creating an HTML page for the browser to display by inserting the retrieved data into placeholders in an HTML template.

Django web applications typically group the code that handles each of these steps into separate files:

<img src="https://mdn.mozillademos.org/files/13931/basic-django.png">

- **URLs**: While it is possible to process requests from every single URL via a single function, it is much more maintainable to write a separate view function to handle each resource. A URL mapper is used to redirect HTTP **requests** to the appropriate view based on the request URL. The URL mapper can also match particular patterns of strings or digits that appear in an URL, and pass these to a view function as data.

- **View**: A view is a request handler function, which **receives HTTP requests and returns HTTP responses**. Views access the data needed to satisfy requests via models, and delegate the formatting of the response to templates.

- **Models**: Models are Python objects that define the structure of an application's data, and provide mechanisms to manage (add, modify, delete) and query records in the database. 

- **Templates**: A template is a text file defining the structure or layout of a file (such as an HTML page), with placeholders used to represent actual content. A view can dynamically create an HTML page using an HTML template, populating it with data from a model. A template can be used to define the structure of any type of file; it doesn't have to be HTML!

As a brief disclaimer, we are not going to work with **Templates** in this course. Instead, we will use a reactive web stack to host our pages content. 

## Installation

Without further ado, let's begin with the installation of Django Rest Framework.

```bash
pipenv install djangorestframework
```

Add to `library/library/settings.py`
```python

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'catalog',
    'rest_framework',
]
```

## Create API Router

To kick things off, we can create a new directory called `api`, and file called `urls.py`: 

```bash
mkdir library/catalog/api
touch library/catalog/api/urls.py
```

Inside `library/catalog/api/urls.py`, add the default router from `djangorestframework`:

```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
app_name = 'catalog-api'

urlpatterns = router.urls
```

Here we initialized the root router. This acts as the "entry point" to our application. Up next, we will add two new endpoints:
- `/books`
- `/authors`

Let's move on to our API views.

## Create Author, Book Serializers

Create a new file, called `library/catalog/api/serializers.py` and insert the following:

```python
from rest_framework import serializers
from catalog.models import Author, Book

class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = '__all__'

class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(required=False)
    class Meta:
        model = Book
        fields = '__all__'
```

Let's break down this file, as it contains a lot of useful functionality.

1. Whenever we query our `Author` or `Book` serializers, we specify the attributes we want.
2. Every `ModelSerializer` requires a `model` in order to fetch the correct database tables.
3. The `AuthorSerializer` and `BookSerializer` allow us to specify a list of fields as an array, or simply `__all__`.

When we use serializers, we are asking our database to transform a model into `JSON` values. 

## Create Author, Book Viewsets

Now that we created serializers, we can display `JSON` data using a `ModelViewSet`.

Add a new file, called `library/catalog/api/views.py`, and insert the following:

```python
from rest_framework import viewsets
from rest_framework.response import Response
from catalog.models import Author, Book
from catalog.api.serializers import AuthorSerializer, BookSerializer

# 1. Define the viewset as a ModelViewSet
class AuthorViewSet(viewsets.ModelViewSet):
	# 2. Specify the queryset as "all authors"
    queryset = Author.objects.all()
    # 3. Assign the serializer class
    serializer_class = AuthorSerializer

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

Let's break down this file, as it *also* contains a lot of useful functionality.

1. First we define two viewsets, which subclass the `ModelViewSet` provided by Django Rest Framework. 
2. To query from each model in our database, we define a `queryset` -- this is a list of data values. 
3. To fetch data from our `Author` and `Book` serializers, we define a `serializer_class` -- this is the class that transforms our models into `JSON` data.

## Wire Up Default Router

It's time to return to the root router in `library/catalog/api/urls.py`, and add the following:

```python
from rest_framework.routers import DefaultRouter

from catalog.api.views import AuthorViewSet, BookViewSet

router = DefaultRouter()
app_name = 'catalog-api'

router.register(r'authors', AuthorViewSet)
router.register(r'books', BookViewSet)
urlpatterns = router.urls
```

## URL Updates

Finally, inside `library/library/urls.py`, insert the following:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('catalog.api.urls'))
]
```

Ready to test it out? Run the server locally, and visit `http://127.0.0.1:8000/api/`.

```bash
pipenv run python3 manage.py runserver
```

You should be able to read, write and edit your authors and books.

Congratulations, today we created a mini library API. **WELL DONE!**

In the next section, we introduce Django filters, permissions and pagination.

## Sources
- [django-rest-framework](https://github.com/encode/django-rest-framework)