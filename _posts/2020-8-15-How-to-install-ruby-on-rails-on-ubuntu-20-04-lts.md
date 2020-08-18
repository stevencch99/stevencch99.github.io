---
layout: post
title: "How to install Ruby on Rails on Ubuntu 20.04 LTS"
description: "Installation guide for installation of Ruby on Rails on Ubuntu 20.04 LTS"
crawlertitle: "How to install Ruby on Rails on Ubuntu 20.04 LTS"
date: 2020-08-15 23:57:06 +0800
categories: Ruby Rails Ubuntu
tags: Ruby Rails Ubuntu
comments: true
visible: true
---
- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Install RVM (Ruby Version Manager) for Ruby on Rails

RVM is a command-line tool which allows you to easily install, manage, and work with multiple ruby environments from interpreters to sets of gems.

> Ref: [RVM Official installation document for ubuntu](https://github.com/rvm/ubuntu_rvm)

### Pre-requisites
You need `software-properties-common` installed in order to add PPA repositories.
```
sudo apt-get install software-properties-common
```

1. Add the PPA and install the package
  ```
  sudo apt-add-repository -y ppa:rael-gc/rvm
  sudo apt-get update
  sudo apt-get install rvm
  ```

2. Change your terminal window  
    In order to always load rvm, enable the opteion of Gnome Terminal to "Run cmmand as a login shell".
    ![](https://i.imgur.com/Lu2PkVe.png)

3. Install Ruby & Rails

    ```
    # List all known version of ruby
    rvm list known

    # Install the latest version
    rvm install ruby

    # Show installed ruby versions
    rvm list

    # To use and set a specific version as default
    rvm --default use 2.7.0
    ```

### Troubleshooting

- gpg key failed with connection timeout.
  ```
  curl -sSL https://rvm.io/mpapis.asc | gpg --import -
  curl -sSL https://rvm.io/pkuczynski.asc | gpg --import -
  ```

- Cannot add PPA under proxy:
  ```
  sudo -E add-apt-repository -y ppa:rael-gc/rvm
  ```

## Install Rails

```
gem install rails

# Check the installation of Rails
rails -v # should return Rails x.x.x.x
```

### Install Webpack for Rails

You need to compile assets (images, CSS, JavaScript, etc.) in production environment.

"Starting with Rails 6, Webpacker is the default JavaScript compiler. It means that all the JavaScript code will be handled by Webpacker instead of the old assets pipeline aka Sprockets."
> Ref: https://prathamesh.tech/2019/08/26/understanding-webpacker-in-rails-6/


According to [Webpack](https://webpack.js.org/) official site:  
> webpack is a module bundler. Its main purpose is to bundle JavaScript files for usage in a browser.

- Install NVM ([Node Version Manager](https://github.com/nvm-sh/nvm))  
  We need **Node.js** to run webpack, and it is recommended to use `nvm` to manage node versions like `rvm`.
  To install or update nvm, you should run the install script. To do that, you may either download and run the script manually, or use the following cURL or Wget command:

  ```
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
  ```
  - Check installation: `nvm -v`
  - List available versions of Node.js: `nvm ls-remote`
  - Install Node.js:
    ```
    nvm install node # "node" is an alias for the latest version
    ```

- Install Yarn - Package Manager by NPM (Node Package Manager):
  ```
  npm install -g yarn
  ```

Finally, it's able to install webpack in a Rails project:
```
rails webpacker:install

# assets precompile
rails assets:precompile
```

## References
- [RVM Official installation document for ubuntu](https://github.com/rvm/ubuntu_rvm)
- [Webpack official](https://webpack.js.org/)