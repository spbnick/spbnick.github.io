---
layout: post
title:  "Letting go of DIGImend"
date:   2024-11-03 17-50-00
excerpt_separator: <!--more-->
issue:  12
---
I'm finally letting go of DIGImend (this time for good).

<!--more-->

I have barely touched [DIGImend](https://digimend.github.io/) since the
beginning of the pandemic. My last considerable effort was attempting to
implement support for [user-space HID device
drivers](https://github.com/DIGImend/digimend-userspace-drivers/)
using libusb and Lua. That curdled upon encountering another convoluted way
Wacom driver stack works, which wouldn't work for other drivers (hardcoded USB
VID:PID again). Plus there was no way to make Wayland input stack work well
with that anyway, at that moment.

Shortly afterwards, at LPC 2022 in Dublin, I talked to [Benjamin
Tissoires](https://github.com/bentiss) about his work and [his
talk](https://lpc.events/event/16/contributions/1364/) on
[HID-BPF](https://docs.kernel.org/hid/hid-bpf.html). The aim of that project
is similar: to make it easier to write HID device fixups, including for
graphics tablets. Albeit using [BPF](https://ebpf.io/) (duh!). That is of
course more complex to write than Lua, but still easier than a full-on kernel
driver in C. Plus Benjamin is working full-time on the input stack, and he has
much better support for taking it to a good place (which he continues doing).

In 2022, [José Expósito](https://joseexposito.me/) has started working on the
HID and input subsystem in the Linux kernel, and he has brought some welcome
changes and support for more tablets to both upstream and DIGImend. He also
synced the DIGImend kernel drivers to upstream. I gave him admin permissions
on the drivers repo, and he worked on it and helped some user for a while
afterwards. Thank you, José!

However, the message from him, Benjamin, and I suppose [Peter
Hutterer](http://who-t.blogspot.com/) too, is to use HID-BPF as much as
possible from now on, instead of creating new or modifying existing kernel
drivers written in C. IIRC, at LPC 2024 they said that it still cannot do the
weird switch-to-proprietary-mode USB requests, but I suspect they have a
solution in mind.

However, I think I exhausted all the energy and interest I had for DIGImend
now. Having [started it in
2008](/2016/07/31/Wrapping-up-DIGImend-work.html#digimend-history),
I think I have done a good deal to help the community in return for all the
awesome Open-Source software I'm getting to use.

With this, I'm closing my Patreon, Liberapay, and Buy Me a Coffee accounts I
created to support my work on the project. Unfortunately it was never enough
to cover my time, but it was good while it lasted. I'm grateful to everyone
who chipped in and sent me whatever they could through those services, and
also for advice on growing the engagement and getting more support, from some
of them, which I largely couldn't follow through on.

I hope the Linux input stack continues evolving and eventually gets a
re-engineering it needs and deserves, so things are fun again, even for people
not paid to work on it.

I'll keep the project website and the GitHub organization in place, of course,
but I will not be the one helping people out, or working on them. Thank you
everyone!
