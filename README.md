# Let Them Pass for Warzone 2100 4.7.0

An unofficial Windows x64 build for the single-player campaign. It helps friendly ground droids with Move orders work through head-on traffic and recover from a movement pause whose collision timer has cleared.

## Install and play

1. Extract the whole ZIP to a folder you can write to, such as Documents. Keep `bin` and `data` beside the launcher.
2. Open the extracted folder in File Explorer and double-click **Launch Let Them Pass.cmd**. The build uses OpenGL. A short-lived command window is normal.
3. Start a campaign or choose **Load Campaign Game**. The launcher stores settings, logs, and saves in the `profile` folder beside it.

To bring over an existing 4.7.0 campaign, copy its save folder from your usual Warzone profile into `profile\savegames\campaign`. Copy the whole folder, including its JSON and map files. The usual Windows profile is `%APPDATA%\Warzone 2100 Project\Warzone 2100`. If your installation uses another profile, use its campaign save folder. Launch one copy of the game at a time when using a given profile.

Use your regular shortcut or Steam entry to return to your normal game. The portable build uses stock campaign limits and includes its own game data. The ZIP is an engine build with an executable and libraries. Extract it as a separate game folder.

## What to expect

- At a detected head-on stall, the yielding side tries a short clear bypass. If needed, it backs along the route or pulls aside and waits while the other side passes.
- When the dispute recurs near the same spot, the other side gets the next turn. The game keeps the issued Move order, and a fresh order takes precedence.
- A paused Move order with a cleared collision timer retries after a staggered 200–800 ms delay in game time. Repeated retries without progress widen the delay window up to two seconds.
- Dense crowds can still pause or jam, particularly when the yielding side has little room to retreat. The traffic rules apply to the local player's ground droids on campaign Move orders. Multiplayer and skirmish use their existing movement rules.

Look for `Friendly traffic:` entries in the portable profile's `logs` directory when reporting a repeatable stall. Include the game version, map or mission, passage layout, and commands issued. A save taken during the stall can capture positions and Move orders.

## Source

The complete ZIP includes a `source` folder. On GitHub, the Windows player ZIP and its corresponding source ZIP are separate downloads. The source folder includes the modified engine and its submodules, a patch against the original 4.7.0 tag, build instructions, and the source archives for the bundled runtime libraries. See `COPYING`, `COPYING.README`, `COPYING.NONGPL`, and `licenses` for the applicable notices. This modified build carries forward the Warzone 2100 linking exception in `COPYING.README`.

Upstream game and documentation are available from [the Warzone 2100 project](https://github.com/Warzone2100/warzone2100).

## Credits

OpenAI Codex (June) implemented and documented the Let Them Pass engine changes. Tiggr71 identified the movement problems, proposed the staggered retry, supplied saves that reproduced the stalls, and tested the changes in live gameplay. Warzone 2100 itself is the work of the original developers and the upstream project contributors.
