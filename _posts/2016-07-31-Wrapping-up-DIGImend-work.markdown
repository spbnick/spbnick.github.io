---
layout: post
title:  "Wrapping up DIGImend work"
date:   2016-07-31 15-00-00
issue:  7
---
In short: I'm wrapping up my involvement in [DIGImend project][digimend]
within two months.

It was long apparent, but now I would like to state that officially, and stop
misleading users and tablet owners: I no longer have time nor energy to do the
work required. I'm sorry for all the false hope I may have given people, and
the promises I've broken in the meantime.

Starting today, I'm stopping all research on specific tablet interfaces and
protocols required for implementing drivers. I.e. I'm not going to respond to
any diagnostics, or requests to make new tablets work. I'm not going to
support users, or investigate their problems either. However, I will still be
reviewing and accepting patches, including ones already submitted.

For the following two months, until the end of October, I'm focusing first on
syncing the [DIGImend kernel drivers][digimend-kernel-drivers] with the
upstream and writing HOWTOs on how to diagnose problems with tablets and how
to reverse-engineer tablet protocols.

I will also be available for coaching and support of any able person willing
to take my place, as a top priority. Write to [me][me], if you would like to
step in, know C well, have experience with kernel and system programming and
interest in reverse-engineering USB devices, plus you have the patience to
deal with users of widely-varied experience and background.

Lastly, if anybody wishes to employ me to continue working on DIGImend, I'm
open to the offers, and I will gladly continue working on the project,
provided I'm appropriately compensated.

After those two months, in November, I'm going to stop all work on DIGImend,
except, if a replacement maintainer isn't found, I will still be reviewing and
accepting patches as time allows. Otherwise, I will provide no support or help
with any user issues. I will also shutdown the DIGImend maillists, preserving
the archives, if possible, unless the new maintainer (if any) wishes
otherwise. The GitHub issues will remain open for discussions, all the code
and the website will remain online.

DIGImend history
----------------

In August 2005 my future wife and her friend gave me a [Genius MousePen
8x6][genius_mousepen_8x6] as a birthday present. It almost worked in Linux,
but not quite. After looking for a while I found an Aiptek X11 driver hacked
by Jan Horak (then a student at Brno University of Technology) to work with
Genius WizardPen tablets. The driver was named WizardPen and hosted at the
university's web-server. It didn't work that well with my tablet, so I started
hacking. Eventually we made it work with mine and a bunch of other tablets. I
also added a calibration utility. We supported it for a while on the forum Jan
setup, and did a couple releases.

The driver worked by ignoring the fact that the kernel didn't understand these
tablets well and reported garbage to the user-space, and simply tried to make
sense of that. This approach was flawed, but effective, and to this day the
[WizardPen driver][wizardpen], now supported by Ubuntu community, is a go-to
for desperate tablet owners, as it sometimes works when there's no support
otherwise.

I thought that such approach is limited, and decided to instead work at the
root of the problem: the kernel, relying on the catch-all evdev X11 driver in
user-space. While investigating a bunch of tablets I found that most of the
time the problem was in how cheap tablets described themselves. They
misinterpreted the USB HID standard, apparently tweaking the tablets only so
that they would pass the Windows certification/tests and often relying on
their custom Windows drivers to make them actually work. Otherwise, their
protocols were very straightforward, and if I could fix their report
descriptors, they would just work.

So, in July 2008 I started my own project and named it DIGImend (after
"digitizer mending", "digitizers" being the old name for graphics tablets). I
began with making a couple diagnostics tools and basic kernel and evdev
patches. In 2009 I started building an [editor tool][hidrd] for HID report
descriptors - data structures provided by input devices to describe their
protocols. The plan was to make the kernel ignore the device's descriptors and
feed it ones that I fixed instead. Eventually the tool started working, I
contributed requisite fixes to the kernel and started working on the tablets.

