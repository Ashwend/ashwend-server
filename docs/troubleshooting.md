# Troubleshooting

Start with the log: `docker logs ashwend` (or `docker compose logs`).

## The server is not in the server list

- **UDP 27016 is not reachable.** Steam finds and lists servers through this
  port. Forward it as well as 7777, both UDP. A server reachable on 7777 alone
  works through Direct connect but is never listed.
- **On a game panel, the query port is not one of the server's allocations.**
  A panel only lets allocated ports through. Add a second port to the server and
  set Steam Query Port to its number ([Game panels](game-panels.md)).
- **The ports were remapped.** The server tells Steam the ports it listens on,
  so `-p 7778:7777/udp`-style mappings send players to the wrong place. Publish
  the same numbers on both sides (see [Configuration](configuration.md)).
- **No outbound internet.** The server signs in to Steam at start; without a
  route out it cannot be listed and cannot check players.
- **It is a `no-auth` server.** Those are never listed; join them with Direct
  connect.

## Players are turned away

A server admits only players on exactly its own version. If the browser shows
"needs x.y.z" on your server, you and your players are on different builds:

- Players on Steam's beta branch need the `beta` tag (or the matching version),
  players on the default branch need `latest`.
- After an update, Steam may take a while to reach every player. Pull the new
  image when your players have the new version.

## The server lags when it is busy

See [Running a busy server](performance.md): how to tell whether the machine
or its settings are the limit, and what to change.

## The server stops right after starting

- **`Failed to load module '~/.steam/sdkarm64/steamclient.so'`**: the machine is
  ARM. The server needs an x86-64 machine.
- **A `no-auth` server refuses to start**: it will not listen on all addresses
  unless `ASHWEND_ALLOW_UNAUTHENTICATED_LAN=1` is set. Read the warning in
  [Configuration](configuration.md) first.
- **An error about the wipe cycle's length**: `ASHWEND_WIPE_DAYS` differs from
  the saved world's. Remove the setting or start a new world (see
  [the wipe cycle](wipe-cycle.md)).

## What the server connects to

- **Steam**, always: to sign in, to check each player's ticket and to list the
  server.
- **NOAA's forecast server** (`nomads.ncep.noaa.gov`), for the planet's weather.
  Optional: `ASHWEND_WEATHER_FEED=off` stops it.
- **A crash report to the developer.** If the server crashes, a release build
  may send one report about the crash (to PostHog, in the EU) before it exits.
  It sends nothing during normal play and no gameplay data. Set
  `POSTHOG_ENABLED=false` to stop it; the crash is written to the log either
  way.

Nothing else. Players connect to the server; the server does not connect to
them.
