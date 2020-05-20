# Yet Another Proxy (YAP) for SmoothStreams.tv Docker Image

## Usage

```
docker create \
        --name sstvproxy \
        --hostname sstvproxy \
        -p 8098:8098 -p 8099:8099 \
        -e PUID=<UID> -e PGID=<GID> \
        -e TZ=<timezone> \
        -e YAP_USERNAME=<username> -e YAP_PASSWORD=<password> \
	    -e YAP_SERVICE=<short_code_for_smoothstreams_service> \
	    -e YAP_ARGS=<command line args> \
        -v </path/to/appdata>:/config \
        stokkes/sstvproxy
```

This container is based on alpine linux with s6 overlay from [LinuxServer](http://linuxserver.io). For shell access whilst the container is running do docker exec -it sstvproxy /bin/bash.

Currently only the master branch is available from YAP.

## Parameters

`The parameters are split into two halves, separated by a colon, the left hand side representing the host and the right the container side. 
For example with a port -p external:internal - what this shows is the port mapping from internal to external of the container.
So -p 8080:80 would expose port 80 from inside the container to be accessible from the host's IP on port 8080
http://192.168.x.x:8080 would show you what's running INSIDE the container on port 80.`

So INSIDE the container, 8098 is mapped to the local port and 8099 is mapped to the external port. You can map those to your host on any port you want. The right side of the colon must always be 8098/8099 or this container won't work.

* `-p 8098` - YAP exposed local port
* `-p 8099` - YAP exposed external port
* `-v /config` - YAP App data (settings files and cache)
* `-e PGID` for GroupID - see below for explanation
* `-e PUID` for UserID - see below for explanation
* `-e TZ` for timezone EG. Europe/London, America/NewYork
* `-e YAP_GIT_BRANCH` for specifying which branch to use (`master` or `dev`), defaults to `master` if not set
* `-e YAP_SERVICE` for code for SS service EG. `viewms`, etc.
* `-e YAP_USERNAME` for SS username
* `-e YAP_PASSWORD` for SS password
* `-e YAP_SERVER` for SS server EG. `dnae2`, `dmaw2`, etc.
* `-e YAP_QUALITY` for quality (`1` for HD, `2` for HQ, `3` for SD)
* `-e YAP_STREAM` for stream type (`rtmp` or `hls`) 
* `-e YAP_EXTERNALIP` for specifying external IP to use
* `-e YAP_KODIPORT` for Kodi port
* `-e YAP_ARGS` for command line arguments (-t -hl -d)

### User / Group Identifiers

Sometimes when using data volumes (`-v` flags) permissions issues can arise between the host OS and the container. We avoid this issue by allowing you to specify the user `PUID` and group `PGID`. Ensure the data volume directory on the host is owned by the same user you specify and it will "just work" ™.

In this instance `PUID=1001` and `PGID=1001`. To find yours use `id user` as below:

```
  $ id <dockeruser>
    uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)
```

## Setting up the application

### config Folder

The `/config` folder in the container contains the two settings files (`proxysettings.json` and `advancedsettings.json`) as well as the `cache` folder. 

You'll notice the app is not in this folder. During the container initialization process, symlinks are created withini the app folder (located in `/app/sstvproxy` in the container).  This is done to allow new versions of YAP without erasing your custom configurations.

### Docker environment variables for YAP

You'll notice there are many environment variables for YAP that you can pass when creating the container. **Environment variables will take precedence over manual changes to `proxysettings.json` and will persist across container restarts**. This means that if you set the `YAP_USERNAME` and `YAP_PASSWORD` for instance when you create the container, these will always be placed in the `proxysettings.json` file, even if you edit the file manually with a text editor. 

If you prefer not having this occur, do not pass any environment variables and instead just edit the proxysettings.json file manually and drop it in your local folder mapped to `/config` inside the container. 

_This behaviour may change at a later time._
