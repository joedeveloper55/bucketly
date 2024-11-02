# bucketlyd
A network daemon that runs on Python and listens for arbitrary events sent to "buckets" via the bucketly v1 prototcol (a protocol used over long lived TCP connections). These "buckets" are named locations that aggregate the sent events and possibly perform automated scheduled processing based on custom user written logic. Buckets also enable arbitrary "views" and "dumps" of themselves. If multiple processes or threads are sending commands to a bucket (send, view, dump, etc), all access is strictly serialized, and the order of events over a given connection is preserved.

Typically, most event aggreagation is performed in memory and most bucket state is held in memory.
This enables bucketlyd to process very high write volume of events efficiently. There is an option to durably persist bucket state to disk though, configurable on per bucket basis, but with the expected performance drawbacks.

There is [a sample project](), demonstrating the use of bucketly to count unique clicks to urls, sending up changes approximately once every minute to a postgres database. It functions as a scalable buffer (if we had multiple producers of events, we could have multiple bucketly servers) protecting the realtional database from excessive write events.

## installation and configuration

Installing bucketlyd is accomplished via a simple

> pip install bucketly

Once it's installed, you should have access to the "bucketlyd" command.

Type

> bucketlyd --help

to see the usage.

The below command starts the bucketly daemon, and has it listen at localhost on port 8484

> bucketlyd bucketly_database.dat  --bucket_handlers hndlrs.py --hostname 127.0.0.1 --port 8484

bucketly_database.dat is a file that persists your bucket configuration and users and permissions. It might also persist bucket state in a durable manner if you configured certain buckets to be "durable".

When starting bucketlyd for the first time with an empty database file, an "admin" user is created with password "admin", with all the permissions needed to do anything on bucketly. The first thing you should do is change this admin password. You should generally also leave the admin permissions as is unless you really know what you're doing. If you revoke certain permissions from the admin user, you may find yourself unable to modify any permissions or create and manage any new users in a bucketly system, without directly going into the raw database file. In a production system, bucketlyd clients should *never* login in with the admin user. 

hndlrs.py is a python module containing the custom handler logic applied to your buckets. buckets can only be created with handlers present in this file.

## usage
Any client that can talk the "bucketly v1 protocol" (over TCP) can use bucketlyd. Any language with a functional bucketly client can be used. A Python bucketly client library already exists [here]().

There is also a command line app / client [here]()

Since this server is written in Python, bucket handlers must be written in Python.

This project does not maintain or plan to build clients or servers for other languages, although the bucketly v1 protocol is flexible enough to allow clients and servers to be written other languages.

There is [detailed documentation on the bucketly v1 protocol]() (however, this is only particularly relevant to those who wish to implement their own bucketlyd servers or clients, the typical user need not review these docs).

## inspiration
If you're familiar with statsd, reading this README probably reminded you of it a bit; That is because this system was heavily inspired by statsd. We were inpired by the amazingly simple developer experience of statsd, coupled with the impressive robustness and performance of the tool. We wanted to generalize the idea of fast and easy aggregation to be used beyond just stats collection.
