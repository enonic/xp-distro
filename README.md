# Enonic XP Distro

This repository builds the full distro (runtime + bundled apps).

## Upload Docker Image

To upload `xp-base` docker image for this version:

```
gradle pushDocker
```

You will need to set `dockerEmail` and `dockerPassword` in `~/.gradle/gradle.properties`
file.
