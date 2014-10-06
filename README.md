#Docker Locker
> lifegadget/docker-locker [[container](https://registry.hub.docker.com/u/lifegadget/docker-locker)]

## Introduction ##

Hey wouldn't it be nice if you had a place your "stuff" in Dockerland? Well good news trooper, now you do ... put it in the "Docker Locker". This is a simple container which will serve as a [**Data Volume Container**](http://docs.docker.com/userguide/dockervolumes/#creating-and-mounting-a-data-volume-container) and has a few nifty helper methods too to make the locker be a great place to be. Enjoy. Fork, steal, contribute back. Do what you like. Make it your locker.

## Usage ##

The primary use case for the *locker* is to put something into the locker and then have it be used by other Docker containers. This is achieved with the `load` option of docker-locker:

	sudo run lifegadget/docker-locker load [thingy]
	
Sounds easy right? Well it is. All we need to understand is what "thingy" is ... thingy is one of the following:

1. **Git Repo** - just point to a git repo on *GitHub* or *BitBucket* and it will clone your repo into the container and make it available at `/locker`
2. **Tarball** - give a URL to a tarball and the locker will download, uncompress the tarball, and make it available at `/locker`

How great is that? Well it's ok but it gets better.

### Load: Environment Switches ###

In many cases you make just be happy to use things out-of-the-box but the *locker* provides you some flexibility for your flexible lifestyle needs:

- `BRANCH` - allows you specify which branch/tag you want to pull from your git repo. By default it will assume 'master' but choose what you like. 
- `OFFSET` - sometimes you'll want the repo checked out but the volume you're wanting to share is actually not stemming from the root of the source. Not a problem sir, you da boss, just specify a string offset from the root of your repo and the volume share will start off your offset. 
- `PREP` - so maybe you're one of those "I'm never happy types", always asking for something more, maybe what you want is to do something bespoke to your repo? Sound familiar? Well then, you'll pleased to know that here in the *locker* this sort of selfish behaviour is not only expected but rewarded. Here's how it works, you just specify an array of commands you'd like run over your *thingy* and once it's been put into the container we'll do your evil bidding. 

### Load Examples ###

While talking in the abstract with terms like *thingy* is great fun and highly entertaining (for us not you), it sometimes leaves one wondering if there's any substance behind the abstract posing. Well thanks for raining on the party but you're right and here you go:

Assume that ...

- NodeJS Service: 
	
	You love JavaScript so much you want to see it on the server as well as the client. It's your right. We reserve no judgement. So anyway, you've got this killer REST API built using Node and it lives on GitHub at `http://github/some-killer-app/nodejs`. You believe that living on the edge is the only place to live so obviously the `master` branch is far too passé so instead you want the locker to hold the `this-shits-bleeding` branch. So here's how you get that shit out the door and into the locker:

		sudo git run lifegadget/docker-locker run \
			load http://github/some-killer-app/nodejs \
			-e BRANCH='this-shits-bleeding' \
			-e OFFSET='public' \
			-e PREP='["npm install", "bower update"]' \
			--name nodeApp
		sudo git run -d lifegadget/docker-nginx run \
			--volumes-from nodeApp

- Couchbase Database:

	SQL is so yesterday, and you're here bring NOSQL to the masses. Couchbase may not have the immediate street cred as Redis or Mongo but you're a sucker of crazy scalable performance and you're buying the Couchbase sponsored research that shows just how smart a techno hipster you are by using this NO-SQL genius. Pat yourself on the back ... it's so good to be you. Here's what you need to do load up your data partition:

		sudo git run lifegadget/docker-locker run \
			load http://some-killer-app.s3-website-eu-1west-1.amazonaws/i-am-a-nosql-god.tar.gz 
			--name CDB
		sudo git run -d lifegadget/couchbase start \
			--volumes-from CDB

	> this is probably not a very good example, we promise to improve the example in the future when we care more ... in the mean time don't hesitate sending us a PR with a better one.

## Other Commands

The nice thing about Docker containers is that they are perpetual, meaning they don't need to always be running to keep their state on shared volumes. In almost all cases you'll start using `docker-locker` by loading something into it. Equally as likely you'll use this data volume in other containers with the `--`