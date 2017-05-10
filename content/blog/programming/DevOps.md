+++
title = "Simply Said - DevOps"
date = "2017-04-28T13:50:46+02:00"
tags = ["theme"]
categories = ["Programming"]
banner = "images/code.png"
description = "Your Guide to Building a Web Crawler from scratch using Golang"
keywords = ["Programming","Go","Golang","Web","Crawler","web-crawler","recursive"]
draft = false
+++

[1]: https://apiblueprint.org/
[2]: https://github.com/danielgtaylor/aglio
[3]: https://expressjs.com/
[4]: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
# Behold Automation at it's Finest
Today's Topic will be all about Continous Integration! Let's Imagine this scenario, right now you are done with creating an awesome service that is capable of crawling the whole wide web or doing an e-commerce store. The problem now is that, you have to generate your own api blueprints (documentation), you have to manually deploy it to the cloud, make use of kubenetes settings. What is there was a way to automate all this these. By simply pushing to the Repository, all those manual labor will be done for you behind the scenes!

(Pst, we will also be using [kubernetes][4] for automate deployment!)
## Laying the Ground-Work
Of course, laying the ground work is no menial task. Before we begin, lets lay some "rules". The rules are that you should have already built a minimum of one service and is not bias(negative light) against google cloud platform services.

Process is as follows:
Local Repository --> Google Cloud Repository(Private) --> Build Triggers(Cloudbuild.yaml)  
```
// # Sample Cloudbuild.yaml
steps:
- name: 'gcr.io/cloud-builders/go'
  args: ['install', '.']
  env:  ['PROJECT_ROOT=$REPO_NAME']

- name: 'gcr.io/cloud-zen/kube-doc:latest'
  args: ['go', 'run', 'main.go']
  env: ['REPO_NAME=$REPO_NAME']

- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/$REPO_NAME:$REVISION_ID', '.']

- name: 'gcr.io/cloud-zen/kube-deploy:latest'
  args: ['set','image','deployments/$REPO_NAME', '$REPO_NAME=gcr.io/$PROJECT_ID/$REPO_NAME:$REVISION_ID']

images:
  - 'gcr.io/$PROJECT_ID/$REPO_NAME:$REVISION_ID'
```
Before pushing to the Google Cloud Repository, we first set up a trigger to look out for a file named Cloudbuild.yaml.  
</br>
<img src="/images/cloudbuild-trigger.png" class="img-responsive center-block" />

