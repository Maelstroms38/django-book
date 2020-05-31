---
title: Django Filters
seoTitle: Django Filter Setup
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django Filters

### By the end of this lesson, developers will be able to:

- Create a filter for their database models
- Customize search fields for unique models

## Installation

Now that we have a mini library API, we can add some search functionality to the backend. 

Let's begin by installing the `django-filter` library:

```bash
pip3 install django-filter
```

If you want to use the django-filter backend by default, add it to the DEFAULT_FILTER_BACKENDS setting.

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'django_filters',
]

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
}
```

## Adding Filters

Let's create the functionality to filter books by name.

Create a *new* file called `library/catalog/filters.py`:

```python
import django_filters
from .models import Book

class BookFilter(django_filters.FilterSet):
	title = django_filters.CharFilter(lookup_expr='icontains')
	author__first_name = django_filters.CharFilter(lookup_expr='icontains')
	author__last_name = django_filters.CharFilter(lookup_expr='icontains')

	class Meta:
		model = Book
		fields = ('title', 'author__first_name', 'author__last_name')
```

Back inside `library/catalog/api/views.py`, add the following:

```python
from django_filters import rest_framework as filters
from catalog.filters import BookFilter

... 

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = (filters.DjangoFilterBackend,)
    filterset_class = BookFilter
```

Now we can revisit the API docs and try a search for books at `http://127.0.0.1/api/books?title=gone+girl`:

```bash
(libby) $ python3 manage.py runserver
```

## DRF Permissions

This API is coming together nicely, but as of now anyone can add, edit, or delete the entire library!

Let's take a look at securing the objects in our database with permissions.

Add the following to `library/catalog/api/views.py`:

```python
from rest_framework import viewsets, permissions
# ...

class AuthorViewSet(viewsets.ModelViewSet):
    queryset = Author.objects.all()
    serializer_class = AuthorSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    filter_backends = (filters.DjangoFilterBackend,)
    filterset_class = BookFilter
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
```

## DRF Pagination

For pagination, we simply update our `settings.py`:

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
}
```

Now we can pass in `limit` and `offset` parameters into our queries: `http://127.0.0.1/api/books?limit=10`.

With that, we have developed our mini library API with filters, permissions and pagination. **WELL DONE!**

In the next section, we introduce GraphQL queries with Graphene.

## Sources
- [django-rest-framework](https://github.com/encode/django-rest-framework/blob/master/docs/api-guide/filtering.md)