I didn't have money to buy them, so I went around computer hardware shops in
my city (then Saint-Petersburg, Russia) asking for permission to check how
tablets on display worked, and surprisingly most of the shops were fine with
that. So I had a bunch of tablets reverse-engineered and supported that way.
Later, after moving to Finland, I bought some tablets with user's help and had
Waltop send me two of theirs. I put up a wiki on the sourceforge.net hosting
and a couple people joined, helped me fill it in, and helped support the
users.

I contributed patches to the upstream kernel and also released patch packages,
followed by patched kernel packages for Ubuntu. At some point sourceforge.net
abandoned MediaWiki support, forcing people to maintain their own installs,
which didn't work that well, so I reworked and moved the project website to
GitHub. After out-of-tree kernel HID drivers became possible, and report
descriptor replacement started working for them, I made the DIGImend kernel
drivers package, which has now acquired a healthy popularity.

However, in the past year or so I found myself occupied otherwise: I started
getting more involved with my hobby of embedded programming and electronics,
and then I had eight months of intensive Finnish courses, followed by
continued self-study. My daughter is also growing up, and I find myself
needing to spend more time with her. All-in-all, I no longer have time nor
energy required to keep up with the development in the Linux input stack, new
tablets being produced, and the multitude of user requests coming in. I failed
people expectations many times already, and now it's time to stop.

Thanks
------

I would like to thank everyone who cooperated, and helped me with this
project. I'm trying to remember as many people as I can, but please forgive
me if I forgot you, you have my thanks too! So, in no particular order:

* Jan Horak, who started the WizardPen driver development and with whom I
  worked to improve it;
* Peter Hutterer, the maintainer and developer of the evdev X11 driver, a
  Wacom driver developer (among other things), and a fellow Red Hat engineer,
  who accepted my patches and helped me understand how things worked;
* Jiří Kosina, the maintainer of the HID subsystem in the Linux kernel (among
  other things), for reviewing my patches and explaining things to me;
* Greg Kroah-Hartman, maintainer of the USB subsystem in the Linux kernel and 
  usbutils developer (among many other things), and simply a wonderful person,
  for also reviewing my patches and helping me learn;
* Jason Gerecke, a Wacom driver developer, for helping me understand things
  about the Wacom driver and contributing to hidrd;
* Ping Cheng, a Wacom driver developer, for helping me understand the
  Wacom driver;
* Benjamin Tissoires, an input stack developer, and a fellow engineer at Red
  Hat, for helping me with the Wacom driver too, and contributing to the
  tablet drivers, both upstream and in the DIGImend kernel drivers package;
* Martin Rusko, who did initial reverse-engineering of Huion tablets, hacked
  on the driver and tested it;
* Yann Vernier, for contributing the Polostar tablet driver;
* Waltop International Corporation, who sent me two tablets and provided some
  information on them;
* Masaru Hirano from Neoblast Inc., Japan for providing information on Ugee
  tablets they sell;
* Yiynova Europe BV, who did initial Yiynova tablet support hacking and
  then testing;
* Favux, for helping support users and contributing wiki contents and HOWTOs;
* Viktoria S., for helping support users and contributing HOWTOs;
* the incredible number of tablet owners, who meticulously collected
  diagnostics, strived to learn Linux, and tested the drivers, for their
  efforts, patience and gratitude;
* other engineers who contributed small and sometimes not so small fixes to
  all the project software and the website;
* all the people who helped, and still help fellow tablet owners on the
  maillist and in the GitHub issues;
* all the shop assistants who let me play with the tablets in the stores.

[digimend]: http://digimend.github.io/
[digimend-kernel-drivers]: https://github.com/DIGImend/digimend-kernel-drivers
[me]: mailto:spbnick@gmail.com
[genius_mousepen_8x6]: http://digimend.github.io/tablets/UC-Logic_WP8060U/
[wizardpen]: https://launchpad.net/wizardpen
[hidrd]: https://github.com/DIGImend/hidrd
