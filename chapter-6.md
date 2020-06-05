---
title: Django Reviews
seoTitle: Django Book Reviews
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django Book Reviews

### By the end of this lesson, developers will be able to:

- Create, update and destroy book reviews

### Introduction

Now that we have working authentication, we will let users review their favorite books!

First, let's create the new `Review` model.

## Review Model

Each review will have the following properties:

- Review
    - `book`
    - `user`
    - `pub_date`
    - `comment`
    - `value`

Let's kick things off with the following command:

```bash
pipenv run python3 manage.py startapp reviews
```

Add the new app to `library.settings.py`:

```python
INSTALLED_APPS = [
    # ...
    'reviews',
]
```

Insert the following inside `library/reviews/models.py`:

```python
from django.db import models
from django.conf import settings
from catalog.models import Book

# Create your models here.

class Review(models.Model):

	"""
    Represents a rating for one book.
    :book: Book review is associated with.
    :value: Rating value (1-5 stars).
    :comment: The reviewer's comment.
    """

	RATING_CHOICES = (
	    (5, '5'),
	    (4, '4'),
	    (3, '3'),
	    (2, '2'),
	    (1, '1'),
	)
	book			= models.ForeignKey(Book, on_delete=models.CASCADE)
	user			= models.ForeignKey(settings.AUTH_USER_MODEL, default=1, on_delete=models.CASCADE)
	pub_date		= models.DateTimeField(auto_now_add=True, verbose_name='Publication Date') 
	comment 		= models.TextField(max_length=1024)
	value 			= models.IntegerField(choices=RATING_CHOICES, default=1)

	def __str__(self):
		return '{0}/{1} - {2}'.format(self.book.title, self.user.username, self.value)

	class Meta:
		verbose_name = "Book Review"
		verbose_name_plural = "Book Reviews"
		ordering = ['-pub_date']
```

### Review Admin

Insert the following inside `library/reviews/admin.py`:

```python
from django.contrib import admin

# Register your models here.
from .models import Review

admin.site.register(Review)
```

## Review Type

Inside `library/reviews`, create a new `gql` directory and three new files:

```bash
mkdir reviews/gql
touch reviews/gql/mutations.py
touch reviews/gql/schema.py
touch reviews/gql/serializers.py
touch reviews/gql/types.py
``` 

Insert the following inside `library/reviews/types.py`:

```python
from graphene_django import DjangoObjectType
from graphene import InputObjectType, String, Int, ID 
from reviews.models import Review

class ReviewType(DjangoObjectType):
	class Meta:
		model = Review
		convert_choices_to_enum = False

class ReviewInputType(InputObjectType):
	id          = ID()
	user        = ID()
	book        = ID()
	comment     = String()
	value       = Int()
```

We just made our first `InputObjectType`, which will allow us to specify input values for creating and updating reviews. We also created a `ReviewType` which inherits from the `Review` model.

## Review Serializer

Before moving on to mutations, we can create the Review serializer inside `library/reviews/gql/serializers.py`:

```python
from rest_framework import serializers
from reviews.models import Review

class ReviewSerializer(serializers.ModelSerializer):
    class Meta:
        model = Review
        fields = (
            'user',
            'comment',
            'value',
            'book'
        )
```

In order to create new reviews, we will make use of this serializer and it's fields. 

## Review Mutations

Let's move on to review mutations.

Insert the following inside `library/reviews/gql/mutations.py`:

```python
from graphene import Boolean, Field, ID, Mutation
from django.contrib.auth import get_user_model
from reviews.models import Review
from .types import ReviewType, ReviewInputType
from .serializers import ReviewSerializer

from graphql_jwt.decorators import login_required

User = get_user_model()

class ReviewCreate(Mutation):
    review = Field(ReviewType)
    class Arguments:
        input = ReviewInputType(required=True)

    @classmethod
    @login_required
    def mutate(cls, root, info, **data):
        user = info.context.user
        review_data = data.get('input')
        review_data['user'] = user.id 

        serializer = ReviewSerializer(data=review_data)
        serializer.is_valid(raise_exception=True)
        return ReviewCreate(review=serializer.save())

class ReviewUpdate(Mutation):
    review = Field(ReviewType)
    class Arguments:
        input = ReviewInputType(required=True)

    @classmethod
    @login_required
    def mutate(cls, root, info, **data):
        user = info.context.user
        review_data = data.get('input')
        review_data['user'] = user.id 

        review_instance = Review.objects.get(id=review_data.get('id'), user=review_data.get('user'))
        serializer = ReviewSerializer(review_instance, data=review_data)
        serializer.is_valid(raise_exception=True)
        return ReviewUpdate(review=serializer.save())

class ReviewDelete(Mutation):
    class Arguments:
        id = ID(required=True)
    ok = Boolean()

    @classmethod
    @login_required
    def mutate(cls, root, info, **data):
        user = info.context.user
        review = Review.objects.get(id=data.get('id'), user=user)
        review.delete()
        return ReviewDelete(ok=True)
```

In summary: 

1. We are using the `ReviewSerializer` to validate each mutation's input.
2. The `@login_required` decorator protects our reviews, and only allows logged in users to create, update or delete.
3. Deleting a review only requires an `ID`, so we pass this in as an argument.

## Review Schema

```python
from graphene import ObjectType, Schema
from .mutations import ReviewCreate, ReviewUpdate, ReviewDelete

class Mutation(ObjectType):
    create_review = ReviewCreate.Field()
    update_review = ReviewUpdate.Field()
    delete_review = ReviewDelete.Field()

schema = Schema(mutation=Mutation)
```

## Schema Updates

We just wired up all of our mutations, so now we are ready to return to the root schema inside `library/schema.py`:

```python
from graphene import Schema, ObjectType
import catalog.schema
import users.gql.schema
import reviews.gql.schema

class Query(catalog.schema.Query, users.gql.schema.Query, ObjectType):
	# This class will inherit from multiple Queries
	# as we begin to add more apps to our project
	pass

class Mutation(catalog.schema.Mutation, users.gql.schema.Mutation, reviews.gql.schema.Mutation, ObjectType):
	# This class will inherit from multiple Mutations
	# as we begin to add more apps to our project
	pass

schema = Schema(query=Query, mutation=Mutation)
```

## Test Queries and Mutations

Before testing the review mutations, be sure to update your models and migrations:

```bash
pipenv run python3 manage.py makemigrations
pipenv run python3 manage.py migrate
pipenv run python3 manage.py runserver
```

Now be sure to *login* with your admin user, at [http://127.0.0.1:8000/admin](http://127.0.0.1:8000/admin) before testing mutations at [http://127.0.0.1:8000/graphql](http://127.0.0.1:8000/graphql):

Create a review:

```
mutation {
  createReview(input:{book: 1, comment: "Loved this book!", value: 5}) {
    review {
      pubDate
      comment
      value
    }
  }
}
```

Update a review:

```
mutation {
  updateReview(input:{id: 1, book: 1, comment: "This book was great!", value: 4}) {
    review {
      pubDate
      comment
      value
    }
  }
}
```

Delete a review:

```
mutation {
  deleteReview(id: 1) {
    ok
  }
}
```

Creating and updating book reviews is now working as expected. **WELL DONE!!**

In the next section we prepare the library for [deployment](https://testdriven.io/blog/deploying-django-to-heroku-with-docker/). Stay tuned!