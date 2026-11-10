# Running a busy server

A server with a few dozen players needs none of this: start it and leave it
alone. This page is for servers that expect a hundred players or more, where
the machine and a handful of system settings decide whether a busy evening
stays smooth.

The numbers here come from load tests with scripted players on 50 to 100 ms
connections. Scripted players never rest, so read them as a busy evening, not
an average one, and leave yourself room.

This page describes servers newer than 0.29.0-beta.3. Older ones carry far
fewer players, do not ask for larger network buffers and do not print the
lines quoted below.

## Choosing the machine

- **A fast core matters most.** The server does most of its work on one
  thread, so the speed of a single core sets how many players it carries. Two
  or three more cores take the rest off it (wildlife, projectiles, sending
  packets, the system's own network work), and beyond four they add nothing.
- **Dedicated cores beat shared ones.** On a shared-vCPU instance another
  customer's load arrives as stutter you cannot fix. If `top` shows a steady
  `st` (steal) value above a few percent, move to dedicated vCPUs.
- **Memory:** about 250 MB empty, plus what the world's bases take, plus about
  3.5 MB for each player online.
- **Bandwidth:** about 100 kbit/s down to each player and 20 kbit/s up from
  them. When many players fight in one place, the download side rises to three
  to five times that for as long as it lasts.

As a guide for `ASHWEND_MAX_PLAYERS`:

| Machine | Players | Memory | Peak bandwidth |
| --- | --- | --- | --- |
| 1 to 2 shared vCPUs | 80 to 120 | 2 GB | 60 Mbit/s |
| 4 shared vCPUs | 150 to 200 | 4 GB | 100 Mbit/s |
| 4 dedicated vCPUs | 200 to 300 | 8 GB | 150 Mbit/s |
| A current desktop-class dedicated server | 400 to 500 | 8 GB | 250 Mbit/s |

Past what the machine carries, the server slows down rather than stops: the
world runs behind and everyone feels it. Set the limit below that point.

## System settings

These are settings of the Linux host, not of the server. Set them on the
machine itself: a container shares the host's limits, and on a game panel
they are your hosting provider's to set.

### Network buffers (the one that matters)

Linux gives every UDP socket about 200 kB of buffer in each direction, and a
program may ask for more only up to a ceiling. With a few hundred players in
one place, one moment's packets no longer fit. The server then holds packets
back for a few milliseconds and carries on, which players feel as lag in big
fights. Raise the ceilings and the server takes the room it needs:

```bash
sudo sysctl -w net.core.rmem_max=8388608 net.core.wmem_max=8388608
```

To keep the setting across restarts of the machine, put it in a file:

```bash
printf 'net.core.rmem_max = 8388608\nnet.core.wmem_max = 8388608\nnet.core.netdev_max_backlog = 5000\n' | sudo tee /etc/sysctl.d/90-ashwend.conf
sudo sysctl --system
```

Restart the server afterwards: it asks for its buffers once, when it starts.
Its log says what it was given:

```
UDP endpoint bound to 0.0.0.0:7777 (buffers: 16384 kB out, 16384 kB in)
```

(Linux reports twice what was asked for.) A value near 400 kB means the
ceilings are still at their defaults.

`net.core.netdev_max_backlog` in the file above is the queue of packets that
have arrived and wait for the system to hand them on. The default of 1000 is
enough for a hundred players; 5000 leaves room for several hundred.

### Do not cap the container's CPU

A CPU limit on the container (`--cpus`, `cpus:` in a compose file, a panel's
CPU limit below 200%) does not make the server use less: it makes the system
freeze the server for the rest of each 100 ms period once the allowance is
spent, which players see as rubber-banding. Give the server at least two full
cores, or no limit.

### Keep the server out of swap

A server that has been swapped out stalls every time it touches that memory.
Give the machine enough memory that it never swaps, and on a machine that
has swap enabled, make it a last resort:

```bash
sudo sysctl -w vm.swappiness=10
```

### Host networking, for the largest servers

With published ports (`-p 7777:7777/udp`), every packet passes through
Docker's address translation. That costs little, and for most servers it is
the right setup. A server with several hundred players can skip it by using
the host's network directly:

```yaml
services:
  ashwend:
    network_mode: host
```

Remove the `ports:` section when you do; the server then listens on the
machine's own ports 7777 and 27016. Your firewall must still let both through.

### Firewalls

A firewall that tracks connections (most do) keeps one entry for each player.
The defaults are sized for far more than a game server needs. If you have
lowered `nf_conntrack_max` for some other reason, keep it well above your
player limit.

### On your own hardware

On a dedicated machine, set the CPU to run at full speed instead of scaling
its frequency with load, which otherwise adds a delay each time the server
wakes:

```bash
sudo cpupower frequency-set -g performance
```

Cloud instances manage this themselves.

## Is the server keeping up?

Set `ASHWEND_SERVER_STATS` to a number of seconds and the server prints a
line that often:

```
stats online=212 ... | tick_hz=20.0 frame_ms mean=14.60 p50=13.90 p95=24.10 p99=31.90 max=35.50 | phase_ms ... | idle_us=310 busy_pct=45.9 rss_mb=677
```

- **`busy_pct`** is how much of its main thread the server is using. Under 75
  there is room to spare. At 100 the server is behind.
- **`frame_ms`** is how long each step of the world takes. The world steps 20
  times a second, so a step has 50 ms. `p99` above 50 means occasional
  hitches; `mean` above 50 means constant ones.
- **`tick_hz`** should read 20.
- **`rss_mb`** is the memory the server is using.

In the log, a line like

```
UDP socket full 37 times and 0 packets dropped in the last 10 s
```

means the network buffers are too small for the number of players: raise them
as described above. A few such lines during a big fight are harmless; a steady
stream of them, or any dropped packets, is not.

## What to do when it is full

- Lower `ASHWEND_MAX_PLAYERS` to what the machine carries comfortably.
- Move to a machine with a faster core. More than four cores will not help.
- Check `st` in `top` on a shared instance, and that the machine is not
  swapping (`free -m`).
