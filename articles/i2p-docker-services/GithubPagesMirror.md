Mirroring an existing Github Pages site using a Static eepSite
==============================================================

This is kind of a softball tutorial, it's almost the same as the **[Previous tutorial](BasicStaticeepSite.md):**
but it's intended specifically to allow you to mirror a Github Pages based site
using Jekyll to i2p using i2pd and Tor. Of course, this means that the
eepSite is by definition linked to your identity on Github, but this might be
desirable, for instance for i2p applications that collaborate primarily by way
of Github, but who wish to have a presence on i2p as well. Probably other cases
where long-term pseudonymity is desirable.

So, just to be clear
--------------------

  * *For this tutorial to be unlinkable to your real identity, your github*
   *must be unlinkable to your real identity*.
  * *Just because anonymity is a feature doesn't make non-anonymous or*
   *pseudonymous replication of information atop anonymous infrastructure is a*
   *less valid goal. There are plenty of cool things you can do with eepSites*
   *that do not require anonymity in all parts of the world.*

Now with that out of the way, let's get down to mirroring a Github Pages site,
for example this exact wiki, to your own i2p eepSite.

Preparation
-----------

To get started, you'll need an i2pd docker container with an http server tunnel
prepared. For convenience's sake, you can just use the one from the previous
tutorial. First, make sure you've created your network and specified a subnet,
as in the first tutorial:

```
docker network create --subnet 172.81.81.0/29 eepsite
```

And alter the tunnels.conf in your tunnels.conf to look like this:

```
[DARKHTTPD]
type = http
host = eepsite-darkhttpd
port = 8080
inbound.length = 3
outbound.length = 3
keys = darkhttpd.dat

[GITHUB]
type = http
host = eepsite-github
port = 8090
inbound.length = 3
outbound.length = 3
keys = github.dat
```

Once you've done that, you can "docker build" the new container and your
original, darkhttpd tunnel will continue to exist at the same destination but a
new unique destination will be generated for your forthcoming Github mirror.
From the perspective of someone browsing the i2p network, and the other routers
in the i2p network, there will be no good reason to believe that either of those
tunnels are running on the same i2p router. If you no longer want the original
darkhttpd tunnel, then you can just delete the [DARKHTTPD] section from your
tunnels.conf.

Obfuscating the IP address of your container host
-------------------------------------------------

In order to fetch your repositories from git pseudonymously, or in order to
obfuscate your location as a git user, you will need to ensure that git is
configured to fetch clearnet resources via Tor. In order to do this, we'll add a
container running Tor to our docker network, with an exposed SocksPort.
Additionally, we will do no actual building or generation of the Github Pages
mirror until the containers are running and present on the network. In order to
safely host any site that requests resources from the clearnet, you will need to
do a similar procedure to route it through Tor. For git, this procedure is
simple to do so it makes a good example.

For the purposes of this demonstration, we're going to assume that the pertinent
adversaries do not have sufficient power to link your fetching of packages via
the rubygems package manager to your eepsite activity. If that's an issue for
you, I kinda hope you didn't need this tutorial. It also assumes that you are
not concerned about separating contextual identities *because* you are only
going to use this Tor SocksProxy to fetch git repositories associated with one
account anyway. So here's our Tor container Dockerfile:

### Tor container Dockerfile

```Dockerfile
FROM alpine:3.7
ARG TOR_CONTROL_HOST=172.81.81.6
ARG TOR_CONTROL_PORT=9151
ARG TOR_SOCKS_HOST=172.81.81.6
ARG TOR_SOCKS_PORT=9150
RUN apk update && apk add tor
COPY torrc /etc/tor/torrc
RUN sed -i "s|172.81.81.6|$TOR_CONTROL_HOST|g" /etc/tor/torrc
RUN sed -i "s|9151|$TOR_CONTROL_PORT|g" /etc/tor/torrc
RUN sed -i "s|172.81.81.6|$TOR_SOCKS_HOST|g" /etc/tor/torrc
RUN sed -i "s|9150|$TOR_SOCKS_PORT|g" /etc/tor/torrc
RUN mkdir -p /var/lib/tor
RUN chown -R tor /var/lib/tor
RUN chmod -R 2700 /var/lib/tor
RUN chmod -R o+rw /var/lib/tor
EXPOSE $TOR_CONTROL_PORT
EXPOSE $TOR_SOCKS_PORT
USER tor
CMD tor -f /etc/tor/torrc
```

Of course, you'll need a minimal .torrc to configure your Tor service with.

### minimal torrc

```
SOCKSPort 172.81.81.6:9150
SOCKSPolicy accept 172.81.81.0/24
DataDirectory /var/lib/tor
ControlPort 172.81.81.6:9151
CookieAuthentication 1
```

