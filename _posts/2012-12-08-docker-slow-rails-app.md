---
layout: post
title: Docker is great, slow Rails apps is not
---

It's [apparently a known issue](http://oliverguenther.de/2015/05/docker-containers-for-development/)
to have slow Rails apps under Docker in development due to folder sharing in VirtualBox.

There are many suggestions as to how to fix this, so I tried a few.

* [Sharing Host Volumes with Docker Containers](http://oliverguenther.de/2015/05/docker-host-volume-synchronization/) is a good overview of multiple things to try by Oliver Günther.
* I started with [docker-osx-dev](https://github.com/brikis98/docker-osx-dev) but received rsync errors after running it for a few minutes.
* Switched to VMWare Fusion and that didn't help so maybe it's not VirtualBox file sharing.
* Tried [docker-rysnc](https://github.com/synack/docker-rsync) under Fusion but it gave me:

```bash
rsync: connection unexpectedly closed (0 bytes received so far) [sender]
rsync error: error in rsync protocol data stream (code 12) at /BuildRoot/Library/Caches/com.apple.xbs/Sources/rsync/rsync-47/rsync/io.c(453) [sender=2.6.9]
error: exit status 12
Watching for file changes ...
```

Maybe this magic bullet shit won't work, let's try using an image as part of compose so that we're more explicit about this syncing process.

* Looks like [docker-unison](https://github.com/leighmcculloch/docker-unison) works to sync files and it syncs them quickly but the application still takes 20 seconds to load a simple page.

NFS options:

* [Boot2docker: Using nfs instead of vboxsf to mount /Users](http://syskall.com/using-boot2docker-using-nfs-instead-of-vboxsf/)
* [How Blackfire leverages Docker](http://blog.blackfire.io/how-we-use-docker.html) under the File Sharing section
* [boot2docker-nfs.rb](https://gist.github.com/mattes/4d7f435d759ca2581347) is a simple gist working with VirtualBox

Since I have Fusion installed though, this seems like a lot of work to just get a basic Rails app to run.

My solution:

* Use Docker for everything not Rails or Guard. Postgres and workers seem fast enough.
* Run Rails locally with your standard `gem install bundler; bundle`.
* Leave the app running in Docker Compose if I need to check out anything in Linux but avoid it at all costs.

I've got too much real work to do.
