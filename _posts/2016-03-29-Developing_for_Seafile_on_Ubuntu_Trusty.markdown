---
layout: post
title:  "Developing for Seafile on Ubuntu Trusty"
date:   2016-03-29 23:02:32 +0200
categories: [seafile, dev, python]
---

## Introduction

After dealing with Seafile for quite a while at work I recently felt like getting involved more with the Seafile development.
Unfortunately this is harder, than it has to be (in my opinion).

For other interested parties and my future self I want to document how I got a working test environment on an Ubuntu trusty (14.04 LTS) machine.

I started at the Github page of [Seahub][github-seahub], which is the obvious starting point. Unfortunately the README is currently a little outdated.
I filed a [PR][seahub-README-pr] for that.

So at first, you should [install Seafile from source][seafile-install-source]. I used [this revision][seafile-install-source-commit] of the manual.  

For sake of completeness and because my way to get it running deviates from the official manual at some points, I will document the whole procedure.



## Installing Seafile from source

I assume you are installing your dev env at `/data/seafile`.

### Install dependencies

At first we install the needed build dependencies:

```
sudo apt install build-essential autoconf automake libtool libevent-dev libcurl4-openssl-dev libgtk2.0-dev uuid-dev intltool libsqlite3-dev valac libjansson-dev libqt4-dev cmake libfuse-dev re2c flex libarchive-dev
```

Now we install the packages needed for this manual:

```
sudo apt install git python-pip python-dev
```

### Create target directory structure

Now we create the target directory structure and go there:

```
sudo mkdir -p /data/seafile/seafile-server/src
cd /data/seafile/seafile-server/
## our user needs write permissions
sudo chown -R `whoami` .

cd src
```

### Install needed external libraries

Let's obtain and install the needed external libraries:

```
wget http://www.tildeslash.com/libzdb/dist/libzdb-2.12.tar.gz
tar xzf libzdb-2.12.tar.gz 
cd libzdb-2.12/
./configure 
make
sudo make install
cd ..

wget https://github.com/ellzey/libevhtp/archive/1.1.6.tar.gz
tar xzf 1.1.6.tar.gz 
cd libevhtp-1.1.6/
cmake -DEVHTP_DISABLE_SSL=ON -DEVHTP_BUILD_SHARED=ON
make
sudo make install
cd ..
```

### Install Seafile components

Because we want to get involved with development instead of downloading source tarballs, we simply clone the referenced components.

```
git clone https://github.com/haiwen/libsearpc/
cd libsearpc
./autogen.sh
./configure
make
sudo make install
cd ..

git clone https://github.com/haiwen/ccnet.git
cd ccnet
./autogen.sh
./configure --disable-client --enable-server   # `export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig` if libsearpc is not found
make
sudo make install
cd ..

cd seafile
git clone https://github.com/haiwen/seafile.git
./autogen.sh
./configure --disable-client --enable-server
make
sudo make install
cd ..

# refresh system library cache
sudo ldconfig
```

### Installing Seahub

Since I'm predominantly interested in improving the web interface, I forked this component ([Seahub][github-seahub]) on Github and cloned my fork.  

Installing webinterface Seahub:

```
cd /data/seafile/seafile-server
git clone git@github.com:<your account>/seahub.git
# Of course you could clone the upstream repo instead:
# git clone https://github.com/haiwen/seahub
```

### Installing Seahub dependencies

Instead of installing the dependencies manually, we use the provided requirements.txt files:

```
cd seahub
sudo pip install -r requirements.txt
sudo pip install -r test-requirements.txt

# for seafile-admin gunicorn is needed
# phantomjs is required for running tests
sudo apt install gunicorn phantomjs
```

For the `make dist` command in a later section we need requirejs, which can/should be installed using Nodejs package manager (npm):

```
sudo apt install npm/trusty
sudo npm install -g requirejs

# path fix for Ubuntu, see https://github.com/nodejs/node-v0.x-archive/issues/3911
# in later Ubuntu versions update-alternatives should work and would be cleaner
sudo ln -s /usr/bin/nodejs /usr/bin/node
```

## Prepare dev environment

### Setup environment

Set needed paths in setenv.sh:

```
cd /data/seafile/seafile-server/seahub
cp setenv.sh.template setenv.sh
vim setenv.sh
```

The contents of setenv.sh should look like this:

```
export CCNET_CONF_DIR=/data/seafile/ccnet
export SEAFILE_CENTRAL_CONF_DIR=/data/seafile/conf
export SEAFILE_CONF_DIR=/data/seafile/seafile-data
export PYTHONPATH=/usr/local/lib/python2.7/dist-packages:thirdpart:$PYTHONPATH
```

The `SEAFILE_CENTRAL_CONF_DIR` is necessary since Seafile 5.0 and at the time of writing was missing in setenv.sh.template.

### Create static files

Before you can run your dev version of Seahub, you need to generate some needed files (compressed js, css and so on):

```
# read permissons and path quirks make it hard to run this as non-root
# as this is just a test env vm, simply run as root
sudo -i
cd /data/seafile/seafile-server/seahub
source setenv.sh
make dist
exit
```

## Start Seafile

```
cd /data/seafile
sudo seafile-admin start
```

## Create admin account

Before you can login, you need to create an admin account:

```
cd /data/seafile/seafile-server/seahub
sudo python tools/seahub-admin.py
```

The ccnet directory `seahub-admin.py` asks for is `/data/seafile/ccnet`.

## Prepare testing environment

If you want to develop / improve some featues of Seahub, you should write appropriate tests and check if your changes broke some tests.  
The script responsible for managing and running the tests is `seahubtests.sh`.

To make seahubtests.sh work, I had to patch it:
{% gist egroeper/9a1c2414607a9ac70674a783f0ca8995 %}

Now we can init the test environment:

```
cd /data/seafile/seafile-server/seahub
# setup tests
sudo tests/seahubtests.sh init
```

<del>I didn't manage some (most?) of the api tests to pass. My guess is, that I'm running into throttling issues although the `local_settings.py` created by `seahubtests.sh init` should fix that.  
But most tests should pass now.</del>

<del>My results was:
```
42 failed, 367 passed, 1 skipped, 6 xfailed in 271.62 seconds
```
</del>

Of course you need to restart Seafile (especially seahub) for the changes to take effect. After that you can run the tests for the first time:

```
# restart Seafile
cd /data/seafile
sudo seafile-admin stop
sudo seafile-admin start

# run tests
cd /data/seafile/seafile-server/seahub
sudo tests/seahubtests.sh test
```

You are ready to code. Have fun!



**Update 2016-03-31:** Seems like my problems with the api tests were caused by omitting the restart of Seahub. My bad. Should have known that.

## Appendix

### Starting Seahub manually through runseahub.sh

Instead of using seafile-admin tool to start and stop all the components of Seafile, you could start and stop Seahub using the provided `runseahub.sh.template`, but then you would still need to start the other components separately.

If you run your test env in a VM and not on your local machine, you need to adjust `runseahub.sh.template`.
By default your testserver would only listen on localhost. But you can append the ip and port to listen on:

```
cp runseahub.sh.template runseahub.sh
vim runseahub.sh

# runserver line should look like this:
# ./manage.py runserver 0.0.0.0:8000
```

[github-seahub]: https://github.com/haiwen/seahub
[seahub-README-pr]: https://github.com/haiwen/seahub/pull/1086
[seafile-install-source]: http://manual.seafile.com/build_seafile/server.html
[seafile-install-source-commit]: https://github.com/haiwen/seafile-docs/blob/928b9d61260aa9cab12ff366e3b806bf5b782b1f/build_seafile/server.md
