---
layout: post
title:  "Making sense of libtevent"
date:   2016-07-04 18-30-00
issue:  6
---
I'm working on integrating [tlog][tlog] with [SSSD][sssd] and am finding a
specific part of SSSD hard to understand. Among other things, it uses the
[tevent][tevent] library, made by Samba developers to solve particularly hairy
problems of asynchronous programming. While its file descriptor, signal, and
timer event handling is relatively straightforward, I found its
"[requests][tevent_requests]" feature confusing and hard to reason about.

My guess is it uses some CS terminology I'm not familiar with (not having
finished my university studies), and without that background it hardly makes
sense to me. Anyway, following the old adage "if you want to learn something,
teach it", I'll try to explain what the terminology used by the "requests"
part of tevent means. It will definitely help me, and I'll be glad if someone
else finds it useful.

**DISCLAIMER**: This is just my opinion and my attempt to understand tevent
better. I'm only starting to make sense of it, and can make serious mistakes.
Don't take the below as facts or the best approach at understanding tevent,
use your own judgement. For that matter, I'll be glad to receive comments,
corrections and suggestions for better approaches.

Tevent represents the "event loop" approach to asynchronous programming: it
implements an "[event context][tevent_context]" - basically, a list of
event/callback pairs, functions to create and modify it, and a function
waiting for any of the events in the list, and invoking their corresponding
callbacks when they happen. An [event][tevent_events] can be a file descriptor
state change, a signal, a timer firing, or a synthetic "immediate" event.

That is all fine and dandy while you have just a few of those, plus basic
processing in between - you can have all the code spread over callbacks,
sharing a bit of global state. Yet, things get hairy when you need to e.g.
take an input from one socket, wait for some input from another, send the
result to a third, and track this process for tens of clients at the same
time. That needs a good deal of shared data and progress tracking, and rolling
custom code each time you implement something like that quickly gets boring.

This is where tevent "requests" come in (named "tevent_req" in the code). A
"request" is just an abstraction of a piece of work done (i.e. code executed)
on a piece of data. It can be any code and any data whatsoever, working for
whatever purpose. You can give it the data to work on, a function to call when
it's done, and afterwards you can ask it for the results of its work. There
are more housekeeping operations and data available, but that's basically it.

And here we come to the first confusion: the "request" name. It's not actually
a request. There is no entity being requested anything. A more suitable name
would perhaps be simply "work" ("tevent_work" in the code).

The library provides the following major functions to deal with requests:

* tevent_req_create - allocate and initialize a request (normally done from
  within the request code, talloc is used to destroy),
* tevent_req_set_callback - set a function to call when request is done
  (normally done from outside the request code, by the user),
* tevent_req_done - mark request done, and call the callback specified with
  tevent_req_set_callback (normally done from within the request code),

So far so good. However, the "tevent_req_set_callback" operation doesn't give
a clue as to what that callback does or when it is called, plus you can
actually specify other callbacks, which makes it more confusing. Sure, it is
the only callback set by the user normally, but that is not clear from the
name, nor even from the documentation. Perhaps this operation would have
been better named "set_done_fn", or, in full, "tevent_work_set_done_fn".
That would also match the pattern used to name other callback-modification
functions.

The library documentation describes how to implement a request. It says, you
need to define a function with "send" on the end of its name, which would
accept data to work on, create a request using "tevent_req_create", attach the
data to it, and would start the actual work. You also need to define a
function with "recv" (meaning "receive") on the end of its name, which would
extract the work results from the request.

This convention follows the idea of a work request being "sent" somewhere for
execution and results being "received". However, nothing is actually being
sent to, or received from anywhere. This is particularly confusing in code
actually sending/receiving something, e.g. network servers and clients.

If the "request" was named "work" instead, then these two functions could be
named "begin", and "reap", or "collect". Let's say we had a task of reading a
structure describing a user from a socket, and have named the operation
"user_read". Then, the corresponding functions would be named:

    user_read_begin
    user_read_collect

Which appears to me more obvious than:

    user_read_send
    user_read_recv

So how does all this help asynchronous programming? The requests help keep the
idea of a process between all those callbacks for various events, since
otherwise those are just functions called from the main loop.

E.g. in an actual request's "send" (or "begin") function you would create a
new file descriptor event waiting for input on a socket, and give it a
callback, along with a reference to the request you just created. That
callback would get some data, and then re-add the event, until everything
necessary was read, when it would call "tevent_req_done".

Furthermore, requests can be nested, i.e. a larger job can be split into
several pieces. No special API is used there, but rather requests just start
new requests in the course of their work, or wait for completion of others.

Overall, tevent requests is a good approach at organizing complicated
processing within an event loop system. It would be even better if the API
used terms closer to the actual purpose and function, and not some (archaic?)
CS terminology requiring imagining what's not there before you can understand
it.

The event loop approach by itself is very verbose, unfortunately, and requests
play to the same tune: separate function to start work, to collect results,
event callbacks in between, callback for completion, dedicated types to carry
work data between them - all add up to a lot of code which has little to do
with the actual problem being solved.

All that makes me wish for the directness and simplicity of coroutines. C
language doesn't support them, but there are multiple implementations, some of
which are quite good and even portable. E.g. [picoro][picoro] from Tony Finch,
and a well-explained (but non-portable) attempt by Yossi Kreinin: [Coroutines
in one page of C][yossi_kreinin_coroutines].

If I have to start something complicated and asynchronous in C without using
threads or separate processes, I'll try coroutines, but for now I have to
understand SSSD and tevent.

[tlog]: http://scribery.github.io/tlog/
[sssd]: https://fedorahosted.org/sssd/
[tevent]: https://tevent.samba.org/
[tevent_context]: https://tevent.samba.org/tevent_context.html
[tevent_events]: https://tevent.samba.org/tevent_events.html
[tevent_requests]: https://tevent.samba.org/tevent_request.html
[picoro]: http://dotat.at/cgi/~fanf/dotat/~fanf/dotat/git/picoro.git
[yossi_kreinin_coroutines]: https://www.embeddedrelated.com/showarticle/455.php
