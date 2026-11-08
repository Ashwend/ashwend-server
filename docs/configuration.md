# Configuration

Every setting is an environment variable on the container (`-e NAME=value` with
`docker run`, or under `environment:` in the compose file). Change one and
recreate the container for it to take effect.

| Variable | Default | Meaning |
| --- | --- | --- |
| `ASHWEND_SERVER_NAME` | `Ashwend Server` | The name in the server browser. |
| `ASHWEND_ADMINS` | empty | Comma-separated SteamID64s allowed to use the in-game admin panel. Read once at start. |
| `ASHWEND_MAX_PLAYERS` | `50` | The most players online at once, also shown in the browser. Admins bypass it, and a player who already has a body in the world can always come back to it, so the count can briefly go over. `0` means no limit. |
| `ASHWEND_SEED` | random | The seed of a **new** world. Has no effect on a world that already exists. The log records the seed every new world was made from, so you can make the same map again elsewhere. |
| `ASHWEND_WIPE_DAYS` | `30` | How many days of server running time a new world lasts. See [the wipe cycle](wipe-cycle.md). |
| `ASHWEND_VAC` | `1` | `0` runs without VAC. Steam tickets and game ownership are still checked. |
| `ASHWEND_GSLT` | unset | An optional Game Server Login Token. See below. |
| `ASHWEND_WEATHER_FEED` | on | `off` stops the server fetching real-world weather data. See below. |
| `ASHWEND_AUTH` | `steam` | `no-auth` turns off Steam sign-in checks entirely. For a private LAN only, see below. |
| `ASHWEND_ALLOW_UNAUTHENTICATED_LAN` | `0` | `1` lets a `no-auth` server listen on all addresses. See below. |
| `ASHWEND_BIND` | `0.0.0.0:7777` | The game's UDP address and port inside the container. |
| `ASHWEND_QUERY_PORT` | `27016` | Steam's browser query port inside the container. |
| `ASHWEND_WORLD` | `/data/world.save` | Where the world is saved. |
| `ASHWEND_BANS_FILE` | `/data/bans.json` | The ban list. |
| `ASHWEND_AUDIT_LOG` | `/data/moderation.log` | The moderation log. |

The server tells Steam the ports it listens on, so the ports you publish with
`-p` have to be the same numbers on both sides. To run a second server on the
same machine, move both its ports and publish those:

```bash
-e ASHWEND_BIND=0.0.0.0:7778 -e ASHWEND_QUERY_PORT=27017 \
-p 7778:7778/udp -p 27017:27017/udp
```

Extra arguments after the image name are passed to the server as they are, and
`docker run --rm ghcr.io/ashwend/ashwend-server:beta --help` lists them all.

## Game Server Login Token (GSLT)

You do not need one. Without a token the server signs in to Steam anonymously
and still gets full listing, player counts and VAC. A token ties the server to a
Steam account that owns the game, which keeps its identity when its IP address
changes. Create one at
[steamcommunity.com/dev/managegameservers](https://steamcommunity.com/dev/managegameservers)
(app ID `4985910`) and set `ASHWEND_GSLT`.

## Weather

The planet's clouds and storms follow real-world forecast data from NOAA, which
the server downloads over HTTPS (about 800 KB every six hours from
`nomads.ncep.noaa.gov`). It is optional: with `ASHWEND_WEATHER_FEED=off`, or
with no route to NOAA, the world runs on its own generated climate and nothing
else changes.

## `no-auth` is for a private LAN only

`ASHWEND_AUTH=no-auth` turns off the Steam ticket check, so **any player can claim
to be anyone**, including one of your admins. Such a server is also not listed
in the browser; players join it with Direct connect.

To make that hard to do by accident, a `no-auth` server refuses to listen on
anything but the machine itself unless you also set
`ASHWEND_ALLOW_UNAUTHENTICATED_LAN=1`. With both set it starts with a warning in
its log. Only do this on a network where every player is someone you trust, and
never expose such a server to the internet.

On a `no-auth` server the admins are the SteamID64s in `ASHWEND_ADMINS`, and
anyone who knows one of those numbers can claim it.