Inside the cloudbuild.yaml sample as we have shown above, it will execute a set of instructions and only upon successful execution will it be a successful build. In this sample, the service that we have built is based on go. We will first make use of the built-in docker image by cloudbuilder to run the commands:
```
go install
go test ./... (optional)
```
Followed by which, it will start to build the docker image based on the tags given to it and sends it to the Google Cloud Registry (GCR). The final command to execute would be to pull in a custom docker image that we have built to set the kubenetes system.
```
# Sample  Kube-Deploy Dockerfile
FROM ubuntu:16.04

ENV CLOUDSDK_PYTHON "/usr/bin/python2.7"
ENV PATH /root/google-cloud-sdk/bin:$PATH
ENV CLOUDSDK_PYTHON_SITEPACKAGES 1

# Install dependencies
RUN apt-get update && apt-get install -y curl && apt-get install -y python2.7

# Install gcloud
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
RUN apt-get update
RUN curl https://sdk.cloud.google.com | bash

# Authenticate gcloud
COPY /configs/gcloud /root/.config/gcloud
RUN ls -a /root/.config/gcloud

# Install kubectl
RUN /root/google-cloud-sdk/bin/gcloud components install kubectl

# Set and config cluster
RUN gcloud config set container/cluster cloudzen
RUN gcloud container clusters get-credentials cloudzen

# RUN gcloud auth application-default login
RUN gcloud container clusters describe cloudzen --zone asia-east1-a
RUN gcloud container clusters list --zone asia-east1-a

# Updating the cluster
RUN gcloud container get-server-config --zone=asia-east1-a
RUN gcloud container clusters get-credentials cloudzen --zone asia-east1-a

ENTRYPOINT ["kubectl"]
```
What kube-deploy does is that it installs python, gcloud and kubectl on an ubuntu base image with gcloud authenticated. This allows others services to pull this image to run the kubectl command to set the image in the pod(replica set). With that, you have successful built a integration that helps you build and pull docker images and at the set time set your image for proxy purposes and your services is updated with 0 downtime and without any manual labor (apart from running git push to Google Cloud Repository)
# Appendix
```
# Sample haproxy.cfg - for kubernetes
global
   log 127.0.0.1 local0
   daemon
#   maxconn 4000
#   debug

defaults
   log global
   mode http

   option http-server-close

   timeout connect 5s
   timeout client 30s
   timeout client-fin 30s
   timeout server 20s
   timeout tunnel 1h

   stats enable
   stats refresh 5s
   stats show-node
   stats uri  /stats/haproxy

frontend www
    bind *:80

    acl is_kube_aglio path_beg /doc
    use_backend kube-aglio-http if is_kube_aglio

    default_backend nomatch

backend nomatch
    errorfile 503 /usr/local/etc/haproxy/errors/404.http

backend kube-aglio-http
    balance roundrobin
    server api1 kube-aglio.default:80 check
```
</br>  
Ever developer knows that documentation is the key to everything and one of a nice tool that I've chanced upon is [API Blueprints][1]. in our current repository, we will have to have a .apib file which contains our documentation is .apib language. We would need some form of way to render it to html. One such tool we can make use of is [aglio][2] which helps us generate themes to come along with our html. All these works well but it comes with a price, what happens when we forget to run the command and push it to the cloud repository, also another problem is that anyone who wants to look at the documentation would have to clone the repository and that is not something that we want. So in the following steps, we would be building a function in go that would take care of passing the .apib in the repository into a google storage bucket.  
</br>
```
// Sample Kube-Doc Custom
func main() {
	bucketName   := "api-doc-build"
	repoName     := os.Getenv("REPO_NAME")
	APIBfilePath := os.Getenv("APIB_FILE_PATH")
	ctx          := context.Background()

	//Starts the client
	client, err := storage.NewClient(ctx)
	if err != nil {
		log.WithError(err).Fatal("Authentication Error!")
	}

	if repoName == "" {
		log.Fatal("'REPO_NAME' is required")
	}
	object := fmt.Sprintf("%s/doc.apib", repoName)

	doc, err := os.Open(APIBfilePath)
	if err != nil {
		log.Fatal("APIB File does not exist")
	}
	defer doc.Close()

	wc := client.Bucket(bucketName).Object(object).NewWriter(ctx)
	if _, err := io.Copy(wc, doc); err != nil {
		log.WithError(err).Fatal("Error Copying File")
	}
	if err := wc.Close(); err != nil {
		log.WithError(err).Fatal("Error Closing Client")
	}
	// Close the client when finished.
	if err := client.Close(); err != nil {
		log.WithError(err).Fatal("Error Closing Client")
	}
}
```
If you build a docker image with this custom go script, what it does is that it will throw the .apib file in the repository that is pulling the image into a bucket. The question now is that we would want to create a custom server that is capable of allowing the developers to use our api without needing to clone our repository or having to use the aglio command manually.

```
// Sample Dockerfile for Running Node Server
FROM node:boron

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Install app dependencies
COPY package.json /usr/src/app/
RUN npm install

# Bundle app source
COPY . /usr/src/app

EXPOSE 8080
CMD [ "npm", "start" ]
```


```
// Sample Index.js
'use strict';
const express = require('express');
const aglio = require('aglio');
const gcs = require('@google-cloud/storage')({
    projectId: '$projectID'
});
const bucket = gcs.bucket(process.env.GCS_BUCKET || 'api-doc-build');

// Configure express app
const app = express();
app.set('views', __dirname + '/views');
app.engine('html', require('ejs').renderFile);

app.get('/doc', (req, res) => {
    return bucket.getFiles((err, files) => {
        if (err !== null) {
            // files is an array of File objects.
            console.log(err);
            return res.status(500).send({error: 'error getting files'});
        }

        res.render('index.html', {
            docs: files.map((file) => {
                return file.id.split("%")[0];
            })
        });
    });
});

app.get('/doc/:repo', (req, res) => {
    let fileData = new Buffer('');
    const repo = req.param("repo");
    const remoteFile = bucket.file(`${repo}/${process.env.APIB_FILE_NAME || 'doc.apib'}`);

    // Validate
    if (repo === "") {
        return res.status(400).send({error: `Invalid repo name`});
    }

    // Download file from GCS bucket
    return remoteFile.createReadStream()
        .on('error', function(err) {
            console.log('Error:', err);
            return res.status(404).send({error: '404 File not found'});
        })
        .on('data', function(chunk) {
            fileData = Buffer.concat([fileData, chunk]);
        })
        .on('end', function() {
            // The file is fully downloaded.
            fileData = fileData.toString();
            if (fileData === undefined || fileData === "") {
                return res.status(500).send({error: `${process.env.APIB_FILE_NAME} document can not be empty`});
            }

            return aglio.render(fileData, {
                themeVariables: process.env.THEME || 'default'
            }, (err, html) => {
                if (err !== null) {
                    return res.status(500).send({error: err});
                }
                return res.send(html);
            });
        });
});

app.listen(process.env.HTTP_PORT || 8080);
console.log(`Running on :${process.env.HTTP_PORT || 8080}`);
```
</br>
This script above downloads the .apib file from the GCS bucket and later renders it on a [node express server][3]. With that, you have just been through the basic fundamentals of continous integration!
