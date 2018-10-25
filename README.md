# Enonic XP Distro

Welcome to the home of Enonic XP.  To get started, please read our docs here: http://xp.readthedocs.org.

This project bundles the runtime (https://github.com/enonic/xp) with the admin applications (https://github.com/enonic/xp-apps)
that are necessary to use XP in a meaningful way.

## Building

Set the desired version in `gradle.properties`, and run:

    gradle build
    
### Platform specific builds

By default it will build a generic build without including the JDK.

There are 2 optional parameters:
- os: `linux` | `mac` | `windows` (by default the current platform)
- type: `generic` | `jdk` | `jre` (by default `generic`)
 
To build a distribution that includes the JDK or JRE, pass a parameter with the type, and also the desired platform: linux, mac, windows  

    gradle build -Pos=linux -Ptype=jre
    
    gradle build -Pos=mac -Ptype=jdk
    
    gradle build -Pos=windows -Ptype=jdk

Omit the os parameter and specify type=jdk or type=jre to build a distribution for the current platform:

    gradle build -Ptype=jdk
    
    gradle build -Ptype=jre

## Running

After building the project, you can start it locally by running the server script (or `bat` on Windows):

    modules/distro/build/install/bin/server.sh

## Upload Docker Image

To upload `xp-base` docker image for this version:

    gradle pushDocker

You will need to set `dockerUser`, `dockerEmail` and `dockerPassword` in `~/.gradle/gradle.properties`
file.
