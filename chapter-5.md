---
title: Django Authentication
seoTitle: Django GraphQL Auth
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django GraphQL Auth

### By the end of this lesson, developers will be able to:

- Login or sign up using token authentication
- Create a user authentication mutation 

### Introduction

Now that we have queries and mutations for books and authors, we would like to allow our users to log in and sign up for an account. Building auth is easier with the `django-graphql-jwt` package, which we will cover. 

First, let's cover the Django user model.

## Django User Model

Django's built in user model covers most of our authentication needs. What it is missing is the ability to create users with our GraphQL interface. 

For managing users, we can simply create a `users` app and add any new authentication mutations there.

Let's kick things off with the following command:

```bash
pipenv run python3 manage.py startapp users
```

As a result, you will notice a new `users` directory. We are not going to edit any of the files in the root, like `models` or `views`. Instead, we can create a subdirectory for GraphQL queries and mutations.

Add the new `users` app to `library.settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'users',
]
```

Inside `library/users`, create a new `gql` directory and three new files:

```bash
mkdir users/gql
touch users/gql/mutations.py
touch users/gql/schema.py
touch users/gql/types.py
``` 

### User Type

Let's begin with adding the following GraphQL types to `users/gql/types.py`:

```python
from graphene_django import DjangoObjectType
from django.contrib.auth import get_user_model

User = get_user_model()

class UserType(DjangoObjectType):
    class Meta:
        model = User
        exclude = ('password', )
```

By declaring a `User` type, we can include or exclude certain fields from our mutations, like `password`. You may notice that every object in graphene has to subclass as a `DjangoObjectType`. 

If we want to declare custom model mutations, we need to work with the `DjangoObjectType` and declare it's fields.

### User Mutations

Let's move on to the `UserCreate` mutation. Insert the following inside `users/gql/mutations.py`:

```python
from graphene import Field, Mutation, String
from django.contrib.auth import get_user_model
from .types import UserType

User = get_user_model()

class UserCreate(Mutation):
	# 1. Declare the mutations result field
    user = Field(UserType)

    # 2. Declare arguments for the mutation
    class Arguments:
        username = String(required=True)
        password = String(required=True)
        email = String(required=True)

    # 3. Create new user model
    def mutate(self, info, username, password, email):
        user = User(
            username=username,
            email=email,
        )
        user.set_password(password)
        user.save()

        return UserCreate(user=user)
```

We included the Django user model to create an authentication mutation. Here's whats happening under the hood:

1. When we create custom mutations, we declare a type for our return value. In this case, we reference the `UserType` from earlier.

2. Like any function, mutations expect input arguments to run and recieve data. Creating a new user requires `username`, `email` and password.

3. Finally the graphene mutation handles creating, saving and returning a new user record. 

Alright, so now we can wire up the user schema. Add the following to `users/gql/schema.py`:

```python
from graphene import Field, List, ObjectType, Schema
from .types import UserType
from .mutations import UserCreate
from django.contrib.auth import get_user_model
# from graphql_jwt.decorators import login_required
# import graphql_jwt

User = get_user_model()

class Query(ObjectType):
    current_user = Field(UserType)
    users = List(UserType)

    def resolve_users(root, info):
        return User.objects.all()

    # @login_required
    def resolve_current_user(root, info):
        user = info.context.user
        return user

class Mutation(ObjectType):
    user_create = UserCreate.Field()

schema = Schema(query=Query, mutation=Mutation)
```

You can compare this with our previous schema in `catalog/schema.py`. Rather than throw everything into the same file, we separated out the types and mutations.

If you prefer to design around your GraphQL schema, this is a preferred approach. Our previous schema leveraged Django Rest Framework, this time we are focusing on the schema definition.

With this schema we can create and query users. Now let's add the `django-graphql-jwt` package to login, refresh and verify JSON Web Tokens.
 
## Django GraphQL JWT

### Installation

```bash
pipenv install django-graphql-jwt
```

Add the following to `library/library/settings.py`:

```python
GRAPHENE = {
    'SCHEMA': 'library.schema.schema',
    'MIDDLEWARE': [
        'graphql_jwt.middleware.JSONWebTokenMiddleware',
    ],
}

AUTHENTICATION_BACKENDS = [
    'graphql_jwt.backends.JSONWebTokenBackend',
    'django.contrib.auth.backends.ModelBackend',
]
```

Adding the `django-graphql-jwt` library allows us to include three new authentication mutations:

1. ObtainJSONWebToken
2. Verify
2. Refresh 

We can infer that the JSON web token will be included with each Authorization header. Here's an example HTTP request:

```
POST / HTTP/1.1
Host: domake.io
Authorization: JWT <token>
Content-Type: application/json;
```

Back in `users/gql/schema.py`, make the following additions:

```python
from graphql_jwt.decorators import login_required
from graphql_jwt import ObtainJSONWebToken, Verify, Refresh

# ...

class Query(ObjectType):
    current_user = Field(UserType)

    def resolve_users(root, info):
        return User.objects.all()

    @login_required
    def resolve_current_user(root, info):
        user = info.context.user
        return user

class Mutation(ObjectType):
    user_create = UserCreate.Field()
    token_auth = ObtainJSONWebToken.Field()
    verify_token = Verify.Field()
    refresh_token = Refresh.Field()

schema = Schema(query=Query, mutation=Mutation)
```

Finally return to the root schema inside `library/schema.py`, and insert our `users` schema.

```python
from graphene import Schema, ObjectType
import catalog.schema
import users.gql.schema

class Query(catalog.schema.Query, users.gql.schema.Query, ObjectType):
	# This class will inherit from multiple Queries
	# as we begin to add more apps to our project
	pass

class Mutation(catalog.schema.Mutation, users.gql.schema.Mutation, ObjectType):
	# This class will inherit from multiple Mutations
	# as we begin to add more apps to our project
	pass

schema = Schema(query=Query, mutation=Mutation)
```

## Test Queries and Mutations 

Let's fire up a local server and test out the authentication flow:

```bash
pipenv run python3 manage.py runserver
```

Visit [http://127.0.0.1:8000/graphql](http://127.0.0.1:8000/graphql) and try fetching all users:

```
query {
  users {
    id
    username
  }
}
```

You can look up anything about a user except their `password`.

Now let's create a new user:

```
mutation {
  userCreate(username: "alice", email: "alice@gmail.com", password:"password") {
    user {
      id
      username
      email
      dateJoined
    }
  }
}
```

Login with the new user:

```
mutation {
  tokenAuth(username: "alice", password: "password") {
    token
    payload
    refreshExpiresIn
  }
}
```

Viola! Authentication with Django and Graphene is now working as expected. **WELL DONE!!**

In the next section we will let users write reviews for their favorite books. 

## Sources
- [django-graphql-jwt](https://django-graphql-jwt.domake.io/en/latest/)