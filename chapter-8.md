---
title: Django Deployment
seoTitle: Django Deployment
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## Django Deployment

### By the end of this lesson, developers will be able to:

- Use docker to test containerized deployments
- Create a continuous delivery pipeline with Gitlab CI and Docker

### Introduction

Launch time. Are you ready?

In this section, we will cover two approaches to deploying the library app:

1. Standard deployment with the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

2. Containerized deployment with [Docker](https://www.docker.com/).

We will also include references to an advanced deployment with [Gitlab CI](https://docs.gitlab.com/ee/ci/#overview).

## Docker

Download [Docker Desktop](https://www.docker.com/products/docker-desktop), and ensure it's running on your machine with the following command:

```bash
docker
```

If you see the list of docker commands, you're all set. We will utilize the `docker build` command to test our builds locally. 

Let's get started with a new `Dockerfile` in our root folder:

```
# pull official base image
FROM python:3.8.3-slim

# set work directory
WORKDIR /app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV DEBUG 1
ENV HEROKU_APP_NAME libby-app

# install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc

RUN apt-get install -y python3-pip

RUN pip install pipenv

# install dependencies
COPY Pipfile* /app
RUN cd /app && pipenv lock --requirements > requirements.txt
RUN pip install -r /app/requirements.txt

# copy project
COPY . .

# collect static files
RUN python manage.py collectstatic --noinput

# migrate database
RUN python manage.py migrate

# run gunicorn
CMD gunicorn library.wsgi:application --bind 0.0.0.0:$PORT
```

Here, I made a few decisions in the above file. Namely, to keep the build small, I selected the latest `python:3.8.3-slim` base image.

In summary:

1. Create a work directory at `/app`
2. Set environment variables
3. Install dependencies
4. Copy project content
5. Run python commands

You may notice that `pipenv` handles locking our dependencies at build time. We would like to omit the `Pipfile.lock` in this build process, so let's add a `.dockerignore` to omit files from the build process.

Add a new `.dockerignore` file in our root folder:

```
__pycache__
*.pyc
env/
db.sqlite3
Pipfile.lock
```

As of now, we can build and test the docker image locally with the following commands:

```bash
docker build -t web:latest .
docker run -d --name django-heroku -e "PORT=8765" -e "DEBUG=0" -p 8007:8765 web:latest
```

Verify [http://localhost:8007/api](http://localhost:8007/api) works as expected.

To stop and remove your local containers, run the following:

```bash
docker stop django-heroku
docker rm django-heroku
```

## Docker Deploy

With this approach, you can deploy pre-built Docker images to Heroku.

Log in to the Heroku Container Registry, to indicate to Heroku that we want to use the Container Runtime:

```bash
heroku container:login
```

Re-build the Docker image and tag it with the following format:

```
registry.heroku.com/<app>/<process-type>
```

Make sure to replace `<app>` with the name of the Heroku app that you just created and `<process-type>` with web since this will be for a web process.

For example:

```bash
docker build -t registry.heroku.com/libby-app/web .
```

Push the image to the registry:

```bash
docker push registry.heroku.com/libby-app/web
```

Release the image:

```bash
heroku container:release -a libby-app web
```

Go ahead and verify your live deployment. 

## Gitlab CI

After verifying your initial deployment succeeded, we can move on to a more advanced CI/CD pipeline. 

The purpose of Continuous Integration and Deployment, or CI/CD is to automate parts of the deployment process, such as releasing new commits to Heroku.

So our goals with gitlab are to automate the release process, and verify each deployment is building successfully.

For more information about the Gitlab CI/CD, visit [testdriven.io](https://testdriven.io/blog/deploying-django-to-heroku-with-docker/#gitlab-ci).

**That's a wrap! Congratulations!!**

Finally, we made it! Congratulations on completing the library application.

## Final Challenge

Try adding a frontend client to communicate with our library. Whether you decide to use a RESTful service, or GraphQL query interface, you should be able to catalog books, write reviews and upload book images.

## References
- [testdriven.io](https://testdriven.io/blog/deploying-django-to-heroku-with-docker/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)