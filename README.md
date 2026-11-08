# Ashwend dedicated server

Run your own server for [Ashwend](https://store.steampowered.com/app/4985910), a
multiplayer survival game on Steam. Your server shows up in the in-game server
browser and in Steam's, and players join it with their own copy of the game.

This repository holds the server's documentation and a ready-made compose file.
The server itself ships as a container image:

```
ghcr.io/ashwend/ashwend-server
```

Every server release is listed under [Releases](../../releases) with the tags it
published.

## What you need

- **A Linux x86-64 machine with Docker.** ARM machines (a Raspberry Pi,
  Hetzner's CAX line, AWS Graviton) cannot run it: Steam's server library
  exists for x86-64 only.
- **Two UDP ports reachable from the internet: 7777 and 27016.** See
  [Ports](#ports).
- **Outbound internet access**, so the server can sign in to Steam and check
  each player's ticket.
- **Memory:** an empty server uses about 250 MB. Leave room on top for players
  and what they build.

## Quick start

```bash
mkdir -p ashwend/data && cd ashwend
docker run -d --name ashwend --restart unless-stopped --stop-timeout 30 \
  -p 7777:7777/udp -p 27016:27016/udp \
  -e ASHWEND_SERVER_NAME="My Ashwend Server" \
  -e ASHWEND_ADMINS="<your SteamID64>" \
  -v "$PWD/data:/data" \
  ghcr.io/ashwend/ashwend-server:beta
```

Or download [`docker-compose.yml`](docker-compose.yml) into the folder, set the
server name and your SteamID64 in it, and run `docker compose up -d`.

The first start generates a new world, which takes a few seconds. Follow the log
with `docker logs -f ashwend` (or `docker compose logs -f`). When it prints
`Lightyear game server listening on 0.0.0.0:7777`, players can join.

Your SteamID64 is the 17-digit number starting `7656119` on your Steam profile
page. Putting it in `ASHWEND_ADMINS` gives you the in-game admin panel (see
[Administration](docs/administration.md)).

## Choosing a version

A server only admits players on **exactly** its own version. Players get their
version from Steam, so run the tag that matches the Steam branch your players
are on:

| Tag | What it runs |
| --- | --- |
| `beta` | The build on Steam's `beta` branch. |
| `latest` | The build on Steam's default branch, once the game is released. |
| `0.29` | The newest release in one line (production releases only). |
| `0.29.0-beta.1` | One exact build, for when you want to choose when to update. |

Until the game's store release, the default branch carries the beta as well, so
**run `beta`**. The server browser shows players whose version differs from
yours a "needs x.y.z" badge on your server instead of letting them fail to
connect.

## Ports

Both ports are UDP, and both need to reach the server:

| Port | What it is |
| --- | --- |
| 7777 | Game traffic. This is what players connect to. |
| 27016 | Steam's server browser query port. |

Forwarding only 7777 gives you a server that works for anyone who types your
address into **Direct connect**, but never appears in the server list, because
Steam cannot query it. That is the most common setup mistake.

## Updating

When a new version reaches Steam, update the server with it:

```bash
docker pull ghcr.io/ashwend/ashwend-server:beta
docker stop ashwend && docker rm ashwend
# then the same `docker run` as above
```

With compose: `docker compose pull && docker compose up -d`.

Update when your players' Steam client does, not before: a server on a newer
version than your players turns them away until Steam updates them.

## Stopping and backups

`docker stop ashwend` (or `docker compose stop`) saves the world before the
server exits. Give it the time to: the commands above allow 30 seconds. The
world is also saved automatically every 30 minutes, so only a crash or a forced
kill loses play, and at most that much.

Everything the server keeps is in the `data` folder: the world, the ban list and
the moderation log. Back up that folder and you have backed up the server.

## More

- [Configuration](docs/configuration.md): every setting, Steam login tokens, LAN
  servers.
- [Administration](docs/administration.md): admins, kicks and bans, the files in
  `data`.
- [The wipe cycle](docs/wipe-cycle.md): every world ends after 30 days and a new
  one begins.
- [Troubleshooting](docs/troubleshooting.md): a server missing from the list,
  players turned away, what the server connects to.
