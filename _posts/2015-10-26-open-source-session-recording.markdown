---
layout: post
title:  "Open-Source Session Recording"
date:   2015-10-26 15:30:00
---

Many companies need to have their systems used or even managed by people they
don't entirely trust: contractors, outsourced support, peripheral IT staff,
etc. It helps to know what these users or operators were doing on your
systems, or even what they're doing right now, so you can not only prevent
repeated issues, but also stop an incident about to happen.

Government, medical and certain other organizations can be required by law to
collect recordings of user sessions. Support desks also appreciate a way to
look back at what exactly happened, so they don't need to talk through a
user's recollection of events.

Session recordings are typically required to include whatever happens on the
user's screen (be it a text terminal or a GUI), what commands or programs the
user executes, what files he/she reads or modifies. They can also include
hosts the user contacts, what URLs he/she opens in the browser, and other
information.

The interface for auditors needs to allow playing back the sessions the way
users saw them, plus searching for and rewinding to particular session events.
It needs to look like the familiar video playback interface with extra
controls. The searchable events are text entered or appearing on the screen,
entered commands, started applications, accessed files, hosts, URLs, etc. 

Actual recording implementations range from processes on the session's host,
through [jump servers][jump_server], to [application-level gateways][ALG]. The
collected data is stored in a central location, where it can be protected and
easily accessed. The auditor's interface can be implemented as a standalone
application, or a web UI.

There are many commercial, closed-source solutions from such companies as
[BalaBit][solution_balabit],
[Centrify][solution_centrify],
[Citrix][solution_citrix],
[CyberARK][solution_cyberark],
[Dell][solution_dell],
[ObserveIT][solution_observeit],
[Thycotic][solution_thycotic],
[WHEEL Systems][solution_wheel_systems], and others.
Yet, so far there is no integrated open-source solution available, and
that's exactly what we're working on right now, at [Red Hat][redhat] [Identity
Management][redhat_identity_management] group.

We decided to start small, reuse as many components as possible, and make it
easy to take apart and reuse in turn. We're also putting it out in the open,
from the start.


The Stage
---------

Linux already has a very good [audit subsystem][audit] in the form of `auditd`
and the accompanying tools. It can capture most of the required session data.
The audit subsystem can record syscalls done by processes, including files
being accessed or modified and other processes executed. It can also log
network activity when coupled with `iptables`. Auditing can be
[configured][auditctl] using a flexible rule system, allowing fine control of
what exactly is recorded and for which users.

Audit subsystem supports [recording terminal input][pam_tty_audit]. However,
it doesn't support recording terminal output, i.e. what the user actually
sees. Graphical display recording is also far outside its domain.

Any combination of the available logging servers can deliver the session data
(apart from video recordings) to a central location: [journald][journald],
[RSYSLOG][rsyslog], [Fluentd][fluentd], [Logstash][logstash], etc.

There are also a few open-source log aggregation and analysis tools on which
an auditor interface can be built: [Nagios][nagios],
[ElasticSearch][elasticsearch], [Graylog][graylog], [Kibana][kibana].


The Plan
--------

We start with implementing recording terminal I/O and other session data. The
graphical display recording presents more challenges and we'll leave it for
the future.

Adding terminal output recording to audit subsystem both in the kernel and the
user space appears the right thing to do. However, the TTY subsystem, into
which the audit subsystem plugs, is complicated, `auditd` might not be able to
handle higher data bandwidth in a timely manner, and arriving to a solution
satisfying everyone can take a long time. So, for the sake of saving time and
having a solution as early as possible, we chose an easier way, and left the
possible proper integration for the future.

We're implementing a program substituting the user's login shell, starting the
actual user's shell under a PTY and logging everything that passes between the
PTY and the actual terminal. The terminal I/O is joining the audit records in
the system log.

All of the above will be set up by [SSSD][sssd], which in turn can be
controlled by [FreeIPA][freeipa]. That includes setting up audit rules and
substituting the shell with the recording program for specific users and
groups. Still, all the "handles" will be exposed for easy integration into
arbitrary systems.

From there, we forward all the collected data to ElasticSearch via any number
of log servers. We will playback the sessions using a purpose-built
visualisation in Kibana.

We chose ElasticSearch for our storage as the most universal, flexible and
scalable solution, with a good following in the community. The choice of
Kibana as the analyzing and visualisation front-end was then straightforward.


The Progress
------------

We have a proof-of-concept terminal I/O recording package implemented, called
[`tlog`][tlog]. It can already be used to log I/O to ElasticSearch in a
searchable way, and play it back on a terminal. We're working on adding
features and polishing it at the moment. See the demo below, or try it
yourself.

<iframe width="740" height="416"
        src="https://www.youtube.com/embed/F5sOmosbYik?rel=0&amp;showinfo=0"
        frameborder="0" allowfullscreen>
</iframe>

