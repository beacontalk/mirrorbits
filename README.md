[![Build Status](https://travis-ci.org/etix/mirrorbits.svg?branch=master)](https://travis-ci.org/etix/mirrorbits)
[![Go Report Card](https://goreportcard.com/badge/github.com/etix/mirrorbits)](https://goreportcard.com/report/github.com/etix/mirrorbits)

# Mirrorbits

Mirrorbits is a geographical download redirector written in [Go](https://golang.org) for distributing files efficiently across a set of mirrors. It offers a simple and economic way to create a Content Delivery Network layer using a pure software stack. It is primarily designed for the distribution of large-scale Open-Source projects with a lot of traffic.

![mirrorbits_screenshot](https://cloud.githubusercontent.com/assets/38853/3636687/ab6bba38-0fd8-11e4-9d69-01543ed2531a.png)

## Main Features

* Blazing fast, can reach 8K QPS on a single laptop
* Easy to deploy and maintain, everything is packed in a single binary
* Automatic synchronization with the mirrors over **rsync** or **FTP**
* Response can be either JSON or HTTP redirect
* Support partial repositories
* Complete checksum / size control
* Real-time monitoring and reports
* Disable misbehaving mirrors without human intervention
* Real-time decision making based on location, AS number and defined rules
* Smart load-balancing over multiple mirrors in the same area to avoid hotspots
* Ability to adjust the weight of each mirror
* Limit access to a country, region or ASN for any mirror
* Clustering (multiple Mirrorbits instances)
* High-availability using redis-sentinel
* Automatically fix timezone offsets for broken mirrors
* Real-time statistics per file / mirror / date
* Real-time reconfiguration
* Seamless binary upgrade (aka zero downtime upgrade)
* [Mirmon](http://www.staff.science.uu.nl/~penni101/mirmon/) support
* Full **IPv6** support
* more...

## Is it production ready?

**Yes!** Mirrorbits has served **billions** of files already and is known to be running in production at:

* [CarbonROM](https://carbonrom.org/)
* [Chaos Computer Club](https://media.ccc.de/) to distribute media
* [Jellyfin](https://jellyfin.org/) since [April 2021](https://jellyfin.org/posts/mirrorbits-cdn/)
* [Jenkins](https://www.jenkins.io/) to distribute Jenkins releases since [February 2020](https://github.com/jenkins-infra/docker-mirrorbits)
* [Kodi](http://kodi.tv/) (previously XBMC) since [July 2015](https://forum.kodi.tv/showthread.php?tid=233824)
* [LineageOS](http://lineageos.org/) (previously CyanogenMod) since January 2017
* [VideoLAN](http://www.videolan.org/) to distribute [VLC media player](http://www.videolan.org/vlc/) since [April 2014](https://blog.l0cal.com/2014/07/11/mirrorbits-is-now-on-github/)

Yet some things might change before the 1.0 release. If you intend to deploy Mirrorbits in a production system it is advised to notify the author first so we can help you to make any transition as seamless as possible!

_Previous projects which have used Mirrorbits:_
* [Endless OS](https://endlessos.com/)
* [OSMC](https://osmc.tv)
* [Parrot OS](https://www.parrotsec.org)
* [Popcorn Time](https://popcorntime.io)
* [SuperRepo](https://superrepo.org)

# Quick start

## Prerequisites

* Go 1.11 or later
* Protobuf (protoc)
* Redis 3.2 or later (with [persistence](https://redis.io/topics/persistence) enabled)
* GeoIP2 databases from [Maxmind](https://dev.maxmind.com/geoip/geoip2/geolite2/) (preferably updated regularly)

:warning: **GeoIP-legacy is not supported any more, please use the new GeoIP2 mmdb databases!**

**Optional:**

* redis-sentinel (for high-availability support)

## Upgrading

Before upgrading to the latest version, please check [this guide](https://github.com/etix/mirrorbits/wiki/Upgrade-Guide).

## Installation

You can either get a [pre-built version](https://github.com/etix/mirrorbits/releases), [Docker image](https://github.com/etix/mirrorbits/wiki/Running-within-Docker), [Debian package](https://packages.debian.org/unstable/net/mirrorbits) or choose to build it yourself.

For more information see the [Quick start wiki page](https://github.com/etix/mirrorbits/wiki/Quick-Start).

### Pre-built Version

:warning: _The latest pre-built version at times will be behind:_

```
wget $(wget https://api.github.com/repos/etix/mirrorbits/releases/latest -O -2>/dev/null|grep browser_download_ur|cut -d'"' -f4)
tar -xvf mirrorbits-v*.tar.gz
cd mirrorbits/
```

### Docker Image

Mirrorbits does not yet have an official image. _The next release will be supported._

A docker "quick start" can be found [on the wiki](https://github.com/etix/mirrorbits/wiki/Running-within-Docker) showing how to build it locally.

### Debian Package

[Debian](https://www.debian.org/) has Mirrorbits [package](https://packages.debian.org/unstable/net/mirrorbits) ready to go:

```
sudo apt install mirrorbits
```

### Manual Build

Debian-based OS need the following packages pre-install:

```
sudo apt install build-essential golang geoipupdate redis-server protobuf-compiler
```

Go >= 1.11:

```
git clone https://github.com/etix/mirrorbits.git
cd mirrorbits
sudo make install
```

Go < 1.11:

```
go get -u github.com/etix/mirrorbits
cd $GOPATH/src/github.com/etix/mirrorbits
sudo make install
```

The resulting executable should now live in your */usr/local/bin* directory. You can also specify a `PREFIX` or `DESTDIR` if necessary:

```
sudo make install PREFIX=/usr
```

## Configuration

A sample configuration file can be found [here](mirrorbits.conf).

## Running

Mirrorbits is a self-contained application and can act, at the same time, as the server and the cli.

To run the server:

```
mirrorbits daemon
```
Additional options can be found with ```mirrorbits -help```.

To run the cli:

```
mirrorbits help
```

Add a mirror:

```
mirrorbits add -ftp="ftp://ftp.mirrors.example/myproject/" -http="http://ftp.mirrors.example/myproject/" mirrors.example
```

Enable the mirror:

```
mirrorbits enable mirrors.example
```

### Real-time file availability

By appending `?mirrorlist` to any file served by Mirrorbits, you'll be able to get some useful real-time informations about the given file. You can see a [live example here](https://get.videolan.org/vlc/2.2.4/win32/vlc-2.2.4-win32.exe?mirrorlist).

### Real-time mirrors statistics

Mirror statistics are available by querying Mirrorbits with the `?mirrorstats` argument. You can see a [live example here](https://get.videolan.org/?mirrorstats).

## Clustering / High availability

Multiple instances of Mirrorbits can be started simultanously on different servers, discovery of other nodes should be automatic as long as all the instances are connected to the same redis server. In addition to the clustering it is advised to use redis-sentinel to monitor the database and gracefuly handle failover.

## Upgrading

Mirrorbits has a mode called *seamless binary upgrade* to upgrade the server executable at runtime without service disruption. Once the binary has been replaced on the filesystem just issue the following command in the cli:

```
mirrorbits upgrade
```

## Considerations

* When configured in redirect mode, Mirrorbits can easily serve client requests directly but it is usually recommended to set it behind a reverse proxy like nginx. In this case take care to pass the IP address of the client within a X-Forwarded-For header:

```
proxy_set_header X-Forwarded-For $remote_addr;
```

* It is advised to never cache requests intended for Mirrorbits since each request is supposed to be unique, caching the result might have unexpected consequences.

# We're social!

The best place to discuss about Mirrorbits is to join the [#VideoLAN IRC channel](https://wiki.videolan.org/Contact_VideoLAN/#IRC) on [libera.chat](https://libera.chat/).
For the latest news, you can follow [@mirrorbits](http://twitter.com/mirrorbits) on Twitter.

# License MIT

> Permission is hereby granted, free of charge, to any person obtaining a copy
> of this software and associated documentation files (the "Software"), to deal
> in the Software without restriction, including without limitation the rights
> to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
> copies of the Software, and to permit persons to whom the Software is
> furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in
> all copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
> IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
> FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
> AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
> LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
> OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
> THE SOFTWARE.
