---
title: Graphene
seoTitle: Django Graphene
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django Graphene

### By the end of this lesson, developers will be able to:

- Query for library books with GraphQL
- Setup input types for book mutations
- Create a Graphene GraphQL schema

### Introduction

As we develop the library API, it would be useful to understand how we can extend our query interface with GraphQL. As it turns out, there is already support for GraphQL and Python using the [Graphene](https://docs.graphene-python.org/projects/django/en/latest/installation/) library.

## Installation

```bash
pipenv install graphene-django
```

Add `graphene_django` to the INSTALLED_APPS in the `settings.py` file of your Django project:

```python
INSTALLED_APPS = [
    'django.contrib.staticfiles', # Required for GraphiQL
    'graphene_django'
]

GRAPHENE = {
    'SCHEMA': 'library.schema.schema'
}
```

## Queries

We are going to add the ability to query a list of books and authors. 

Let's begin with a new file called `library/catalog/schema.py`:

```python
from graphene import relay, ObjectType
from graphene_django import DjangoObjectType
from graphene_django.filter import DjangoFilterConnectionField
from catalog.filters import BookFilter
from catalog.models import Author, Book

# 1. Each object becomes a "node"
class BookNode(DjangoObjectType):
    class Meta:
        model = Book
        interfaces = (relay.Node, )

class AuthorNode(DjangoObjectType):
    class Meta:
        model = Author
        filter_fields = []
        interfaces = (relay.Node, )

# 2. Queries can fetch nodes using a ConnectionField (with filters)
class Query(ObjectType):
    book = relay.Node.Field(BookNode)
    books = DjangoFilterConnectionField(BookNode, filterset_class=BookFilter)
    author = relay.Node.Field(AuthorNode)
    authors = DjangoFilterConnectionField(AuthorNode)
```

With the above snippet, we are reusing the `BookFilter`, and creating two new lists for `books` and `authors`.

1. For each `DjangoObjectType`, we add a unique node. With Django and Graphene, we are also utilizing cursor-based pagination, provided by [Relay](https://relay.dev/docs/en/graphql-server-specification.html).

> For more information about GraphQL pagination, visit [graphql.org](https://graphql.org/learn/pagination/#pagination-and-edges)

2. With each node, we establish a connection field and include an array of filters. 

> For more about GraphQL connections, visit [relay.dev](https://relay.dev/graphql/connections.htm)

## Schema

Let's define the root application schema definition and begin with our queries.

Create a new file, called `schema.py` and add it to `library/library/schema.py`.

```python
from graphene import Schema, ObjectType
import catalog.schema

class Query(catalog.schema.Query, ObjectType):
    # This class will inherit from multiple Queries
    # as we begin to add more apps to our project
    pass

class Mutation(ObjectType):
    # This class will inherit from multiple Mutations
    # as we begin to add more apps to our project
    pass

schema = Schema(query=Query)
```

For schemas, here are the main takeaways:

1. The above schema acts as an entry point to access GraphQL data, through `Query` and `Mutation` classes. 

2. The schema's type system allows us to define what object types are available on the backend.

> For more information about GraphQL schemas, visit [graphql.org](https://graphql.org/learn/schema/)

## Mutations 

In order to modify the books in our library, we can continue to create mutations.

Add the following mutations to `library/catalog/schema.py`:

```python
from catalog.api.serializers import BookSerializer
from graphene_django.rest_framework.mutation import SerializerMutation
# ...

class BookMutation(SerializerMutation):
    class Meta:
        serializer_class = BookSerializer

class Mutation(ObjectType):
    book_mutation = BookMutation.Field()
```


### SerializerMutation 

In this mutation, are taking advantage of the `BookSerializer` we wrote earlier. In fact, the serialzier is going to run into some issues from it's nested `Author` attribute. We will resolve those next. 

For some context, the issue pertains to the way Django Rest Framework handles nested model updates. We cannot simply "update books" with a nested attribute, so we can now add two methods to the `BookSerializer`:

- `create`
- `update`

## Book Serializer Updates

Add the following methods to the `BookSerializer` inside `library/catalog/api/serializers.py`:

```python
class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(required=False)
    class Meta:
        model = Book
        fields = '__all__'
        # 1. Add extra keyword arguements to allow readable IDs
        extra_kwargs = {
            'id': {'read_only': False, 'required': False}
        }

    # 2. Define create method to assign a nested authors attributes
    def create(self, validated_data):
        author_data = validated_data.pop('author')
        author, created = Author.objects.get_or_create(**author_data)
        book = Book.objects.create(author=author, **validated_data)
        return book
    # 3. Define update method to update book and authors attributes
    def update(self, instance, validated_data):
        author_data = validated_data.pop('author')
        author, created = Author.objects.get_or_create(**author_data)
        instance.author = author
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        return instance
```

In summary:
1. We added `extra_kwargs` in order to make the book ID readable. 
2. Adding the `create` method lets us choose how to handle a nested author.
3. We also apply the `update` method in order to patch the book attributes.

## Schema Updates

Make the following additions to `library/library/schema.py`:

```python
from graphene import Schema, ObjectType
import catalog.schema

class Query(catalog.schema.Query, ObjectType):
    # This class will inherit from multiple Queries
    # as we begin to add more apps to our project
    pass

class Mutation(catalog.schema.Mutation, ObjectType):
    # This class will inherit from multiple Mutations
    # as we begin to add more apps to our project
    pass

schema = Schema(query=Query, mutation=Mutation)
```

## URL Updates

Finally, inside `library/library/urls.py`, insert the following:

```python
from django.contrib import admin
from django.urls import path, include
from django.views.decorators.csrf import csrf_exempt
from graphene_django.views import GraphQLView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('catalog.api.urls'))
    path('graphql/', csrf_exempt(GraphQLView.as_view(graphiql=True))),
]
```

## Test Queries and Mutations

Let's run the application to test our queries and mutations:

```bash
pipenv run python3 manage.py runserver
```

Visit [http://127.0.0.1:8000/graphql](http://127.0.0.1:8000/graphql), and try fetching all books, with their respective authors:

```
query {
  books {
    edges {
      node {
        id
        title
        summary
        author {
          id
          firstName
          lastName
        }
      }
    }
  }
}
```

Now try adding a new book:

```
mutation {
  bookMutation(input: {
    title: "A Game of Thrones",
    summary: "First book in the epic series",
    author:{
      firstName: "George R.R.",
      lastName: "Martin"
    }
  }) {
    id
    title
    summary
    author {
      firstName
      lastName
    }
  }
}
```

Creating, fetching and updating books works successfully! **WELL DONE!!**

In the next section, we introduce GraphQL Authentication.

## References
- [akiradev.netlify.com](https://akiradev.netlify.com/posts/django-graphql-api/)
- [apirobot.me](https://apirobot.me/posts/django-react-ts-how-to-create-and-consume-graphql-api)
- [howtographql.com/](https://www.howtographql.com/graphql-python/8-pagination/)
