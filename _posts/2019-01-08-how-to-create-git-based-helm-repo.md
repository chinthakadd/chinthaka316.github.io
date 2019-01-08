---
layout: post
title: How to Create a Helm Repo in Git?
---
If you know `Helm`, you know helm chart repositories. It is like maven repositories, instead these repositories hold onto the
different charts that can be used to install different tools and products into a kubernetes environment. If you are planning to use Helm in your organization to manage your own deployments, lets say your micro-services, then most probably you have come across the need for a chart repository of your own.

According Helm:

"
A chart repository is an HTTP server that houses an index.yaml file and optionally some packaged charts. When you're ready to share your charts, the preferred way to do so is by uploading them to a chart repository.
"

Source: https://github.com/helm/helm/blob/master/docs/chart_repository.md

So all you simply need is a HTTP Server. The simplest of all the options that comes to mind is GIT in this case.
If you organization is using GitHub, GitLab, BitBucket or any Git variation, you can easily create a helm repository.

The following are the set of steps that I followed.

Step 1: Your chart has to be packaged as a Tar File.

```
$ helm package test-chart
Successfully packaged chart and saved it to: /Users/admin/Development/Research/Helm/helm-repository/test-chart-0.0.1.tgz
```

Step 2: Create a chart repository

```
$ mkdir test-charts
$ # Copy the charts and tar file
$ mv test-chart test-chart-0.0.1.tgz ./test-charts
```

Step 3: Create a index.yaml file for the chart_repository
```
$ cd test-charts
$ helm repo index .
```

Index.yaml file should look like the following


```
apiVersion: v1
entries:
  test-chart:
  - apiVersion: v1
    created: 2019-01-08T00:01:10.623604673-05:00
    description: Simple Micro-service Chart
    digest: 748594dc2815a12ea3a2b4860b731ff0f3d7cfce0102538214448dd962b8ac08
    name: test-chart
    urls:
    - test-chart-0.0.1.tgz
    version: 0.0.1
generated: 2019-01-08T00:01:10.622439226-05:00
```

As a pre-requisite, you need to have a Git Repo Ready.
Mine is https://github.com/chinthakadd/simple-helm-repo

Step 4: Initialize your repo and Push the chart repository to the remote Git repository

```
$ git init
$ git add .
$ git commit -m "init chart repo"
$ git remote add origin git@github.com:chinthakadd/simple-helm-repo
$ git push -u origin master
```

Step 5: Add the repo to Helm

```
$ helm repo add simple-helm-repo {Git Raw URL}
```

NOTE: Git URL is the Raw Git URL.
Ex: https://raw.githubusercontent.com/chinthakadd/simple-helm-repo/master for my chart repo.

Step 6: Check if you can fetch a Helm Chart from the Repository

```
$ helm fetch simple-helm-repo/test-chart --untar
```

A simple asciicinema on how I achieved this.

{% asciicast JEeKhFymVSg0Kh2bI8gYyfzcS %}

<!-- [![asciicast](https://asciinema.org/a/JEeKhFymVSg0Kh2bI8gYyfzcS.svg)](https://asciinema.org/a/JEeKhFymVSg0Kh2bI8gYyfzcS) -->
