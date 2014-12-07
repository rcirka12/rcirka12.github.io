---
layout: post
title: "Deploying Play Framework to Elastic Beanstalk in Docker"
date: 2014-12-06 21:06:06 -0500
comments: true
categories: 
---

It is very easy to create a docker image of a scala play app using sbt, which can then be uploaded to AWS EC2 or Elastic Beanstalk

Using the command line:

```
 sbt docker:stage
```

This will generate a docker image in target/docker, with a file called 'Dockerfile' and a directory called 'files'. In Dockerfile, you will need to append the following line: EXPOSE 9000. You may also want to add any parameters to the line ENTRYPOINT. For example, I added a parameter for loading the production configuration file.

Dockerfile
```
FROM dockerfile/java
MAINTAINER John Doe <john.doe@gmail.com>
ADD files /
WORKDIR /opt/docker
RUN ["chown", "-R", "daemon", "."]
USER daemon
ENTRYPOINT ["bin/myapp", "-Dconfig.resource=application.prod.conf"]
CMD []
EXPOSE 9000
```

You will also need to create a file called Dockerrun.aws.json. It goes at the same level as Dockerfile.

Dockerrun.aws.json

```json
{
   "AWSEBDockerrunVersion": "1",
   "Ports": [{
       "ContainerPort": "9000"
   }]
}
```

Once those are created, you can zip the files and then upload it to elastic beanstalk. Caution, you must not zip the parent 'docker' directory.  Dockerfile, Dockerrun.aws.json, and files need to be in the root. For example, in terminal, navigate to target/docker and run

```
zip -r docker.zip *
```

### Automated Deployment with Jenkins

Best practice is to have a continuous integration server automatically deploy to a staging environment. Below is a bash script that can be used in Jenkins to automatically test, build and deploy to elastic beanstalk. It uses the git revision as the application version.

```bash
cd app

# Run unit tests
activator test

# Build the docker image
sbt docker:stage

# Copy Dockerfile and Dockerrun.aws.json to target/docker
cd ../scripts/deployment
cp Dockerfile Dockerrun.aws.json ../../app/target/docker

# Zip the files
cd ../../app/target/docker
zip -r $GIT_COMMIT.zip *

# -- AWS --
# Upload the image
aws s3api put-object --bucket my_bucket --key $GIT_COMMIT --body $GIT_COMMIT.zip

# Create a new application version
aws elasticbeanstalk create-application-version --application-name "myapp" --version-label $GIT_COMMIT --source-bundle S3Bucket=my_bucket,S3Key=$GIT_COMMIT

# Deploy the image
aws elasticbeanstalk update-environment --environment-name myapp-env --version-label $GIT_COMMIT
```