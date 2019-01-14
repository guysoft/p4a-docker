p4a docker
==========

Genrated p4a docker files from [p4a-build-spaces](https://github.com/JonasT/p4a-build-spaces) so they can be hosted in docker hub.

The containers are avilable at: https://hub.docker.com/r/guysoft/p4a

Avilabe tags:


* ``p4a-py2-api19-ndkbundle``
* ``p4a-py2-api28ndk21``
* ``p4a-py3-api28ndk19``
* ``p4a-py3-api28ndk21``
* ``p4a-py3crystax-api19-ndkbundle``
* ``p4a-py3crystax-api26ndk19``

How to use
==========
Use them by running
```
docker pull guysoft/p4a:p4a-py2-api28ndk21
```

docker-comose
-------------

You can use a ``docker-compose.yml`` in your project which would let you access your files from the container, and lets you ship it so others can build your kivy apps with only docker installed:


```
version: '2'

services:

  buildozer:
    privileged: true
    image: guysoft/p4a:p4a-py3-api28ndk21
    container_name: buildozer
    tty: true
    entrypoint: /bin/sh
    volumes:
      - ./:/buildozer/
      # The following volume let ubuntu machines push android apps via adb
      - /dev/bus/usb:/dev/bus/usb 
      # The followimg volume lets you run android sdk GUI and manage your sdks
      - /tmp/.X11-unix:/tmp/.X11-unix
```
