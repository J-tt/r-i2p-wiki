Gaming on i2p with Freeciv and Docker
=====================================

It may surprise you, but it's actually possible to play games across anonymous
networks under the right circumstances. While it's not especially likely that
you'll be able to play real-time, demanding games while also having a high
degree of anonymity anytime soon, it's entirely possible to play turn-oriented
games, or to turn down the anonymity(and use i2p as a peer-to-peer network) and
take advantage of the higher speed. This guide builds incrementally on the
previous two guides by doing slightly more advanced configuration of i2pd's
server tunnel, and adding in a client tunnel with a fixed destination.

In this tutorial:
-----------------

  * We use single-hop tunnels. This is a bit like a VPN, but where you would use a random VPN server every time. *The intermediate node will know your IP address, but not what you are transmitting. The endpoints will not be able to discern eachother's IP addresses. If you desire greater anonymity, Freeciv can be played successfully with as many as 3 hops for both clients and servers*, but is sometimes less reliable.

Freeciv over i2p: The Server
----------------------------

### Preparation

Like the previous tutorials, this tutorial requres a docker network for
configuring the hosts and i2pd running in a docker container. For convenience's
sake, we can use the existing container and network on 172.81.81.0, or we can
create new ones. Hopefully by now this process is familiar, and readers have
realized that configuring services to run across eepSites is very similar across
the board.

Alter your tunnels.conf file to contain this section:

```
[FREECIV-SERVER]
type = server
host = i2p-freeciv
port = 5555
inbound.length = 1
inbound.quantity = 15
outbound.length = 1
outbound.quantity = 15
keys = freeciv.dat
```

You'll see that we have some new details in this one compared to our old,
http-based tunnels. While the options are pretty self explanatory, I'll go over
them here too.

  * inbound.length: This is the length of the tunnel that the client connects to to send messages to the server. A shorter length corresponds to more speed, but it's easier to conduct traffic analysis against users who are using fewer hops.
  * outbound.length: This is the length of the tunnel that the server uses to reply to clients. The same basic rules apply to it as to inbound tunnels.
  * inbound.quantity: This is the number of tunnels that will be created to deal with inbound connections. In this case we're using alot of them, because we're trying to game.
  * outbound.quantity: This is the number of tunnels that will be created to deal with outbound connections.

Now that you've customized your i2pd container, you can docker build/docker run
it.

### Setting up Freeciv

In order to make Freeciv run across i2p, it helps to make a few adjustments to
Freeciv.

#### Adjust the timeouts for connecting to the Freeciv Server

Now that you've got your server tunnel, it's time to configure the Freeciv
server. Freeciv server on Debian uses the folder /usr/share/games/freeciv/
to store it's ruleset configurations. I happen to like playing the Civ2/Civ3
ruleset: /usr/share/games/freeciv/civ2civ3.serv. Copy that file to the
directory where you're configuring your Freeciv server container. In order to
make it work reliably with i2p, turn up the timeout options a bit by copying
the following lines to the bottom:

```
set nettimeout=120
set netwait=20
set pingtime=60
set pingtimeout=600
```

Of course, you can add these lines to any freeciv ruleset. I'd just make sure
to add them at the end, to ensure that they are applied last.

#### Install the Freeciv Server in a container

A common mistake in Docker containers is to run applications as root in the
container unnecessarily. In our Dockerfile, we create a user 'freeciv' to run
our freeciv service. Besides that, we'll have to ensure that our server doesn't
connect to the clearnet Freeciv metaserver. We can do this by explicitly
defining the command to be run to start the server.

#### The Dockerfile

So, to create a container which uses your modified ruleset, you can use this
Dockerfile:

```Dockerfile
FROM debian:sid
ARG file=civ2civ3.serv
ENV file=$file
RUN apt-get update
RUN apt-get install -y freeciv-server
RUN adduser --disabled-password --gecos ',,,,' freeciv
WORKDIR /usr/share/games/freeciv/
COPY $file /usr/share/games/freeciv/$file
WORKDIR /home/freeciv
USER freeciv
CMD /usr/games/freeciv-server \
    --bind 0.0.0.0 \
    --port 5555 \
    --identity "i2p-freeciv" \
    -r /usr/share/games/freeciv/$file \
    --exit-on-end
```

And build it like this:

```
docker build --rm \
	--build-arg "file"="civ2civ3.serv" \
	-f Dockerfile -t eyedeekay/i2p-freeciv .
```

And finally, run it like this:

```
docker run --restart=always -i -t -d \
	--name i2p-freeciv \
	--network eepsite \
	--network-alias i2p-freeciv \
	--hostname i2p-freeciv \
	--ip 172.81.81.3 \
	eyedeekay/i2p-freeciv
```

Freeciv over i2p: The Client
----------------------------

### Prerequisites

Similar to the previous tutorials, except in this case it is intended to be run
on a computer other than the computer running the server. You still need an i2p
router running on the client computer.

Particularly, you will need the destination(either the base32 or the
addresshelper address) of an existing i2p-based Freeciv server. Intermittently,
there will be one available at the following b32:
4q7zxgr27pwybbqizpv6nhmriidgwj3owqt33jvuijmro57l7jdq.b32.i2p, but it's not going
to be up all the time(it's the one I used to test this on my laptop). I'm going
to use it in the example.

### Setting up i2pd

We haven't configured a client tunnel in this series of guides yet. Let's do
that now.

#### Client tunnel configuration

Clients will need a destination to connect to, corresponding to a server like
the one we already set up. In this case, the client tunnel should also listen on
all addresses so other containers on the docker network can use it to connect to
the Freeciv server.

```
[FREECIV-CLIENT]
type = client
address = 0.0.0.0
port = 5555
destination = 4q7zxgr27pwybbqizpv6nhmriidgwj3owqt33jvuijmro57l7jdq.b32.i2p
inbound.length = 1
inbound.quantity = 15
outbound.length = 1
outbound.quantity = 15
keys = freeciv-client.dat
```

You'll notice that most of the options are the same between the server and
client. The big difference, the destination option, is set to the base32 of a
freeciv server running on i2p.

### Setting up Freeciv

In this example, we're using the freeciv-client-gtk3 client to connect to the
Freeciv server. The instructions below can at least be adapted to the SDL and
QT versions, possibly all versions of the Freeciv client.

#### Freeciv Client Configuration

In order to successfully connect to the Freeciv server through i2p, use the
newly created client tunnel host:port with the --autoconnect option in your
final CMD in your Dockerfile.

```
/usr/games/freeciv-gtk3 \
    --autoconnect \
    --server 172.81.81.2 \
    --port 5555
```

#### The Dockerfile

Is extremely self-explanatory.

```Dockerfile
FROM debian:sid
RUN apt-get update
RUN apt-get install -y freeciv-client-gtk3
RUN adduser --disabled-password --gecos ',,,,' freeciv
USER freeciv
CMD /usr/games/freeciv-gtk3 \
    --autoconnect \
    --server 172.81.81.2 \
    --port 5555
```

You can build it with:

```
docker build --rm \
	-f Dockerfile.client -t eyedeekay/i2p-freeciv-client .
```

and run it by forwarding to your local X server with:

```
docker run -i -t --rm \
	-e DISPLAY=:0 \
	--name i2p-freeciv-client \
	--network eepsite \
	--network-alias i2p-freeciv-client \
	--hostname i2p-freeciv-client \
	--ip 172.81.81.4 \
	--link freeciv-i2pd \
	--volume freeciv-client:/home/freeciv \
	--volume /tmp/.X11-unix:/tmp/.X11-unix:ro \
	eyedeekay/i2p-freeciv-client
```
