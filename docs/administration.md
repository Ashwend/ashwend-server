# Administration

## Admins

Put the SteamID64s of your admins in `ASHWEND_ADMINS`, comma-separated, and
restart the server. The list is read once at start.

On a server using Steam sign-in (the default), the game's developer is an admin
as well.

In game, an admin presses Escape and chooses **Admin**. The panel has:

- **Players**: everyone online with their ping and SteamID64 (click to copy).
  Per player: **Go to** (teleport to them), **Bring** (teleport them to you),
  **Kick** and **Ban**, each with a reason the player sees.
- **Bans**: the active ban list, with **Unban**.
- **World**, **Launch sites**, **Yourself** and **Spawn**: tools for the world
  itself, from the time of day to the end of the world (see
  [the wipe cycle](wipe-cycle.md)). The chat has no commands; everything is in
  the panel.

A kick disconnects a player, who can rejoin at once. A ban lasts across
restarts until it is lifted. Admins cannot be banned, and nobody can kick or ban
themselves.

## The files in `data`

| File | What it is |
| --- | --- |
| `world.save` | The world. A compressed binary file: do not edit it. Replaced by the next world at the end of each cycle. |
| `bans.json` | The ban list. Readable and editable, see below. |
| `moderation.log` | A running record of every kick, ban and unban. The server only ever adds to it. |

Back up the whole folder to back up the server. To start a fresh world, stop the
server and move `world.save` out of the folder; your bans stay.

## Editing `bans.json`

You can fix a ban by hand without the game running:

```json
{
  "version": 1,
  "bans": [
    {
      "steam_id": "76561198000000000",
      "display_name": "Someone",
      "reason": "Griefing spawn",
      "banned_by": "76561198000000001",
      "banned_at": 1769000000,
      "expires_at": null
    }
  ]
}
```

- Edit it while the server is **stopped**. The server rewrites the file on every
  ban and unban and would overwrite your change.
- `steam_id` is written as a string on purpose: a SteamID64 is too large for
  many JSON tools to keep exact as a number, and a changed digit bans someone
  else. A plain number is still accepted.
- `banned_at` and `expires_at` are Unix times in seconds; `expires_at: null`
  means permanent, which is what the panel issues.
- If the file cannot be read, the server still starts, with no bans, logs the
  error and leaves your file as it is so you can fix it.
