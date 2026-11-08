# The wipe cycle

Every world ends. A new world's cycle starts the first time it is hosted and
lasts `ASHWEND_WIPE_DAYS` days (30 unless you set it). It counts the server's
running time, not the calendar: a server that is down for a week comes back a
week less far into its cycle.

## What players see

A comet in the sky, growing over the whole cycle. Looking at it shows the time
left, and so does the map: whole days, then hours and minutes on the last day,
then minutes and seconds in the last hour. The server browser shows each
server's place in its cycle ("Day 3 of 30").

At the end the comet comes down and every player dies, online or asleep. Nobody
can respawn into that world.

## What the server does

Shortly after the impact the server disconnects everyone with "The world has
ended. A new one has begun on this server.", starts the next world in place with
a new seed and writes it over `world.save`. Your admins and bans carry over;
nothing else does. The process, the port and the listing stay, and players
rejoin as usual. You do not need to do anything.

## The length is fixed for a world's life

- Leave `ASHWEND_WIPE_DAYS` unset and an existing world keeps the length it
  started with.
- Set it to a different length than the saved world's and the server refuses to
  start, with an error naming the saved length, rather than move a live world's
  end. With `restart: unless-stopped` the container keeps restarting into that
  error until you put the setting back or remove it.
- To change the length, start a new world: stop the server, move `world.save`
  aside, set the new length, start it again.

## Testing it

The admin panel's **World** tab can bring the comet in early. That is real: the
world ends and is replaced a few minutes later, so try it on a throwaway world.

To watch the end without losing the world, rehearse it: the comet's **Aim** in
the admin panel lets you pick a spot (left click sends the comet there ten
seconds later, right click stands it down), and its **Reset** puts the cycle
back as it was and lets the dead respawn. A rehearsal plays out everything but
the wipe, and still kills whoever is on the ground near it, so warn your players
first.
