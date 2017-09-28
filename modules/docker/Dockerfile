FROM java:8-jre-alpine

# Environment variables
ENV XP_ROOT /xp
ENV XP_APP $XP_ROOT/app
ENV XP_HOME $XP_ROOT/home
ENV XP_USER enonic-xp
ENV XP_UID 1337

# Add distro files
ADD ./build/distro/ $XP_APP

# Adding Enonic XP user
RUN adduser -h $XP_ROOT -g "" -H -u $XP_UID -D $XP_USER
RUN chown -R $XP_USER $XP_ROOT

# Exposing web port, debug port and telnet port
EXPOSE 8080 5005 5555

# Expose volume
VOLUME /xp/home

# Command to run
CMD ["/xp/app/bin/server.sh"]
