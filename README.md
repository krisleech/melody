# Melody

A small demo of evented HTTP services using Routemaster.

## Requirments

* VirtualBox
* Vagrant

## Quickstart

```
./bin/launch
```

This will launch 3 Virtual Machines.

http://localhost:9000 Jukebox App
http://localhost:7777 JukeStats App
https://localhost:4434 Routemaster
http://localhost:5555 RabbitMQ Admin UI

Go to the Jukebox app, vote for some songs.
Then visit the JukeStats app and see the recorded statistics.
