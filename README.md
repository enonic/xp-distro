# Enonic XP Distro

Welcome to the home of Enonic XP.  To get started, please read our docs here: http://xp.readthedocs.org.

This project bundles the runtime (https://github.com/enonic/xp) with the admin applications (https://github.com/enonic/xp-apps)
that are necessary to use XP in a meaningful way.

## Building

Set the desired version in `gradle.properties`, and run:

    gradle build
    
### Platform specific builds

By default it will build a generic build without including the JDK.

To build a distribution that includes the JDK, pass a parameter with the desired platform: linux, mac, windows  

    gradle build -Pplatform=linux
    
    gradle build -Pplatform=mac
    
    gradle build -Pplatform=windows

Pass platform=current to build a distribution for the current platform:

    gradle build -Pplatform=current

## Running

After building the project, you can start it locally by running the server script (or `bat` on Windows):

    modules/distro/build/install/bin/server.sh

## Upload Docker Image

To upload `xp-base` docker image for this version:

    gradle pushDocker

You will need to set `dockerUser`, `dockerEmail` and `dockerPassword` in `~/.gradle/gradle.properties`
file.
