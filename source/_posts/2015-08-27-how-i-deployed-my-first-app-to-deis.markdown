---
layout: post
title: "How I Deployed My First App To Deis"
date: 2015-08-27 06:30:43 -0600
comments: true
categories:
---

This post originally appeared on [The Deis Blog](http://deis.com/blog/2015/how-deployed-my-first-app-to-deis)

## Introduction

Have you ever felt the pain that comes when your app runs fine on development, but breaks terribly in production? Maybe your CI build has been red for days, but you haven't had time to figure out how the CI server is misconfigured?

With containers, you can easily rid yourself of such dependency woes. If the app runs in a container on one machine, it will most likely run in the same container on another.

Once you've bought into a container-based development workflow, the question soon arises: how can I get my production server to run my application in a container without the difficulty of having to provision a bare server with all of the other services, writing deploy tasks, and handling scaling issues on my own? In short, can I have a managed production environment that also supports containers?

The answer is yes. Using [Deis](http://deis.io/), an open source Platform as a Service, you can host and manage your Docker-based application using your own Amazon Web Services (AWS) servers, without the hassle of configuring a bare Linux server.

I recently deployed a simple Rails app to Deis, and took notes along the way. In this post, I'll share the steps I took to set up a [Deis Pro](https://try.deis.com/) account and deploy a new application.

## Setting Up AWS

Deis applications run on a cluster of servers tied to your AWS account. This allows you to control your settings with Amazon, and takes the middle man out of the billing process for server resources. Since I didn't have an AWS account, I had to create one. To do this, I followed [this guide from Engine Yard](https://support.cloud.engineyard.com/hc/en-us/articles/205501008-Before-You-Begin-Set-Up-Your-AWS-Account-and-User). Here's a quick run down of what I did.

First, I visisted [aws.amazon.com](http://aws.amazon.com) and clicked on "Create a Free Account". I walked through the signup process, entering my credentials and credit card information.

Once my account was all set up and I was logged in, I visited the "Identity & Access Management" page, then clicked "Groups" on the sidebar. I then created a group called "DeisAdminGroup".

Next, I created a user called "deis_user" under the "Users" tab. When I created it, Amazon asked me to download the user credentials in a CSV file, which I saved. Finally, I went back to "Groups", selected my group, clicked "Add Users to Group", and added "deis_user" to the group.

With these steps completed, I had my AWS user and security group set up. Next I turned my attention to setting up a Deis Pro account.

## Signing Up For Deis PRO

Before I could deploy my app to Deis Pro, I had to [sign up for an account](https://try.deis.com/).

I filled out the form, and a few minutes later I got a verification email. I clicked the link in the email, then entered my billing information. Deis then asked for my AWS credentials. I entered the information saved in the CSV I downloaded when I was setting up AWS, and I was ready to go.

## Creating the Cluster

Now that I was signed in to my Deis PRO account, I began to set up some server resources that I could deploy my application to. Here are the steps I followed to get them ready.

First, I visited my dashboard and clicked "Create Environment". Since I didn't need any special performance resources or customizations, I just used the default options for server size and memory allocations. I created an administrator username and password.

Next, I created three servers on AWS using the Deis PRO UI as part of the "create environment" process. For me, this process failed the first time, but when I tried again, the servers eventually came online. The last thing before leaving the Deis PRO site was to make note of my Deis endpoint name (it looks like deis.1ab2345.my.ey.io) so that I could use it to configure my app from the command line.

## Installing Deis Locally

After setting up my AWS and Deis accounts, I turned my attention to getting the [Deis command line client](http://docs.deis.io/en/latest/using_deis/install-client/) installed on my development machine. It was as simple as running this `curl` command:

```
curl -sSL http://deis.io/deis-cli/install.sh | sh -s 1.7.3
```

From there, I needed to put the `deis` executable into my `$PATH`. Although I could have used a symlink, I opted to move it directly to `usr/local/bin` with `mv deis /usr/local/bin/deis` instead. After running these two commands, I was able to run `deis -h`.

As a last step before setting up an app, I also needed to login to the server cluster I'd created and add the SSH key from my local machine to it. I logged into my Deis endpoint using `deis login deis.1ab2345.my.ey.io`. The endpoint name was the one I got from the Deis Pro website when I was setting up my resources. I found the username and password for the cluster in the Deis PRO UI. After I logged in, I added my local SSH key with `deis keys:add`.

With the Deis CLI set up and connected to my server cluster, I was ready to begin deploying my application.

## Deploying The App

When I was finishing bootcamp, I wrote an application called [Twelve Stepper](https://github.com/fluxusfrequency/12stepper) to help 12 Step Program participants interact with friends, find meetings, and work with the steps. Since it's a fairly simple application, I thought it would be a good one to use for my first Deis deploy.

I cloned it down from GitHub, bundled my gems and ran migrations as I would for any other Rails project. I also made a `Dockerfile` and got the app running in a container locally. Then I set up a Deis project by running `deis create 12stepper` from within the project folder.

After that, I tried to deploy using `git push deis master`, but I ran into an error: `tar: invalid tar magic`. After doing a little research, I found that I had forgotten to include a `Procfile`, so I created one. After adding it to `git` and pushing again, my app successfully deployed.

I ran `deis open`, and there was my app, up and running on the web!

## Wrapping Up

Getting Twelve Stepper set up on a Deis PRO cluster was pretty easy, all told. Most of my time was spent setting up accounts on AWS and Deis PRO and installing the Deis CLI. But these were one-time tasks. From here on out, deploying apps to Deis will be as easy as creating a new server cluster from my Deis PRO dashboard, then running `deis create <appname>` and `git push deis master` from my project folder.

I was surprised how easy it was to get my app up and running on a managed production environment using a container with Deis. If you're using a container-based development environment, I would definitely recommend checking Deis out as a hosting and deployment solution. Good luck!
