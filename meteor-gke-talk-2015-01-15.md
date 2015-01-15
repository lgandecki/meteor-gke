footer: [Q42.com](http://q42.com)
slidenumbers: true

^- make a short url for the different IPs of the clusters
^- $500 giveaway
^- email kit draft of talk

# Hi, I'm @Rahul!
## Running US office of @Q42

^Today I'm going to talk a little about our efforts at building a script that will
help you deploy your Meteor app to Container Engine.

^But first you should know who I am and why we're doing this.

^I work at Q42, a Dutch tech consulting company of which I'm running the US branch.

---

# Explorers & creative technologists

![left 200%](/Users/rahul/Dropbox/dev/Q42/Q42 logo/Q42_logo_RGB.png)

- 60 engineers in Mountain View & NL
- Early start on App Engine in '08
- Early start with Meteor in '12
- Both now core technologies for us

^We're also explorers: of our 60 employees, 55 are engineers and we love learning new stuff that
helps us spend more time on building cool products.

^So we got into App Engine in 2008 because it felt like the right direction for a cloud platform.
And we started playing with Meteor in 2012. Both of these follow the philosophy of minimising
dev ops, giving you more time to spend on the business logic.

^While we've progressed to using Google Cloud Platform to host all kinds of large scale projects
like the infrastructure of the Philips Hue smartphone-controlled lights,
we're still at that stage of being a little insecure about
Meteor. We use it a lot: internal tools, projects, even our own educational startup. But it's
clearly still growing.

---

# Jumpstarts

- 1 week project with your startup
- Start Monday, end Friday
- Result is working code thanks in part to Meteor
- More at [q42.com](http://q42.com)

^One area where it's really excelled is in our jumpstarts. We use 'jumpstart' to refer to a
week-long acceleration we do with startups and companies still in search of the right idea.
Often we'll start with a high level idea and end with a working first version of the product.
Meteor has been invaluable here.

---

# Why Meteor?

- Dramatically faster iteration
- Client & server - fully functional demo
- Everyone can write Javascript

^With Meteor so many things we used to worry about are a thing of the past. Like using
grunt to watch a folder and compile your less files. Or having to reload the page after
you change a file. But also basic front end stuff like the glue or boilerplate you always
have to write to send data to the server, or do form validation, etc. It's like programming
in the future. On top of that, since the whole stack - client, server, and database -
is in Javascript, basically anyone can join the team at any time. There's barely any
learning curve.

---

# Why Container Engine?

- All the features of Cloud Platform
- Declarative configuration in JSON
- Kubernetes is open source
- Containers, duh!

^So why are we looking at Container Engine? Because we want Meteor to be able to make that leap.
From a focus on tools, sprints, and demos to supporting real scale. It feels right. But it's not
there yet. And while the company behind Meteor is working on a hosting product called Galaxy,
there's no release date in sight. So we need something in the mean time.

^We started looking at options and discovered that Container Engine would be a great fit.
Since we're really excited about container tech, we've been keeping an eye on Container Engine
and its underlying Kubernetes platform. We got in touch with Google and have been working on
putting together the instructions necessary to get a Meteor app running on Container Engine.
We want to open source it and get feedback from both communities about which direction to
take this, but this is our first attempt.

---

# Getting started

- Install `gcloud` command line
- Install `preview` features

^Just as a reminder: all of the following code depends on installing the command line tools!

^Also before we get started, one note: I'm actually a designer; one of my colleagues
in the Netherlands actually wrote the scripts, I just happen to be the one giving the talk.
So keep that in mind and feel free to correct me if I say something that makes no sense.
I'll also be relying on Kit to keep me on point :)

---

![fit](https://raw.githubusercontent.com/Q42/meteor-on-gke/master/containers.png?token=AAIN4IedoSGJNVHkscZol5N9g-YOyv_Nks5UwLw2wA%3D%3D)

^This is a high level overview of what I'm going to outline. Hope it makes sense :)

---

# The following code is open source

## [github.com/q42/meteor-on-gke](http://github.com/q42/meteor-on-gke)

---

# Demo

## [http://146.148.123.98](http://146.148.123.98/)

---

# What we need

1. Script to set everything up
2. JSON configuration
3. Persistent disk for Mongo
4. Meteor Docker image
5. Example Meteor app

---

# Step 1: Set everything up

```sh
gcloud preview container clusters create meteor

gcloud preview container pods create
	--config-file mongo-pod.json

gcloud preview container services create
	--config-file mongo-service.json

gcloud preview container replicationcontrollers create
	--config-file meteor-controller.json
gcloud preview container services create
	--config-file meteor-service.json

gcloud compute firewall-rules create meteor-80
	--allow=tcp:80
	--target-tags k8s-meteor-node
```

---

# Step 2: Set up a persistent disk for MongoDB

```sh
gcloud compute disks create
	--size=$DISK_SIZE
	--zone=$ZONE $DISK_NAME
```

---

# Step 3: JSON configuration
## 1. Setting up a pod

```javascript
{
	"id": "mongo",
	"kind": "Pod",
	"desiredState": {
		...
	},
	"labels": {
		...
	}
}
```

^The important thing about this configuration format is that
^it's declarative: we just describe what we want our cluster
^to look like, and when we tell Kubernetes or GKE about it,
^it's their job to figure out how to make it happen.
^We'll see why this is great in a moment.

---

## Setting up a pod (2)

```javascript
"containers": [{
	"name": "mongo",
	"image": "mongo",
	"cpu": 1000,
	"ports": [{ "name": "mongo", "containerPort": 27017 }],
	"volumeMounts": [{
		"mountPath": "/data/db",
		"name": "mongo-disk"
	}]
}]
```

^We want a pod with a volume mounted to a given disk and exposed
^over a given port.

---

## Setting up a pod (3)

```javascript
"volumes": [{
	"name": "mongo-disk",
	"source": {
		"persistentDisk": {
			"pdName": "mongo-disk",
			"fsType": "ext4"
		}
	}
}]
```

^We want a persistent disk that stays around even if we tear down the cluster.

---

## 2. Setting up a replication controller

```javascript
{
	"id": "meteor-controller",
	"kind": "ReplicationController",
	"apiVersion": "v1beta1",
	"desiredState": {
		...
	},
	"labels": {"name": "meteor"}
}
```

^Replication controllers follow a pretty similar format to pods.

---

## Setting up a replication controller (2)

```javascript
"replicas": 3,
"replicaSelector": {"name": "meteor"},
"podTemplate": {
	"desiredState": {
		"manifest": {
			"version": "v1beta1",
			"id": "meteor-controller",
			"containers": [{
				"name": "meteor",
				"image": "chees/meteor-gke-example",
				"cpu": 1000,
				"memory": 500000000,
				"ports": [{"name": "http-server", "containerPort": 8080, "hostPort": 80}]
			}]
		}
	},
	"labels": { "name": "meteor" }
}
```

^This tells GKE that we want 3 replicas using the following template
^for the pods running inside it. Note the 1000 value for CPU, which is measured
^in milli-cores :)

---

## 3. Setting up the Mongo service

```javascript
{
	"id": "mongo",
	"kind": "Service",
	"apiVersion": "v1beta1",
	"port": 27017,
	"containerPort": "mongo",
	"selector": {
		"name": "mongo", "role": "mongo"
	},
	"labels": {
		"name": "mongo"
	}
}
```

^Here we're telling Mongo to go ahead and run on a given port

---

## 4. Setting up the Meteor service

```javascript
{
	"apiVersion": "v1beta1",
	"kind": "Service",
	"id": "meteor",
	"port": 80,
	"containerPort": "http-server",
	"selector": { "name": "meteor" },
	"createExternalLoadBalancer": true,
	"sessionAffinity": "ClientIP"
}
```

^Finally, we define that all of this should be accessible on port 80 (as defined earlier)
^The last two lines are important: we set up a load balancer and
^tell it to apply session stickiness based on your IP. So every time
^you visit from the same machine, you should get the same machine and
^the same session.

---

# Step 4. Docker image for Meteor

## [fit][registry.hub.docker.com/u/chees/meteor-kubernetes](https://registry.hub.docker.com/u/chees/meteor-kubernetes/)

^Now that we've configured how Container Engine should serve our app,
^we need to wrap Meteor in Docker. Thankfully we've gone ahead and done
^that and it's available here.

---

# Step 5. Example Meteor app

### [registry.hub.docker.com/u/chees/meteor-gke-example](https://registry.hub.docker.com/u/chees/meteor-gke-example/)

^As I demo'd earlier, we also made a simple Meteor app that runs in
^the container. It uses MongoDB to store some data, syncs that data
^down to the client over DDP, and gives you a sense of whether your
^configuration is working correctly.

^The way we've set it up now is with a repo watcher on Github that
^will inform the Docker registry whenever we push code, and rebuild
^the associated image.

---

# So what if we want to scale?

## Just change the number of replicas!

**`"replicas": 5,`**

```javascript
"replicaSelector": {"name": "meteor"},
"podTemplate": {
	...
}
```

^This feels like magic to me, but maybe I'm just new at this.

---

# (more slides go here)

- show bash command to increase number of replicas
- load testing?
- deployversion.sh
- session affinity

- slide with links to docs for GKE/Kubernetes


---

# Future options

- Extend Meteor command line?
	- `meteor deploy gke`?
- Auto-scaling?
- Fix MongoDB bottleneck?
- ??? What do you need?

---

# Get involved!

## [github.com/q42/meteor-on-gke](http://github.com/q42/meteor-on-gke)

---

![fit left](~/Desktop/goobernetes.png)

# Thanks for coming :)

- Next week: "Introduction to Meteor"
	- Jan 21st, 6pm, same place
	- [meetup.com/javascript-9](http://www.meetup.com/javascript-9/events/219807509/)
- Wanna host a meetup? Get in touch! [@q42](http://twitter.com/q42) or [@rahul](http://twitter.com/rahul)

---

# Demo

x slide showing meteor on kubernetes overview
x talk about setting up mongo on a persistent disk
- update the image being deployed to the latest version
- show json config, talk about declarative nature
- talk about scaling by changing number of replicas
- show creating an image from meteor app and publishing it