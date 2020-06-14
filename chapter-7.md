---
title: Django Image Uploads
seoTitle: Django Image Uploads
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django Image Uploads

### By the end of this lesson, developers will be able to:

- Create, upload images to Amazon S3 and `boto3`
- Accept uploaded file images using `graphene-file-upload`

### Introduction

Having the ability to upload book cover images would enhance the library catalog.

For our local library the "shelves" would feature cover images for browsing users.

So how about it? Let's uncover the steps to introduce a new client-facing feature.

## BookImage Model

We will create a new `BookImage` model to reflect these attributes:

- `book`
- `url`

Fairly simple, let's revisit `catalog/models.py`. 

Add the following anywhere below your book model:

```python
""" BookImage model (Book -< BookImage) """
class BookImage(models.Model):
	book = models.ForeignKey(Book, on_delete=models.CASCADE)
	url = models.CharField(max_length=255)

	def __str__(self):
		return self.book.name or ""

	class Meta:
		verbose_name = "Book Image"
		verbose_name_plural = "Book Image"
```

Here, we establish a new one-to-many relationship between `Book` and `BookImages`.

### Update Models, Migrations

Before testing with Amazon S3, let's apply the new model migrations.

```bash
pipenv run python3 manage.py makemigrations
pipenv run python3 manage.py migrate
```

## S3 Bucket Policy

Ready to build an image library with S3? Start by adding a new folder, `.aws` and then a new file inside, called `.aws/credentials`. 

Paste the following inside `.aws/credentials`:

```
[default]
aws_access_key_id=XXXX
aws_secret_access_key=XXXX
```

> NOTE: You must replace the "XXXX" values with your own S3 bucket keys.

Check out this [guide](https://git.generalassemb.ly/WDI-CC-LIBRARY/django-uploading-images#intro) to learn more about S3 image hosting.

Once you have the two keys in your `.aws/credentials`, proceed to accepting file uploads.

## Accepting Files

### Boto3 Install

```bash
pipenv install boto3
```

### Graphene File Upload

```bash
pipenv install graphene-file-upload
```

Add the following `BookImageMutation` to `catalog/schema.py`:

```python
from catalog.models import Author, Book, BookImage
from graphene_file_upload.scalars import Upload
import boto3
import uuid

# AWS S3 constants
S3_BASE_URL = 's3.amazonaws.com' 
BUCKET = 'libby-app'

# Once we add this node, we can query for book images
class BookImageNode(DjangoObjectType):
    class Meta:
        model = BookImage

# ...

class BookImageMutation(Mutation):
    class Arguments:
        file = Upload(required=True)
        id = ID(required=True)

    success = Boolean()

    def mutate(self, info, file, **data):
        # do something with your file
        # photo-file will be the "name" attribute on the <input type="file">
        photo_file = file
        book_id = data.get('id')
        if photo_file and book_id:
            s3 = boto3.client('s3')
            # 1. need a unique "key" for S3 / needs image file extension too
            key = uuid.uuid4().hex[:6] + photo_file.name[photo_file.name.rfind('.'):]
            # just in case something goes wrong
            try:
                s3.upload_fileobj(photo_file, BUCKET, key)
                # build the full url string
                # url has to be unique, 
                # otherwise we risk overwriting existing files.
                url = f"https://{BUCKET}.{S3_BASE_URL}/{key}"
                # 2. we can assign to book_id or book (if you have a book object)
                photo = BookImage(url=url, book_id=book_id)
                photo.save()
            except Exception as err:
                print('An error occurred uploading file to S3: %s' % err)
                return BookImageMutation(success=False)
        else: 
            print('Missing image or book ID')
            return BookImageMutation(success=False)

        return BookImageMutation(success=True)

class Mutation(ObjectType):
    book_mutation = BookMutation.Field()
    book_image_mutation = BookImageMutation.Field()

schema = Schema(query=Query, mutation=Mutation)
```

In this upload mutation, we introduce a new GraphQL type, called `Upload`. Then we called the S3 boto client's `upload_fileobj` function, which sends the file to our image bucket. 

In summary:

1. We give each file a unique `key` string, so we never overwrite existing images.

2. Once the upload succeeds, we create a new `BookImage` and assign it's public url.

As you may have guessed, this mutation will handle creating the file upload using a `multi-part` form data header. Testing locally requires a new request interface called [Altair](https://altair.sirmuel.design/#download).

## Testing Locally

### Altair Client

If you are on macOS, download the GraphQL client with `homebrew`:

```bash
brew cask install altair-graphql-client 
```

Otherwise, check out this link:

[Download Altair](https://altair.sirmuel.design/#download)

See the diagram below for an overview of the Altair client usage:

![Imgur](https://i.imgur.com/MjiIFv7.png)

### Authorization Header

In Altair, be sure to include your logged-in user's JSON web token with each requests' headers as follows:

```
Authorization: JWT <token>
```

While the `Authorization` header is not strictly required, you may need to include it if you added `PrivateGraphQLView` earlier in Chapter Five. 

### Upload Book Image

OK, fire up the client, and send a sample mutation:

```
mutation UploadBookImage($image: Upload!, $book_id:ID!) {
  bookImageMutation(file: $image, id: $book_id) {
    success
  }
}
```

Once you see this response: 

```json
{
  "data": {
    "bookImageMutation": {
      "success": true
    }
  }
}
```

Your first book image was successfully uploaded. 

## Book Image Serializer

We can create GET requests for book images, once they are serialized using a new `BookImageSerializer`:

```python
from catalog.models import Author, Book, BookImage

# Add anywhere above BookSerializer

class BookImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = BookImage
        fields = ['url']

class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(required=False)
    images = serializers.SerializerMethodField(read_only=True)

    def get_images(self, obj):
        images = BookImage.objects.filter(book_id=obj.id)
        serializer = BookImageSerializer(instance=images, many=True)
        return serializer.data

    # ...
```

Now you can query for book images at [http://127.0.0.1:8000/api/books/](http://127.0.0.1:8000/api/books/) or using this example:

```
query {
  books {
    edges {
      node {
        id
        title
        bookimageSet {
          url
        }
      }
    }
  }
}
```

**That's a wrap! WELL DONE!!**

In the next section we prepare the library for deployment. Stay tuned!

## References

- [General Assembly](https://git.generalassemb.ly/WDI-CC-LIBRARY/django-uploading-images#intro)
- [Adding Photos](https://git.generalassemb.ly/WDI-CC-LIBRARY/django-uploading-images#add-the-add_photo-view-function)
- [Accepting Files](https://github.com/lmcgartland/graphene-file-upload#graphene-file-upload)
- [Login Required](https://gist.github.com/jsmedmar/d846eee063fa23148f8a87313dd590a3)
