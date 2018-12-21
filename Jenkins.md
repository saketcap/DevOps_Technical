# Docker and Jenkins CI

Combining Jenkins with Docker provides some powerful opportunities when it comes to building and running pipelines.

In this workshop we will try to get Jenkins running in a docker image, give this image the ability to spin up its own containers as called from the pipeline, and demonstrate how such a pipeline might look.

For this part of the workshop, we are going to try to apply some of the things we've learned in the docker and git tutorials and apply them to an assignment.

## Objective

We want to create a pipeline that runs Jenkins 2.0 or greater (for pipeline support), and is able to launch other Docker containers from the pipeline.

We want to set up a pipeline to build a java application in a docker container and then perhaps run some tests/scans over the code to see if it meets out standards.

## Approach

We will split into 3 groups of no more than 3 persons to a group. Each group will pick up one of the following tasks/challenges.

## Install Jenkins with Docker

The idea is to take a base image of Jenkins from Docker hub and configure it to run with access to the Docker engine. There are a few different approaches out there making the distinction, for instance, between Docker out of Docker (DooD) and Docker in Docker (DinD).

It is up to you to decide which approach (or perhaps you'll develop your own one) you take.

```
Tip: You will need to set up a volume for Jenkins to store things like its settings and its workspace.
```

## Find a java app on github and build it

Find a not too complex java application on github that we can build and run in our pipeline. For the time being, pull your own docker containers for java, maven, gradle, etc., depending on what you want to use, to a local environment and compile and test the application to be sure it works.

Ideally, this application has some automated tests with it that let us run them to see if everything is working alright.

The challenge is to build and test them using docker images. Which build tool you use if up to you (Maven, Gradle, etc.)

## Look into Jenkins Pipeline Scipting

Look into the Pipeline Scripting language of Jenkins. First decide what style you prefer to use Declarative or Scripted. Be prepared to motivate your answer. :)

Set up a basic pipeline script that is checked into git (locally) and that is pulled into your Jenkins Docker container and run.

For the moment, just place echo messages at the various stages to prove the stage ran and worked.

What is required to run your stages in docker containers?

## Putting It All Together

When each of the groups is satisfied with their solutions, we will come together and try to blend the various solutions into one working pipeline.