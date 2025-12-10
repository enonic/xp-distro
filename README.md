# Enonic XP Distro

Welcome to the home of Enonic XP.  To get started, please read our docs here: https://developer.enonic.com/docs/xp/stable.

This project bundles the runtime (https://github.com/enonic/xp) with admin applications
(app-xp-wellcome (in sdk only), app-admin-home, app-applications, app-standardidprovider and app-users) that are necessary to use XP in a meaningful way.

## Building

Set the desired version in `gradle.properties`, and run:

    ./gradlew build

Or, to assemble distribution content and installs it on the current machine:

    ./gradlew installDist


By default, it will build a `generic` `server` build without including the JDK.


### Platform specific builds

There are 2 optional parameters:
- os: `linux` | `linux-arm64` | `mac` | `mac-arm64` | `windows` | `generic` (by default `generic`)
- type: `sdk` | `server` (by default `server`)

To assemble a distribution that includes the JDK or JRE, pass a parameter with the type, and also the desired platform: linux, mac, windows.
The type should be 'sdk' to include the JDK and 'server' to include the JRE.

    ./gradlew installDist -Pos=linux -Ptype=server

    ./gradlew installDist -Pos=mac -Ptype=sdk

    ./gradlew installDist -Pos=windows -Ptype=sdk
