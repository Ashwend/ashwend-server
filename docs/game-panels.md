# Game panels and shared IP addresses

For anyone running Ashwend on Pterodactyl, Pelican or a hosting provider's
panel, and for providers setting up the [egg](../pterodactyl/egg-ashwend.json).

## The server needs two ports

| Port | Default | What it is |
| --- | --- | --- |
| Game port | 7777 | Players connect to it. On a panel, the server's primary allocation. |
| Query port | 27016 | Steam's server browser asks it for the server's name, player count and day. |

Both are UDP, and both have to reach the server. A server whose query port
cannot be reached still runs and can be joined with **Direct connect**, but it
never appears in the server list: the browser drops any server that does not
answer.

## Why 27016 is usually not yours on a panel

A panel only lets traffic through to the ports it has allocated to the server.
On a shared IP address every port belongs to at most one server, so 27016 can be
the query port of one server on that address and no other, and usually it is
not allocated to yours at all. Each Ashwend server therefore needs **a second
port of its own** for Steam's queries, and the server has to be told its number.

## If you rent a server

1. In your panel, add a second port to the server. Panels call it an extra
   port, an additional allocation or a query port. You get a number, for
   example 25571.
2. In the server's settings (Startup), set **Steam Query Port** to that number.
3. Restart the server.

Within a minute or so the server should appear in the in-game server browser.
To check it yourself, add it as a favourite in Steam: View, Game Servers,
Favorites, Add a server, then enter the address with the **query** port
(`45.45.216.173:25571`). Steam shows the server's name and its day ("Day 3 of
30") when the query port answers.

If your panel does not let you add a port, ask your host for one.

## If you are a hosting provider

Give every Ashwend server two allocations when you create it, and set its
**Steam Query Port** variable to the second one's number. In Pterodactyl that is
the server's Build Configuration (an additional allocation) and its Startup
variables. If you would rather customers do it themselves, allow each Ashwend
server at least one extra allocation (Pterodactyl's allocation limit, with
automatic allocation creation switched on).

A port pair that always goes together, such as the game port and the one after
it, makes this easy to automate, but the server does not assume it: it uses
exactly the port in Steam Query Port.

The egg's default of 27016 only works where 27016 is allocated to that one
server, which on a shared address is the exception.

## The server's other settings

Everything else (the name, admins, player limit, VAC, world seed, the length of
the wipe cycle) is in the server's settings, described in
[Configuration](configuration.md). The world, the ban list and the logs are in
the server's files.
