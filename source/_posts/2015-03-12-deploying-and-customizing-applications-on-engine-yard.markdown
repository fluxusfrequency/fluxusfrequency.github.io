---
layout: post
title: "Deploying and Customizing Applications on Engine Yard"
date: 2015-03-12 06:29:30 -0600
comments: true
categories:
---

This post originally appeared on [Engine Yard](https://blog.engineyard.com/2015/deploying-and-customizing-applications).

## Introduction

I've tried a lot of different Platform as a Service (PaaS) providers for hosting my applications. Some of them make it super-easy to get everything running on the server, but the magic gets in the way when you need to customize things.

Some platforms give you full control, but it can be time-consuming to get all of your dependencies properly set up. It would be nice to have some of the boilerplate taken care of, while still retaining full control of my server environment.

If you haven’t deployed an application to Engine Yard (EY), you should give it a try. You’ll be pleasantly surprised to find that this is exactly the kind of service offered. It’s a breeze to get Redis, cron, and any other tools you need installed. You also get root access to your server and can SSH in just as you would with a bare server.

My favorite feature has always been the ability to push custom Chef recipes to my server, making it super-easy to tweak the server as needed without having to spend a lot of time downloading Ubuntu packages and managing user permissions.

There is a little bit of a learning curve though, so I decided to deploy and customise a new app on Engine Yard so that I could document the process and help first-timers get up and running with a minimum of fuss.

## Setting Up An Engine Yard Environment

I started out with a production application in a Git repository that was ready to go. It just needed a server to run on.

I went to Engine Yard and signed up for [a free trial account](https://www.engineyard.com/trial).

Once I was done filling out my contact and billing information, I created my first "application" resource on Engine Yard Cloud. Here are the steps I followed to get it running from there.

First, I created a "production" environment for my application and configured it. I was given four choices of server beefiness: single instance, staging, production, or custom. I went with a production box, and added Phusion Passenger and PostgreSQL to the stack. Since I was deploying a Rails app, I also added Ruby 2.2.0 and set up my migration command. I was happy to see that EY would backup by my database and take a server snapshot on a recurring schedule. I opted in for that service.

While the server was being provisioned, there were a few access-related tasks I had to take care of as well. First, I [added the SSH keys](https://support.cloud.engineyard.com/entries/21016533-Connect-to-Your-Instance-Using-SSH) from my development machine to my production environment. To do so, I visited the EY Cloud dashboard, then clicked on *Tools*, then *SSH Keys* and pasted my key into the text area, then hit the big *Apply* button on my app's "production" environment page.

I also had to add an SSH key EY provided to my GitHub account. This allowed EY to grab my code and push it to the server directly.

A few minutes later, the server and my credentials were all set up, and I was ready to deploy. Next, I pressed *Deploy*.  Unfortunately, there was a problem with my deploy, so I decided to dig into it from the command line...

## Using the `engineyard` Gem

### Configuring and Deploying

It turned out I’d forgotten to add a `config/ey.yml` file to my Rails project. This file is used to customize each of the Engine Yard environments the app is being deployed to. To add one, it’s easiest use the [engineyard gem](https://github.com/engineyard/engineyard).

To install the gem globally, I ran `gem install engineyard` on my local machine. Then I initialized an EY configuration file using `ey init`. I checked out the `config/ey.yml` file it generated. Everything looked good, so I committed and pushed it up to GitHub.

This time, I deployed using `ey deploy`, and it worked like a charm. Success!

### Logging In and Out

- `ey login`
- `ey logout`
- `ey whoami`

### Custom Deploys

- `ey status` shows the status of your most recent deploy
- `ey timeout-deploy` marks the current deploy as failed and begins a new deploy
- `ey rollback` revert to a previous deployment

### Environments

- `ey environments` shows the environments for this app (pass `--all` to see all environments for all apps)
- `ey servers` shows all the servers for an environment (if you have multiple)
- `ey rebuild` reruns the configuration bootstrap process, useful for security patches and upgrades
- `ey restart` restarts the servers

### Debugging

- `ey logs` shows the logs
- `ey web disable`/`enable` toggles a maintenance page
- `ey ssh` lets you SSH in
- `ey launch` launches app in a browser window

### Customizing The Server Environment

- `ey recipes upload` adds Chef recipes from your dev machine and the remote server
- `ey recipes download` syncs Chef recipes from the remote server to your dev machine
- `ey recipes apply` trigger a Chef run

These last few commands come in really handy when you want to customize your server setup. Let's take a deeper look at how to upload custom Chef recipes to an application environment.

## Chef Recipes

Engine Yard uses Chef under the hood to make your deploys quick and easy. There's a default set of recipes that get run every time you deploy.

After the default recipes run, Engine Yard runs any custom recipes that you've added to your environment. Since there are [so many Chef recipes available](https://github.com/opscode-cookbooks), getting dependencies set up is pretty straightforward.

Your Chef recipes will run whenever you create a new instance, add an instance to a cluster, run `ey recipes apply`, or trigger a Chef run with from the Cloud Dashboard with the *Upgrade* or *Apply* buttons.

### Getting Set Up

To add recipes to your application, you'll need to fork the [Engine Yard Cloud Recipes repo](https://github.com/engineyard/ey-cloud-recipes). Then, clone your fork down to your development machine, in a different directory than your application.

### Default Recipes

The Engine Yard Cloud Recipes repo comes with comes with cookbooks for most of the things you would ever need: Sidekiq, Redis, Solr, Elasticsearch, cron, PostgreSQL Extensions, and much more.

Here's what I did to add Redis to my project.

1) Opened `/cookbooks` and found the subdirectory I wanted (`/redis`)
2) Uncommented `include_recipe redis` in `cookbooks/main/recipes/default.rb`
3) Saved the file, committed it, and push to my forked repo
4) Uploaded the recipes to my app with `ey recipes upload -e production`
5) Applied the recipes to my app with `ey recipes apply -e production`