Keep it in your docker build context. If you have the resources, consider making
it a relay or hosting a Snowflake service.

### Building and Running the Tor container

Finally, build the Tor container using the following command:

```
docker build --force-rm \
	--build-arg TOR_SOCKS_PORT=9150 \
	--build-arg TOR_SOCKS_HOST=172.81.81.6 \
	--build-arg TOR_CONTROL_PORT=9151 \
	--build-arg TOR_CONTROL_HOST=172.81.81.6 \
	--network eepsite \
	-f Dockerfile.torhost -t eyedeekay/tor-host .
```

And run it using the following command:

```
docker run --rm -i -t -d \
	--net tbb \
	--name eepsite-tor \
	--network eepsite \
	--network-alias eepsite-tor \
	--hostname eepsite-tor \
	--link eepsite-github \
	--expose 9150 \
	--ip 172.81.81.6 \
	eyedeekay/tor-host; true
```

Mirroring your github page
--------------------------

Once your Tor service is up, you're ready to start mirroring your github page.
To do this, while only fetching git repositories over Tor, you can use an Alpine
Linux container with ruby, ruby-bundler, and some libraries. We're also going to
configure our tools, git and bundle, but not use them to fetch any resources
yet.

### Github Pages Mirror Dockerfile

```Dockerfile
FROM alpine:3.7
ARG PAGES_REPO_NWO=j-tt/r-i2p-wiki
ARG proxy=socks5://172.81.81.6:9150
ENV PAGES_REPO_NWO=$PAGES_REPO_NWO JEKYLL_ENV=production proxy=$proxy
RUN apk update && apk add git ruby ruby-dev make gcc g++ musl musl-dev ruby-rdoc ruby-irb ruby-xmlrpc libxml2 zlib zlib-dev markdown ruby-bundler
RUN adduser -h /var/www/ -D user
RUN git config --global http.proxy $proxy
WORKDIR /var/www/
RUN chown -R user .
USER user
RUN bundle config --local path /var/www/.bundle
COPY loop.sh /usr/bin/loop.sh
CMD loop.sh
```

Finally, we need to create a launcher script. This script will create the site
folder, populate it, guarantee the contents of the Gemfile and serve the site.

### Github Pages Mirror Congigurator

```Shell
#! /usr/bin/env /bin/sh
git config --global http.proxy $proxy
git clone https://github.com/$PAGES_REPO_NWO /var/www/site
cd /var/www/site || exit
touch Gemfile
grep "source 'https://rubygems.org'" Gemfile || \
    echo "source 'https://rubygems.org'" | tee -a Gemfile
grep "gem 'github-pages', group: :jekyll_plugins" Gemfile || \
    echo "gem 'github-pages', group: :jekyll_plugins" | tee -a Gemfile
grep "gem '$theme', group: :jekyll_plugins" Gemfile || \
    echo "gem '$theme', group: :jekyll_plugins" | tee -a Gemfile
grep "baseurl:" _config.yaml && \
    grep -v "baseurl:" _config.yaml > _config.yaml.bak && \
    mv _config.yaml.bak _config.yaml
grep "url:" _config.yaml && \
    grep -v "url:" _config.yaml > _config.yaml.bak && \
    mv _config.yaml.bak _config.yaml
grep "url:" _config.yaml || \
    echo "url: \"\"" | tee -a _config.yaml
bundle install
bundle exec jekyll serve --port 8090 --host 0.0.0.0
```

Now you've got a container ready to build and run, which will host a github
page mirror on i2p, but only fetch the repo from github over Tor. Every time you
run the container, you'll get the most up-to-date version of the ruby packages
you need.

### Build the container

This container can generically mirror any github page passed to it using the
--build-arg PAGES\_REPO\_NWO option. For example, in order to mirror this wiki,
you can pass j-tt/r-i2p-wiki:

```
docker build --rm \
	--build-arg PAGES_REPO_NWO="j-tt/r-i2p-wiki" \
	--build-arg  proxy=socks5://172.81.81.6:9150 \
    --build-arg theme=jekyll-theme-minimal \
	-f Dockerfile.github -t eyedeekay/eepsite-github .
```
Now it's ready to run.

```
docker run --restart=always -i -t -d \
	--name eepsite-github \
	--network eepsite \
	--network-alias eepsite-github \
	--hostname eepsite-github \
	--link eepsite-tor \
	--ip 172.81.81.5 \
	eyedeekay/eepsite-github
```

It will take quite a while to bootstrap the github pages environment, but soon
you will have a githhub pages mirror as an eepSite.
