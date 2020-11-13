# Enonic XP Distro

Welcome to the home of Enonic XP.  To get started, please read our docs here: http://xp.readthedocs.org.

This project bundles the runtime (https://github.com/enonic/xp) with admin applications
(app-admin-home, app-applications, app-standardidprovider and app-users) that are necessary to use XP in a meaningful way.

## Building

Set the desired version in `gradle.properties`, and run:

    gradle build
    
### Platform specific builds

By default it will build a generic build without including the JDK.

There are 2 optional parameters:
- os: `linux` | `mac` | `windows` (by default the current platform)
- type: `generic` | `sdk` | `server` (by default `generic`)
 
To build a distribution that includes the JDK or JRE, pass a parameter with the type, and also the desired platform: linux, mac, windows.
The type should be 'sdk' to include the JDK and 'server' to include the JRE.

    gradle build -Pos=linux -Ptype=server
    
    gradle build -Pos=mac -Ptype=sdk
    
    gradle build -Pos=windows -Ptype=sdk

Omit the os parameter and specify type=sdk or type=server to build a distribution for the current platform:

    gradle build -Ptype=sdk
    
    gradle build -Ptype=server

## Running

After building the project, you can start it locally by running the server script (or `bat` on Windows):

    build/install/bin/server.sh

## Upload Docker Image

To upload `xp-base` docker image for this version:

    gradle pushDocker

You will need to set `dockerUser`, `dockerEmail` and `dockerPassword` in `~/.gradle/gradle.properties`
file.