The demo shows recording of a user's session to ElasticSearch with
simultaneous playback, recorded messages as seen in Kibana, and also
highlights the effect of ElasticSearch streaming limitations (and simplicity
of `tlog`'s current implementation, of course).

`Tlog` is still in the early stages of development, there are plenty of items on
the [TODO list][tlog_todo] and we'll be adding more, without doubt. We'll be
glad to receive contributions, suggestions, questions or just comments. Please
[fork][tlog], submit pull requests, issues, or write [me][author] directly!


The Limitations
---------------

Since humans type slowly and the I/O recording needs to be chopped into log
messages, a smoothing delay needs to be introduced to avoid creating a
separate message for each or just a few keystrokes. This introduces latency.
Delivery via a number of log servers, plus ElasticSearch indexing make it
longer. This makes implementing real-time terminal monitoring problematic.

However, most of the time, a real person will not be watching a live stream
from a user's terminal, but instead an automatic system will be watching the
stream for patterns, or a recording will be reviewed later. Plus, other
session data doesn't need to have a smoothing delay and e.g. commands and
accessed files will be delivered faster.

Even though the text on a terminal looks uninterrupted, the actual recorded
I/O stream is often interspersed with control codes: color, font changes and
cursor movements in the output, editing commands and other special characters
- in the input. E.g. while a user enters a specific valid command on a shell
command line, the input stream in the recording will be very different, if the
command was edited before being executed - a common occurrence.

Moreover, since the I/O stream needs to be chopped into messages, a particular
word or phrase can end up split between two messages and thus impossible to
find as a whole using ElasticSearch facilities. For these reasons, the
structured audit records for specific events should be searched instead, and
the I/O stream should be searched only for the data unavailable otherwise, and
the results interpreted carefully.

ElasticSearch is [not a relational database][elasticsearch_modelling] and it
doesn't support joining different data types in search queries as would be
possible with SQL. This means that data structures need to be denormalized,
i.e. a container needs to be stored in a single document with its items, and
data belonging to many containers needs to be copied to each of them. For I/O
recording it means that there can't be a single document describing the
session as a whole, and separate documents describing pieces of the stream.
Instead, overall session data needs to be attached to each piece. This
introduces size overhead to each of the pieces.

However, if the maximum I/O message size is set high enough, this overhead
becomes negligible. The alternative is to join documents client-side, which is
inefficient for smaller documents, but might still be necessary in some cases.
This also concerns multi-part audit events.


The Challenges
--------------

Among the more interesting challenges implementing session recording as
planned are:

* Nobody tried to implement converting audit messages to
  ElasticSearch-compatible JSON, yet. It should be possible and not very hard,
  as audit messages are relatively well-structured.

* So far, ElasticSearch has very limited support for streaming new documents.
  Any stream-oriented client, such as a terminal I/O player, needs to resort
  to inefficient polling, or possibly difficult to use unofficial plug-ins.

* Nobody tried making anything like the terminal I/O playback visualisation we
  plan to implement for Kibana. It is not yet known if it's doable, and while
  theoretically it is possible, it is unlikely to be easy.


The Future
----------

Our target is a turn-key solution: install the FreeIPA client, turn a few
knobs on FreeIPA server, and have sessions recorded for specific users or
groups. Log delivery and storage will have to be configured separately,
though, and we're working on that in another project.

Beside basic playback we plan to have the Kibana visualisation provide
accelerated playback, searching for and rewinding to entered/printed text,
executed processes, accessed files and more. We'll have this contributed
upstream, or available as a plug-in. We're considering embedding the
visualisation into FreeIPA web UI to enable viewing specific sessions with
controls to terminate/suspend them.

[Stay tuned][feed] for more posts on session recording, `tlog` and other tech
topics!

[jump_server]:                  https://en.wikipedia.org/wiki/Jump_server
[ALG]:                          https://en.wikipedia.org/wiki/Application-level_gateway
[solution_balabit]:             https://www.balabit.com/network-security/scb
[solution_centrify]:            https://www.centrify.com/products/server-suite/auditing-compliance/session-auditing/
[solution_citrix]:              http://docs.citrix.com/en-us/xenapp-and-xendesktop/7-6/xad-monitor-article/xad-session-recording/xad-sr-get-started.html
[solution_cyberark]:            http://www.cyberark.com/solutions/by-project/session-monitoring-and-recording/
[solution_dell]:                http://software.dell.com/products/privileged-session-manager/
[solution_observeit]:           http://www.observeit.com/product/platforms/unix-linux-monitoring
[solution_thycotic]:            http://thycotic.com/products/secret-server/features/session-recording/
[solution_wheel_systems]:       http://www.wheelsystems.com/produkty/fudo/
[redhat]:                       http://www.redhat.com/
[redhat_identity_management]:   https://access.redhat.com/products/identity-management-and-infrastructure
[audit]:                        https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/chap-system_auditing.html
[auditctl]:                     http://man7.org/linux/man-pages/man8/auditctl.8.html
[pam_tty_audit]:                http://man7.org/linux/man-pages/man8/pam_tty_audit.8.html
[nagios]:                       https://www.nagios.org/
[elasticsearch]:                https://www.elastic.co/products/elasticsearch
[graylog]:                      https://www.graylog.org/
[kibana]:                       https://www.elastic.co/products/kibana
[tlog]:                         https://github.com/spbnick/tlog
[script]:                       http://man7.org/linux/man-pages/man1/script.1.html
[sudoreplay]:                   http://www.sudo.ws/man/1.8.14/sudoreplay.man.html
[asciinema]:                    https://asciinema.org/
[showterm]:                     http://showterm.io/
[journald]:                     http://www.freedesktop.org/software/systemd/man/systemd-journald.service.html
[rsyslog]:                      http://www.rsyslog.com/
[fluentd]:                      http://www.fluentd.org/
[logstash]:                     https://www.elastic.co/products/logstash
[sssd]:                         https://fedorahosted.org/sssd/
[freeipa]:                      https://www.freeipa.org/
[elasticsearch_modelling]:      https://www.elastic.co/guide/en/elasticsearch/guide/current/modeling-your-data.html
[tlog_todo]:                    https://github.com/spbnick/tlog/blob/master/todo.txt
[author]:                       mailto:spbnick@gmail.com?subject=tlog
[feed]:                         /feed.xml