I took a look at my Engine Yard dashboard, and a few short moments later, Redis was running on my server!

### Custom Recipes

I wanted to add HTTP Basic Auth to my server, but it wasn't one of the recipes in the repo, so I wrote my own recipe for it.

Here's how I did it.

First, I opened up my `ey-cloud-recipes` fork repo and ran `rake new_cookbook COOKBOOK=httpauth`. This generated a bunch of files under `cookbooks/httpauth/`. Then I edited `cookbooks/httpauth/recipes/default.rb` like this:

```
sysadmins = search(:users, 'groups:sysadmin')

template "/etc/nginx/htpasswd.users" do
  source "nginx/htpasswd.users.erb"
  owner node['staging']['nginx']['user']
  group node['staging']['nginx']['user']
  mode "0640"
  variables(
    :sysadmins => sysadmins
  )
  notifies :restart, "service[nginx]", :delayed
end
```

With the `httpauth` recipe written, I next created a `htpasswd.users.erb` under the `cookbooks/httpauth/templates/default/nginx directory, and put this code in it:

```
<% @sysadmins.each do |sa| -%>
  <%= sa["id"] %>:<%= sa["htpasswd"] %>
<% end -%>
```

With the template in place, I added the recipe to `cookbooks/main/recipes/default.rb` (my main cookbook) by adding this line:

```
include_recipe "httpauth"
```

Finally, I checked my syntax with `rake test` (all good), committed my changes, and pushed to my fork. With the recipe ready, all that was left was to upload and apply it to my application with the following:

```
ey recipes upload -e production
ey recipes apply -e production
```

The recipe was successfully added to my server in the `/etc/chef-custom` directory. I know this because I logged in and took a look around.

How did I do that? I'm glad you asked.

## Remote Access with SSH

If you ever need to confirm that your Chef recipes are configuring the server the way you expected, or need to access your server directly with root access for any other reason, you can use SSH to get a remote terminal.

There are three ways to do this:

1) Run `ssh username@123.123.123.123`, the old-fashioned way (you can find your server's IP address in the Engine Yard dashboard).
2) Click on the SSH link in your application dashboard on EY instead
3) Run `ey ssh` from the application directory on your dev machine

When you login to your server, some helpful information about your app's server environment is displayed:

```
Applications:
myapplication:
cd /data/myapplication/current # go to the application root folder
tail -f /data/myapplication/current/log/production.log # production logs
cat /data/nginx/servers/myapplication.conf # current nginx conf

SQL database:
cd /db # your attached DB volume

PostgreSQL:
tail -f /db/postgresql/$1.$2/data/pg_log/* # logs
pg_top -dmyapplication

Inspect node data passed to chef cookbooks:
sudo gem install jazor
sudo jazor /etc/chef/dna.json
sudo jazor /etc/chef/dna.json 'applications.map {|app, data| [app, data.keys]}'
```

Pretty cool. It's nice to have access like this if you need it, without being responsible for configuring (and re-configuring) the entire machine by hand.

## Conclusion

If you're anything like me, you drag your feet about trying new things when it comes to sysops. Perhaps you've felt the pain of trying to take a server from vanilla Ubuntu to a custom build with cron, Redis, Elasticsearch and a bunch of other packages—carefully balancing everything so that it doesn't fall apart. Many of us have also experienced getting stuck when using a full-service PaaS that isn't working the way we expect, and not being able to customise things. Experiences like this make it hard for me to get excited about setting up servers, so I typically avoid it when I can.

That said, Engine Yard makes this sort of work a breeze. Their balance between automation and control gives you the best of both worlds. Getting up and running takes a little bit of learning, but the [docs](https://support.cloud.engineyard.com/home)  are super helpful and  the support team is very responsive if you ever have any questions or need a hand.

If you haven't given Engine Yard a try, why not give it a go?

P.S. What kind of custom Chef recipes are you using on your servers? I know that business requirements can lead to some pretty gnarly setups. Tell me about it via the comments.

