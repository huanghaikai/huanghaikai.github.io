---
layout: post
title:  "My limited experience with Vagrant"
date:   2015-05-13 13:34:00
categories: recurse-center
comments_enabled: true
---

This week I wanted work on one of the issues from the [Community App][1] from [Recurse Center][2]. In order to be able to work on it, I had to pass through all the steps of installing a local version on my system.

The truth is it was a real pain to get the development environment working. I spent more than a full day focused to make it work on my local system. I could not make it work on my own. 

Without the help from [David Albert][3] I think I would have still be stuck.

## Setting up development systems in some cases is not trivial! ##

I am sure there are a lot of more complex scenarios than the one I worked yesterday. The [community app][1] requires to have installed (up to the date):

  - Ruby 2.2.2 via [RVM][8]
  - [Postgres][4]
  - [Leiningen][5]
  - [Elasticsearch][6]
  - [Redis][7]
  - Bundler, Foreman, etc.

In order to setup each of the parts, you need to at least know a bit of each. I never used Postgres, or Elasticsearch. So both of them was a pain for me. But I am sure it can be even harder than this scenario!

##I just wanted to work on a single issue ##

As a developer (or maye a future contributor) I want to read code, write some code, and test it! I should not be bothered by the installation steps.

The diagram below shows how I could describe yesterday: at each cycle in which the installation failed I just felt sader and sader. Maybe some people would quit in the middle of the process.

![DEIF](/public/dev-inst-frustrating.png "DEIF Frustration")

And I have to admit, I spent **a full day** just on the negative part of the cycle!

## Nobody else should spend a day just to set up a Development environment ##

This is where [Vagrant][9] comes in. It allows to create scripts for setting up Virtual Machines. With these scripts you can automate the steps required to set up a Development Environment.

The idea is that only one person should pass the pain of setting up the environment, record the steps to re-create the environment and create the scripts.

Future developers should benefit from this automation.

## Conclusion ##

If you have an OSS Project and you want more contributors, it should be **easy to install the development environment**. One way of achieving this is to use Vagrant.

Regarding the [Recurse Center Community App][1], I have just sent a Pull Request which adds Vagrant support. Check it out [here][10].

[1]: https://community.recurse.com
[2]: https://www.recurse.com
[3]: https://twitter.com/davidbalbert
[4]: http://www.postgresql.org/
[5]: http://leiningen.org/
[6]: https://www.elastic.co/
[7]: http://redis.io/
[8]: https://rvm.io/
[9]: https://www.vagrantup.com/
[10]: https://github.com/recursecenter/community/pull/308