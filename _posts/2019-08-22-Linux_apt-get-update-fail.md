---
layout: post
title: "Solve apt-get 403 forbidden error in Linux Ubuntu"
description: "Solve apt-get 403 forbidden error in Linux Ubuntu"
crawlertitle: "Solve apt-get 403 forbidden error in Linux Ubuntu"
date: 2019-08-22 01:15:46 +0800
categories: Linux
tags: Linux
comments: true
---
I've been using Ubuntu 16.04 on Oracle VM VirtualBox these days, this morning when I run `$ sudo apt-get update` to update software, it breaks and I got this error instead:

![](https://i.imgur.com/ERImZ5i.png)

Seems there is an invalid repository that gives 403 error and causes the problem.

```bash
Err:20 http://ppa.launchpad.net/moka/stable/ubuntu xenial/main amd64 Packages
  403  Forbidden
```

We can also run `$ sudo apt-get update | grep "Failed"` to list failing repositories.

This error might lead other installation processes to fail, for example like installation of RVM, since `$ sudo apt-get update` doesn't work well.

## Solution

I decide to temporarily disable the PPA by comment out the source list for the quick fix.

### Check installed packages

  Use `grep` to get all enabled binary software sources showing together:  
  `$ grep -r --include '*.list' '^deb ' /etc/apt/sources.list /etc/apt/sources.list.d/`

  The output might look like this:

![The output of grep command](https://i.imgur.com/YvmNnWd.png)

  In my case, `moka-ubuntu-stable-xenial.list` is the trouble maker.

### Comment out the PPA source
  Comment out the content or delete `moka-ubuntu-stable-xenial.list` and `moka-ubuntu-stable-xenial.list.save`.

  In `/etc/apt/sources.list.d/moka-ubuntu-stable-xenial.list`:

  ```list
  # deb http://ppa.launchpad.net/moka/stable/ubuntu xenial main
  # deb-src http://ppa.launchpad.net/moka/stable/ubuntu xenial main
  ```

Now run `$ sudo apt-get update`, it should work properly, and **life goes on**.

My environment:  
![Description: Ubuntu 16.04.6 LTS; Release: 16.04; Codename: xenial](https://i.imgur.com/6jZEbNu.png)

## Reference
- [How can I get a list of all repositories and PPAs from the command line into an install script?](https://askubuntu.com/questions/148932/how-can-i-get-a-list-of-all-repositories-and-ppas-from-the-command-line-into-an)
- [Ruby RVM apt-get update error](https://stackoverflow.com/questions/23650992/ruby-rvm-apt-get-update-error)
