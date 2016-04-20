---
layout: post
title:  "Getting interactive shell in the deepest bowels of a system"
date:   2015-01-13 18:55:59
issue:  2
---
Sometimes you might need to debug a failure somewhere in the deepest bowels of
a system. Particularly a complicated shell script or a build system, where
execution is very much affected by the state of and location in the filesystem
and environment variables. In these cases getting an interactive shell in the
exact problematic environment might be very useful.

With Bash scripts that run interactively it might be sufficient to just put
this command at the desired "breakpoint":

    bash -rcfile <(set)

This starts an interactive shell with the invoking shell's functions and
variables. If you need variable attributes (including "export" status) and
various shell options transferred as well, then this might do the job:

    bash -rcfile <(declare -p; declare -f; set +o; shopt -p)

However, sometimes the script or a build system might have no access to a
terminal. Then `screen` comes to the rescue. This command:

    screen -D -m

will start a detached screen session and will wait until it is finished, so
that you can connect to it with `screen -r` and get your interactive shell.

You can combine this and either of the previous approaches, but shell state
will need to be passed via a temporary file or a named pipe, as process
substitution's file descriptors won't be inherited by the shell inside
`screen`:

    set >tmp_rcfile; screen -D -m bash -rcfile tmp_rcfile

Note though, that `screen` is a setuid executable, so it removes some
variables from the environment, such as LD_PRELOAD and LD_LIBRARY_PATH, for
safety. You will need to pass them explicitly. For example, like this:

    screen -D -m bash -c "LD_PRELOAD='$LD_PRELOAD' bash"
