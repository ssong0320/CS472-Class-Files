1.

Manual time changes by software or people can cause the clock to be ahead by 30s. A very common cause is that
something (an installer, a VM, or even some person in the Date & Time settings) manually set the clock. That kind
of change makes it very possible for the clock to be 30 seconds ahead. To investigate, Check system and time logs
around the window for messages like time changed/stepped/set, and look for entries from the system time manager
that might suggest manual adjustment. Look at recent installs, maintenance jobs, or VM activities to identify the
culprit.Another potential cuase is the automatic timesync isn’t actually running or can’t reach the internet time
servers. A one-time “Sync now” fixes the clock only for that moment. if the background set time automatically
feature is off or for some reason blocked by your network, your clock can start being inaccurate. Check that your
device has a stable connection, and that time syncing isn’t blocked. Trying a different network can also quickly
tell you if that’s the issue. A third cause can be sleep/hibernate or pausing a virtual machine. Laptops and VMs
can wake or resume with the clock slightly off. A VM’s host can push its own time onto the guest while the guest
is also trying to sync online. Those two can clash and create a sudden jump. To check, notice whether the error
appears right after waking from sleep or resuming a VM. To fix, pick one source for the time, either let OS keep
time automatically from the internet or rely on the host.

2.

I tested my client five times against a nearby server (us.pool.ntp.org) and a farther one (europe.pool.ntp.org).
The nearby server averaged 57.9 ms round-trip delay, with mean offset 57.9 ms and dispersion 28.9 ms. The farther
server averaged 95.4 ms delay, with mean offset 65.0 ms and dispersion 47.7 ms. The results show that the farther
the packets travel, whether it's more miles or more routers or both, the longer they take and the more they vary
from try to try. NTP’s math cancels some delay, but it can’t remove random variation or one direction moving
slower than the other, which grow with distance and network congestion. That’s why a closer server can yield a
tighter, more reliable result even if its clock isn’t “the best” time source.

./ntp-client -s us.pool.ntp.org \*5

1: Round Trip Delay: 48.3 ms, Offset: 59.2 ms, Dispersion: 24.1 ms
2: Round Trip Delay: 62.8 ms, Offset: 46.7 ms, Dispersion: 31.4 ms
3: Round Trip Delay: 53.8 ms, Offset: 62.7 ms, Dispersion: 26.9 ms
4: Round Trip Delay: 42.8 ms, Offset: 57.3 ms, Dispersion: 21.4 ms
5: Round Trip Delay: 81.8 ms, Offset: 63.8 ms, Dispersion: 40.9 ms

Mean Round Trip Delay: 57.90 ms
Mean Offset: 57.94 ms
Mean Dispersion: 28.94 ms

./ntp-client -s europe.pool.ntp.org \*5

1: Round Trip Delay: 88.8 ms, Offset: 60.2 ms, Dispersion: 44.4 ms
2: Round Trip Delay: 95.3 ms, Offset: 64.9 ms, Dispersion: 47.7 ms
3: Round Trip Delay: 104.6 ms, Offset: 69.6 ms, Dispersion: 52.3 ms
4: Round Trip Delay: 91.0 ms, Offset: 62.7 ms, Dispersion: 45.5 ms
5: Round Trip Delay: 97.0 ms, Offset: 67.4 ms, Dispersion: 48.5 ms

Mean Round Trip Delay: 95.36 ms
Mean Offset: 64.96 ms
Mean Dispersion: 47.68 ms

3.

A "what time is it" protocol does not work well because by the time the response reaches you,
it is already old. Networks add delay and that delay changes from moment to moment. If you just
set your clock to the timestamp you received, you’ll be off by at least that one way delay,
and possibly more if the path to you is slower than the path from you, or if the server paused before
replying. NTP is able to fix this by exchanging four timestamps. With the four timestamps, the client can estimate
the round trip time, subtract the server's processing time, get the network delay, and estimate offset.
Mathematically, it uses T1 to T4 to compute delay, which is (T4−T1) − (T3−T2), and offset which is((T2−T1) + (T3−T4)) / 2, which cancels most transport latency and separates network delay from true clock difference.
