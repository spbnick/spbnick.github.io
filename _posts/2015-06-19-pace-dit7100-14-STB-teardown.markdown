---
layout: post
title:  "Pace DIT7100/14 STB teardown"
date:   2015-06-19 16-06-58
issue:  3
---
Elisa, a Finnish telecom company is phasing out one of its older STBs (set-top
boxes) and they are already showing up in local reuse centers
([Kierrätyskeskus][kierratyskeskus], an analogue of US and Canada's Goodwill
Stores) and flea markets. The particular model I happened to have is Pace
DIT7100/14. These seem to be relatively powerful machines, on par with the
popular [Raspberry Pi Model B][rpi_model_b], but cheaper, selling at about 15
EUR, and I suspect can be acquired for less. So, they can theoretically be put
to a few good uses beside their primary purpose.

[![Top][top_thumb]][top] [![Bottom][bottom_thumb]][bottom] [![Rear][rear_thumb]][rear]

I haven't found any evidence of anyone (beside Pace) successfully putting
Linux on those, and so I cracked mine open to see if it is at least possible.
Unfortunately, I wasn't able to discern much, or even find the serial console
port yet.

The STB has a number of ports. Front: 1 USB socket, 1 infrared receiver. Back:
the tuner's RF in and out, SPDIF optical audio, analog audio L/R, HDMI, SCART,
Ethernet's RJ-45, another USB socket and 12V DC input.

Removing the top cover we can see the main board and the controls board
mounted above it.

[![Top (open)][top_open_thumb]][top_open]

The controls board contains the soft power button, status LED, IR receiver,
front USB socket. Also, amusingly, [TI's CC2533][ti_cc2533] - a 2.4GHz IEEE
802.15.4/ZigBee SoC, along with its 32 MHz oscillator, even though the STB
shipped with only an [IR remote][ir_remote].

[![Controls board (top)][controls_board_top_thumb]][controls_board_top]
[![Controls board (bottom)][controls_board_bottom_thumb]][controls_board_bottom]

Moving onto the most interesting part - the main board, we can find the CPU
under a heatsink. Removing the heatsink reveals the CPU being Broadcom's
BCM7406. I couldn't find much information on it, but I suspect it's closely
related to [BCM7405][broadcom_bcm7405] for which
[some information][phd_wiki_bcm7405] can be found on the wonderful
[Programmer's Hardware Database wiki][phd_wiki].
Including the [pinout][phd_wiki_bcm7405_pinout]. LinuxGizmos.com also has an
[article][linuxgizmos_bcm7405] in its archive introducing 7405 with the
function block and reference design architecture diagrams.

[![Main board (top)][main_board_top_thumb]][main_board_top]
[![Main board (bottom)][main_board_bottom_thumb]][main_board_bottom]
[![CPU (without heatsink)][cpu_thumb]][cpu]

Going further, the main board contains four [1Gb DDR2 SDRAM chips from
Samsung][k4t1g164qf], two on each side. This amounts to 512 MB of RAM.
There's also what seems to be a [Spansion 1Gb NOR flash chip][spansion_flash]
labeled GL01GP11FFSS8. I.e. there's only 128 MB of storage on board, which is
not too much, but USB ports can help. Then there's [CXD2820R][sony_cxd2820r] -
a DVB-T2 demodulator from Sony, and [STV6417][st_stv6417] - an A/V switch from
ST. The rest of the chips seems to be logic glue or power-related.

As I said above, I wasn't able to find the serial console. I poked around the
connector pads to the left of the CPU with a scope during boot-up and didn't
find anything. The major suspect - a four-hole space for a connector had two
grounds in the middle and nothing on the sides, it seems. The 23- and 15-pad
spaces were also mostly silent with some clock on one or two pads.

From reading about getting serial console on other similar devices I gathered
they can be actually routed to some outside connectors and/or require some
buttons pressed to be activated during boot. I tried the only button - Power
and couldn't find anything on the unpopulated pads anyway, but I haven't tried
the connectors. I'll try looking at the pinout and muxing and see if UART can
be shared with some of the pins possibly used by the connectors directly.

I haven't tried locating JTAG yet, mostly because I don't have an EJTAG cable.
It can also be disabled. Lastly, it is quite possible that the device was
Tivoized in some way, making any attempts to hack it useless, but I have to
hope for the best until I'm able to figure it out.

If you have any ideas of where the serial or JTAG can be on this board, or if
this STB is Tivoized, please write to me, or leave a comment on my [Google+
post][gplus_post].

[kierratyskeskus]: http://www.kierratyskeskus.fi/
[rpi_model_b]: https://www.raspberrypi.org/products/model-b/
[top_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/top.thumb.jpg "Top"
[top]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/top.jpg
[bottom_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/bottom.thumb.jpg "Bottom"
[bottom]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/bottom.jpg
[rear_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/rear.thumb.jpg "Rear"
[rear]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/rear.jpg
[top_open_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/top_open.thumb.jpg "Top (open)"
[top_open]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/top_open.jpg
[controls_board_top_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/controls_board_top.thumb.jpg "Controls board (top)"
[controls_board_top]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/controls_board_top.jpg
[controls_board_bottom_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/controls_board_bottom.thumb.jpg "Controls board (bottom)"
[controls_board_bottom]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/controls_board_bottom.jpg
[ti_cc2533]: http://www.ti.com/product/cc2533
[ir_remote]: http://www.uei.com/product/europe-middle-east-africa/widor
[main_board_top_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/main_board_top.thumb.jpg "Main board (top)"
[main_board_top]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/main_board_top.jpg
[main_board_bottom_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/main_board_bottom.thumb.jpg "Main board (bottom)"
[main_board_bottom]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/main_board_bottom.jpg
[cpu_thumb]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/cpu.thumb.jpg "CPU (without heatsink)"
[cpu]: /assets/2015-06-19-pace-dit7100-14-STB-teardown/cpu.jpg
[broadcom_bcm7405]: http://www.broadcom.com/products/set-top-box-and-media-processors/cable-%28stb-and-media-processors%29/bcm7405
[phd_wiki]: http://hwdb.mipt.cc/
[phd_wiki_bcm7405]: http://hwdb.mipt.cc/BCM7405
[phd_wiki_bcm7405_pinout]: http://hwdb.mipt.cc/BCM7405/Pinout
[linuxgizmos_bcm7405]: http://archive.linuxgizmos.com/65nm-stb-on-a-chip-runs-linux/
[k4t1g164qf]: http://www.samsung.com/global/business/semiconductor/file/product/ds_k4t1gxx4qf_rev12-0.pdf
[spansion_flash]: http://www.spansion.com/Support/Datasheets/S29GL-P_00.pdf
[sony_cxd2820r]: http://www.sony.net/Products/SC-HP/cx_news_archives/img/pdf/vol_60/cxd2820r.pdf
[st_stv6417]: http://www.st.com/web/catalog/mmc/FM131/SC631/PF174515
[gplus_post]: https://plus.google.com/+NikolaiKondrashov/posts/SVHbiedQPhy
