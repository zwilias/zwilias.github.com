---
title: "MongoReducer, Map/Reduce in MongoDB."
layout: post
date: 2013-06-05 22:10
tag:
- mongo
- javascript
#image: /assets/images/jekyll-logo-light-solid.png
headerImage: false
projects: true
hidden: true # don't count this post in blog pagination
description: "Old, OLD, old project. MapReduce in a cron-like setup, without external dependencies"
jemoji: '<img class="emoji" title=":older_man:" alt=":older_man:" src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f474.png" height="20" width="20" align="absmiddle">'
author: ilias
externalLink: false
---

[MongoReducer](https://github.com/zwilias/mongoReducer) allows running mongo's mapreduce functionality from within mongo.

---

> Specifically, this is a framework of sorts, built using convention over configuration in mind, which allows saving a complete map-reduce actions, with its mapper and reducer and all other configuration and settings in the `db.mapreduce` collection and running it with a single command.
>
> Furthermore, MongoReducer comes with a polling loop which polls the `db.mapreduce` collection (amongst others) and will run all map-reduce actions found within, at specified time-intervals, without the need for a complicated setup involving a number of cronjobs and a host of other scripts for actually executing the map-reduce actions.
>
> To this end, MongoReducer is split into two main parts, the Poller object and the MapReduce object.

---

*Disclaimer*: This is old. Way old. I don't know to what extent this thing still functions, or if anyone every used it at all. However, it was a pretty neat project, and looking back at the code I wrote for it, back then, makes me realize how much I've learnt over the past few years. Whoosh, nostalgia.

So there you have it. I'm including this project here, for the sake of nostalgia.
