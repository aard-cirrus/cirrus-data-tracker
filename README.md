# cirrus-data-tracker
the only data tracking aardwolf plugin you need

**Cirrus Data Tracker (CDT)** is a MUSHclient plugin for Aardwolf MUD. It records
what you earn, deal, take and heal, as it happens, into a local SQLite database,
and turns it into reports, rates, projections and live meter windows.

Current version: **4.5.0** (database v39). Type `cdt help` in game for the command
list, and `cdt help <topic>` for any topic below.

---

## Contents

- [Install](#install)
- [Features at a glance](#features-at-a-glance)
- [Dates, times and scopes](#dates-times-and-scopes)
- [Gains tracking](#gains-tracking) (`cdt gains`)
- [Damage tracking](#damage-tracking) (`cdt dmg`)
- [Damage meter window](#damage-meter-window) (`cdt dmg meter`)
- [Heals](#heals) (`cdt heal`)
- [Progress: tier, redo, flags](#progress-tier-redo-flags)
- [Instinct, mastery, potential and builds](#instinct-mastery-and-builds)
- [Snapshots, dtrack, gaias and wail](#snapshots-dtrack-gaias-and-wail)
- [Automatic reports](#automatic-reports) (`cdt events`)
- [Channel query listener](#channel-query-listener) (`cdt listen`)
- [Gags](#gags) (`cdt gag`)
- [Configuration](#configuration) (`cdt config`)
- [Data management](#data-management) (db, archive, export, rebuild, reset)
- [Debugging](#debugging)
- [License](#license)

---

## Install

**Requirements**
- MUSHclient **4.73** or later.
- The Aardwolf MUSHclient package (it provides `gmcphelper`, `movewindow` and
  the miniwindow theme the plugin uses). GMCP must be on.

**Steps**
1. Copy `cirrus_data_tracker.xml` into your MUSHclient `worlds/plugins` folder.
2. In MUSHclient: **File → Plugins → Add**, and pick the file.
3. Type `cdt help`.

All data goes into `cirrus_tracker.db`, beside the plugin. The plugin makes a
backup before every database upgrade (see [Database](#database)).

**Updating.** Type `cdt update` to see whether a newer public version exists, and
`cdt update install` to download it and put it in place. The download is checked against the
checksum in the repository's `version.json` and must be CDT of that version; your current file is
kept beside it as `cirrus_data_tracker.xml.<date>-update.bak`. Then type `cdt reload`. CDT never
reloads itself, and the web request runs in the background. `cdt config update_check on` makes it
look once, 20 seconds after CDT loads (off by default). See `cdt help update`.

---

## Features at a glance

| Area | What CDT does |
|---|---|
| **Gains** | XP (kill, rare, bonus, double, blessing, superhero honour), gold by source, QP, TP, trains, pracs, levels, powerups, quests, campaigns, GQs, bonus loot |
| **Reports** | Any kind of gain over any time span, tables over time, personal records, period-vs-period comparisons, per-area / subclass / race rates |
| **Live rates** | Rolling 1/5/15-minute XP/hr, session averages, earning estimates from hourly to yearly, a floating gains meter |
| **Damage** | Every damage line: your attacks and damage types, groupmates' damage, damage taken by mob/attack/type, all in 30-minute slices per level |
| **Damage meter** | A live window with DPS, total damage, crit % and healing per groupmate, plus a per-player detail view |
| **Heals** | Incomplete healing and (4.4.7) regeneration, on you and by you; groupmates' heals on others estimated from their mana; heal scrolls by what they healed |
| **Progress** | Time, kills, XP, QP and damage per 20 levels, per redo, and per PK flag |
| **Builds** | Instinct and mastery cost calculators, benefit tables, damage projections, saved builds, a build optimiser |
| **Sharing** | Automatic event lines to any channel, report output to channels, and a listener that answers allowed players' queries |
| **Data** | Backups and restore, archiving, JSON/CSV export, summary rebuilds, integrity checks |

---

## Dates, times and scopes

**Dates** are typed and shown in your date format: M/D/Y by default, so
`2/3/2026` is February 3rd 2026 at midnight. Add `_HHMM` for a 24-hour time:
`2/3/2026_1630` is 4:30pm. Change the order with `cdt config dateformat`.

**Times** are stored as Unix time and shown in this computer's timezone, or a
fixed offset (`cdt config timezone UTC-8`). Changing either changes how times
read, never the stored data. Aardwolf's day resets at midnight US Eastern.

**Scopes** are used by most reports:

| Scope | Covers |
|---|---|
| `all` | everything recorded (the default for a type) |
| `session` | since you logged in or reloaded the plugin |
| `day` / `week` / `month` / `year` | since midnight / your week start day / the 1st / January 1st, your timezone |
| `aardday` / `aardweek` / `aardmonth` / `aardyear` | the same on Aardwolf's clock (US Eastern; weeks start Monday) |

**Sessions and active time**: a session starts at login or plugin load and ends
at logout, disconnect or reload. Active time is session time minus AFK time;
per-hour figures divide by active time.

---

## Gains tracking

Records XP from kills, gold by source, QP from quests/campaigns/GQs/other,
trains and pracs, level-ups, powerups, quests, campaigns and GQs. Type
`cdt gains` for the subcommand list.

| Command | What it does |
|---|---|
| `cdt gains report <type> [scope] [channel]` | one kind of report |
| `cdt gains report <scope> [channel]` | an overview of everything |
| `cdt gains rate` | rolling and session rates |
| `cdt gains estimate` | projected earnings, hourly to yearly |
| `cdt gains history <type> [n]` | the last n events, newest first |
| `cdt gains subclass` / `race` / `combo` | performance by subclass, race, or both |
| `cdt gains area [filters]` | XP and gold rates per area |
| `cdt gains area quest [sort] [scope]` | where quests send you and what they pay |
| `cdt gains area bloot [scope]` | bonus loot by area, and the mobs behind it |
| `cdt gains best [scope]` | personal records |
| `cdt gains compare <period> [full]` | this period so far against the last |
| `cdt gains milestone` | lifetime targets |
| `cdt gains goal <type> <n>` | a target for this session |
| `cdt gains track` | record a stretch of play by itself |
| `cdt gains meter` | the floating gains meter window |

### Reports (`cdt gains report`)

A type without a scope shows all time; the overview always needs a scope
(`cdt gains report day`).

| Type | Shows |
|---|---|
| `xp` | total XP from kills, rare kill bonus, double XP bonus, daily blessing, superhero honour and campaigns; kills, average XP per kill, kill specials (vorpal, assassinate ...), double XP time, XP/hr |
| `gold` | total gold and every source, largest first (mob, quests, campaigns, GQs, shared, sales, crumble, sacrifices, fence, haggle, other, clan tax); gold/hr |
| `qp` | quests (base, tier, lucky, MCCP, OPK, hardcore, daily blessing), campaigns (base, first of the day), GQs (mob kills, winning), other; QP/hr |
| `quest` | done/failed, double QP and blessing quests, QP with each bonus, gold, TP, trains, pracs, averages, quests/hr |
| `cp` | done/failed, QP (base + first of the day), gold, TP, trains, pracs, XP, average QP |
| `gq` | joined, finished, won, QP (mob kills + winning), gold, TP, trains, pracs |
| `level` | solo/grouped, average and fastest time, trains and bonus trains, HP/MP/MV, per hour; total levels |
| `powerup` (`pup`) | the same for powerups, with lifetime powerups |
| `bloot` | bonus loot by tier (best first), 1 per how many kills, by area |
| `tp` | trivia points from quests, campaigns, GQs, trivia mobs and anything else; TP/hr |
| `trains` | trains from level-ups and powerups (base, lucky, blessing, instinct), quests, campaigns, GQs and other; trains/hr |

**Tables over time**: put a number before a scope for one row per period,
newest first, up to 60: `cdt gains report qp 7day`, `cdt gains report 12 month`,
`cdt gains report xp 10aardweek`. The running period is marked `*`, the best row
is green, and the Average row counts only finished periods you played.

**All-time views** end with period tables (today, this week, this month, this
year), your clock beside Aardwolf's.

**Channel output**: a channel at the end sends the report as one line, and the
table still prints locally:

```
cdt gains report qp week gt
[CDT] [QP this week (19h 08m) | 2,904qp | quests 1,890 (...) | cp 690 (510+180daily) | gq 324 (104mob+220win)]
```

Channels: `echo` (local), `gtell` (`gt`), `clantalk` (`ct`), `ftalk` (`ft`),
`spouse` (`spo`), and any you add with `cdt config channels add`. A line sent
ends with a colour reset, so a report's colours don't bleed: `$C` on the game's
channels (`gtell`, `clantalk`, `ftalk`, `spouse`, their own colour), `@C` on
`say` (bright cyan), and `@w` (grey) on echo, tells and channels you add, where
`$C` would show as text.

**Good to know**
- A session that started before the scope began counts in proportion, so
  those rates are marked approximate.
- Double XP, blessing XP, kill specials and GQ joins are kept apart from DB
  v27 on; older kills show one "bonus (before the split)" line.
- Mob QP from a GQ still running is included before the GQ is recorded.

### Rates (`cdt gains rate`)

- **Rolling XP/hr** over the last 1, 5 and 15 minutes, by the clock: they fall
  while idle or AFK, and show red until the window has filled.
- **Session averages**: XP/hr, gold/hr, kills/min, average XP per kill,
  quests/hr and QP/hr, plus totals, levels, pups and active time. These divide
  by active time, so AFK time doesn't drag them down.

### Estimates (`cdt gains estimate`)

| Column | Based on |
|---|---|
| Hourly (Live) | the past 60 minutes, projected to a full hour (only with kills in the last hour) |
| Hourly (Hist) | per active hour over the last 7 days; prefers the same weekday, else all days if fewer than 3 match |
| Daily (Hist) | per day over the last 30 days; same weekday, then weekday/weekend, then all days (needs 3+) |
| Weekly (Hist) | per week over the last 30 days |
| Monthly (Hist) | up to 90 days; needs 14+ days of data |
| Yearly (Hist) | up to 365 days; needs 60+ days of data |

All estimates are worked out from the event tables. Hourly (Hist) divides by
active time only; calendar estimates scale totals by elapsed calendar time.

### History (`cdt gains history <type> [count]`)

The most recent events of one kind, newest first: 10 by default, 100 at most.

| Type | Each line shows |
|---|---|
| `xp` | kills and campaign XP: XP, rare, bonus, level, mob, area |
| `gold` | every gold gain by source |
| `qp` | every QP gain by source, with breakdown |
| `tp` (`trivia`) | every trivia point and its source |
| `trains` (`train`) | every train, with bonus trains |
| `quest` | done/failed, QP and bonuses, time taken, rewards, target |
| `cp` | done/failed, QP, first-of-the-day bonus, gold, TP, trains, XP |
| `gq` | GQ number, won, time taken, mob and win QP, rewards |
| `level` | level, time taken, HP/MP/MV, trains and bonus trains, stats |
| `powerup` (`pup`) | powerup number, time taken, trains and bonus trains, kills |
| `session` | start, length, active time, XP, QP, gold, levels, pups |
| `all` | everything above in one timeline |

### Subclass, race, combo and area

- **`cdt gains subclass`** (or `class`): per subclass, levels and powerups,
  average and best time, gold (total/solo/group) and XP (total/solo/group),
  from every recorded level-up, powerup and kill. Gold counts mob, crumble and
  shared gold only.
- **`cdt gains race`** / **`cdt gains combo`**: the same by race, and by race +
  subclass.
- **`cdt gains area [filters]`**: XP and gold rates per area, split solo/group,
  leaving out gaps over 60 s between kills. Filters combine: `nodbl` (no double
  XP kills), `dbl` (double XP only), a class (`ninja`), a race (`elf`).
- **`cdt gains area quest [count|qp|time] [scope]`**: the areas quests send you
  to, how often, average base QP (bonuses left out), time taken and QP paid.
- **`cdt gains area bloot [scope]`**: bonus loot per area, best tier and drop
  rate per kill, and under each area the mobs that dropped it by tier.

### Records and comparisons

- **`cdt gains best [scope]`** (or `records`): fastest quest, campaign, GQ win,
  powerup and level-up; best day for XP, QP, gold, kills, powerups and quests;
  biggest kill and richest quest; each with when it happened.
- **`cdt gains compare <day|week|month|year> [full]`**: this period so far
  against the last one up to the same point (so a Wednesday isn't set against
  a whole week), with XP, QP, gold, TP, trains, levels, powerups, kills,
  quests, campaigns, GQs, rates and active time. `full` compares the last
  finished period with the one before. `aardday` ... `aardyear` work too; with
  no period it compares weeks.

### Milestones and goals

**Milestones** (`cdt gains milestone`) are lifetime targets, checked as your
totals grow.

| Command | What it does |
|---|---|
| `cdt gains milestone` | pending and achieved |
| `cdt gains milestone add <type> <name> <target>` | add one, e.g. `add qp 2m_qp 2000000` |
| `cdt gains milestone update <name> <target>` | change a target |
| `cdt gains milestone del <name>` | delete one |

Types: `qp` (total QP, from whois), `pup` (lifetime powerups), `mob` (monsters
killed). Auto-milestones are added at every 100k QP, 10k powerups and 500k mobs
ahead of your totals.

**Goals** (`cdt gains goal <type> <count>`) are targets for this session:
`pup`, `level`, `quest` or `kill`. `cdt gains goal` shows progress,
`cdt gains goal clear` removes it. A reload keeps it; reaching it prints a goal
line.

### Tracked sessions (`cdt gains track`)

Record earnings over a stretch of play by themselves. A **manual** one (start
and stop yourself) and an **auto** one (runs for each double XP) can run
together. The auto one runs for the double's length, as the game announces it:
10 or 15 minutes for most, an hour for the million monsters double, which
replaces any double already running (and the session's end with it).

| Command | What it does |
|---|---|
| `cdt gains track` | what's running now |
| `cdt gains track start [name] [minutes]` | start manual tracking, with an optional time limit |
| `cdt gains track stop` | stop it and show the report |
| `cdt gains track list` / `show <id\|name>` | saved sessions / one report |
| `cdt gains track del <id>` / `rename <id> <name>` | delete / rename |
| `cdt gains track retention <n>` | how many to keep (1-100, default 20) |

A reload ends tracking. The summary line goes to the `session` event's channel.

### Gains meter window (`cdt gains meter`)

A draggable floating window of live figures (rows) over up to three time
intervals (columns, default 5m, 15m and 1h). Right-click it for fonts, rows,
intervals and the data filter.

| Command | What it does |
|---|---|
| `cdt gains meter` | settings and every metric name |
| `cdt gains meter on\|off\|toggle` | show or hide it |
| `cdt gains meter show\|hide <metric>` | add or remove a row |
| `cdt gains meter interval <1-3> <seconds>` / `on\|off` | set or hide a column (10 s minimum) |
| `cdt gains meter session <filter>` | which data it reads |
| `cdt gains meter resetpos` | move it back to the top left |

- **Metrics**: `xp_rate`, `xp_total`, `gold_rate`, `gold_total`, `qp_rate`,
  `qp_total`, `kill_rate`, `kill_total`, `pup_rate`, `pup_total`, `pup_avg`,
  `lvl_rate`, `lvl_total`, `lvl_avg`. `_rate` is per minute, `_total` adds up
  the interval, `_avg` is the average time per powerup or level.
- **Filters**: `all` (or `0`), `current`, `tracking`, `double`, `track:<id>`,
  or a past session number.

---

## Damage tracking

Reads every damage line: your hits (by attack and damage type), damage you
take, and each groupmate's damage. Type `cdt dmg` for the list.

| View | Shows |
|---|---|
| `cdt dmg report` | your damage: this session and all time |
| `cdt dmg subclass [name]` / `race [name]` | per subclass / race; with a name, its damage lines |
| `cdt dmg combo [filter]` | per race + subclass |
| `cdt dmg level [n]` | damage per character level, or level n's lines |
| `cdt dmg area [N]` | combat efficiency per area (top 20 by default) |
| `cdt dmg attacks [N]` | each attack by DPS (top 25 by default) |
| `cdt dmg types` | each damage type (fire, slash, ...) |
| `cdt dmg player <name>` | a scouting report for one groupmate, with their heals on you |
| `cdt dmg players` | every player seen, summarised |
| `cdt dmg taken` | damage you took, by mob, attack, damage type, race and subclass |
| `cdt dmg taken player <name>` / `taken players` | a groupmate's damage taken / everyone ranked by it |

Damage taken is recorded from 4.3 on; hits on groupmates are seen only while
they're in your group.

**Filters** (all views):
- `<minlevel> <maxlevel>`, e.g. `150 201` (201 is superhero).
- `<from> <to>` dates, with optional `_HHMM`.
- Tier tokens: `T9`, `T9R7`, `T9+3`, `T9+3R7` (see [Progress](#progress-tier-redo-flags)).
- `solo` or `group` (4.4.5): only the damage recorded while you were solo, or in
  a group, e.g. `cdt dmg combo gargoyle/necromancer solo`, `cdt dmg area 10 group`.
  Not for `player` and `players`. Totals and time are exact for all your history;
  the per-attack, damage type and damage taken breakdowns are exact from 4.4.5
  on, and older damage from slices that mixed solo and group fights shows as
  "not split". The pre-v3.9 estimate and the instinct/mastery sections can't be
  split, so they're left out.
- For `area` and `attacks`, N comes before any level range:
  `cdt dmg attacks 10 150 201`.
- Example: `cdt dmg attacks T9R7 150 201 2/3/2026 2/4/2026_1600`.

**How damage is saved**: in 30-minute slices per character level, stamped with
tier/remort/redo, written only when there was combat. A level-up, tier/remort
change, or instinct/mastery snapshot starts a new slice. Date filters work in
the same 30-minute steps. Damage from before v3.9 has no slices; filtered views
show an approximate estimate for it in a separate table.

**Damage type mappings**: each attack maps to a damage type, used by mastery and
the build tools. Most are built in.

| Command | What it does |
|---|---|
| `cdt dmg map` | list mappings (`[user]` = yours) |
| `cdt dmg map usage` | every attack seen, with hit counts |
| `cdt dmg map unknown` | attacks with no damage type yet |
| `cdt dmg map <attack> <damtype> [category]` | set a mapping |
| `cdt dmg map delete <attack>` | remove your mapping |

After changing mappings, `cdt rebuild dmg` re-applies them to older damage.

**Removing data**: `cdt dmg purge` (numbered list of attacks),
`cdt dmg purge <#> confirm` (delete one attack's data),
`cdt dmg reset confirm` (delete all damage data).

---

## Damage meter window

A live window for the current fight or session, with four categories: **DPS**,
**Total Dmg**, **Crit%** and **Healing**. Click a player for their **detail
view**, on the same category: their stats and a bar per attack (or per heal, by
Healing), with prev/next arrows to step through players.

| Command | What it does |
|---|---|
| `cdt dmg meter [on\|off]` | show or hide the window |
| `cdt dmg meter start` / `pause` (`stop`) | start/resume or pause counting |
| `cdt dmg meter reset` | clear it and start a new count |
| `cdt dmg meter min` | minimalist mode on/off |
| `cdt dmg meter auto [on\|off]` | start counting by itself when you log in, reload or reconnect (flips if blank) |
| `cdt dmg meter report [dps\|dmg\|crit\|healedme\|healedall\|healedgrp] [ch]` | send the table |
| `cdt dmg meter report player <name\|me> [ch]` | one player's attacks, then their heals on self, the leader and others |
| `cdt dmg meter customreport <metrics> player <name\|me> [ch]` | that player with only the figures you ask for |

- **Auto-start** (4.4.3): with `cdt dmg meter auto` on, the meter starts
  counting by itself when play begins (after a reload, a MUSHclient restart or
  a reconnect) instead of waiting for Start. Off by default and remembered.
  Turned on while the meter is idle, it starts at once; a meter already
  counting or paused is left alone. It doesn't show a hidden window.
- **Heal reports** (4.4.3): there are three. `healedme` sends the heals that
  landed on you, exact, titled *Heals on &lt;your name&gt;*. `healedall` sends
  everyone's heals in the meter's count, each healer's total and casts, biggest
  first (`~` where estimated), titled *Heals w/ estimates*. `healedgrp` sends each
  healer's heals on self, on the leader and on others as three figures,
  `1.Bob: ~9.5k/~14.3k/~23.8k` (self yellow, leader green, others cyan; 0 for
  none), titled *Group heals*. None depends on the meter's right-click option. Add
  `castsonly` (anywhere after the report name) for just the numbers of casts, no
  hp, ranked by casts: *Heal casts on &lt;you&gt;* (`healedme`, with shares),
  *Heal casts w/ estimates* (`healedall`), *Group heal casts* (`healedgrp`). A `~`
  marks an estimated count (a groupmate's casts seen only in their mana).
  One-line reports only. The older names (`heal`, `healing`, `healest`) and
  `cdt heal report group` are gone.
- **Custom metrics** are joined with `+` (`dps+dmg+crit+avg+healedme+healedall`), shown
  in your order, the first one sorting. `avg` is damage per hit, with the hit
  count in brackets. `healedme` adds the player's heals on you; `healedall` their
  heals on self, on the leader and on others, each with its casts. Damage
  figures list the attacks. Each part is labelled, in the order asked:
  `Heals: ... | Attacks (dps/dmg): ...`. The legend calls the heal figures
  *Heals on &lt;your name&gt;* and *Heals w/ estimates*. Example:
  `cdt dmg meter customreport healedall+dps player Bob`.
- **Reports name you** instead of saying "you", since whoever reads the channel
  (or asked over `cdt listen`) isn't you.
- **Right-click the meter** for *Minimalist Mode* and (4.4.2) *Healing: Include
  Estimates*: the Healing tab then adds groupmates' estimated heals on others
  (`~`) and your heals on others. Off by default, and remembered.
- **Detail view** (4.4.2) shows only the current tab's lines: Dmg / Time / DPS on
  DPS and Damage, Hits / Crits on Crit% and Avg/Hit, the heals on Healing. Taken
  / Per min shows on every tab.
- **Healing, group view**: each player's heals on you (yours on yourself on
  your row).
- **Healing, detail view**: who a player's heals landed on (4.4.3). For a
  groupmate: **on self**, **on leader**, **on others - you** (exact, from your
  heal lines) and **on others - not you**; when you lead, **on leader - you**.
  On your row: **on self**, **on leader**, **on others**, all exact. A
  groupmate's cast is put on whoever's hp jumped most in their room when their
  mana dropped; estimates show `~`. The leader is whoever leads the group now.
  Above the bars: **Healed you / Casts**, **Casts~ / On others~**, **Regen~ /
  HP~** (4.4.7) and **Scroll heals / HP**.
- The meter counts only while it runs and doesn't change saved data.

---

## Heals

Two heals are recorded: **incomplete healing** and (4.4.7) **regeneration**.

| Who | Line |
|---|---|
| your incomplete healing on you | `You are [greatly] healed by your magic touch. [N]` |
| a player's incomplete healing on you | `You feel [greatly] healed by <player>'s magic. [N]` |
| your incomplete healing on others | `<player> is [greatly] healed by your magic touch. [N]` |
| regeneration on you (anyone's) | `A great healing power courses through your veins leaving you greatly regenerated. [N]` |
| your regeneration on others | `A great healing power courses through the veins of <player> leaving him greatly regenerated. [N]` |

**Regeneration on you** is the same line whoever cast it, so its caster is
whoever's mana dropped by regeneration's cost (1,100; an Eldar's 825) in the next
group update: you, a groupmate, or `unknown` when no one's did. Solo, it's yours.

**Groupmates' heals on others (4.4)** show no heal line, so two things count
them:
- **Casts (an estimate)**: their mana dropping by 950 in the group data (712 or
  713 for an Eldar, learned per player), or close to it once they're known to
  heal. Some are missed, and wasted heals count. A healer's heal size is their
  heal on you, or else worked out from the hp jumps their incomplete healing
  casts cause.
- **Regeneration (an estimate, 4.4.7)**: their mana dropping by 1,100 (825 for
  an Eldar). Each counts the hp gain of the player it most likely healed (the
  biggest hp jump in their room in that update), or 4,600 when no one's hp
  jumped.
- **Scrolls**: `X recites a scroll of 3*incomplete healing on Y.` (or
  regeneration) in your room counts 3 heals, as much as Y's hp rose in the next
  group update (for incomplete healing, at most 3,400 a heal). Only when that
  can't be measured (Y isn't in your group data) does it count 3,400 a heal
  (regeneration 4,600). A scroll's heals on you are exact, from their lines,
  recorded as `<spell> (scroll)`.

Regeneration counts everywhere heals do: the damage meter's Healing (with
estimates), its detail view (**Regen~ / HP~**), `healedall`, `healedgrp` and
`castsonly`, and `cdt heal report` (a **Regen~** column in `healedall`; their
regeneration on others in `player`).

Other heals (`The life within you is renewed.`, `You feel your life restored.`,
`A warm feeling fills your body.`) name no one and aren't recorded.

| Command | What it does |
|---|---|
| `cdt heal report [scope]` | your own heals: total, casts, average and highest |
| `cdt heal report player <name> [scope]` | the same for `<name>`'s heals on you, plus their heals on others |
| `cdt heal report healedme [scope]` | everyone who healed you, most first, you included |
| `cdt heal report healedall [scope]` | each groupmate's casts, casts on others, heal size, healing on others, and who each cast most likely healed (the biggest hp jump in the healer's room; right about 3 times in 4) |
| `cdt heal history all\|self [n]` | the last n heals on you (10 default, 100 max) |
| `cdt heal history player <name> [n]` | `<name>`'s heals on you only |

**To a channel** (4.4.3): put a channel last and the report is sent as one line,
over the scope (all time unless you give one): your own heals; `player <name>`,
their heals on self and on others; `healedall`, each groupmate's total healed and
casts. `healedme` stays a table on your screen, and the tables on your screen
show everything, as before. The damage meter's own heal lines (`healedme`,
`healedall`, `healedgrp`) are `cdt dmg meter report`, above. Allowed players can
ask for `player <name>` over `cdt listen`.
Add `castsonly` to `healedall` for just the numbers of casts, ranked
(`~` where estimated): *Groupmates' heal casts*. It needs a channel (or `echo`);
`player`, `healedme` and your own heals don't take it.

**Long lines**: a line CDT sends to a channel is at most 999 characters (colour
codes included); a longer report goes on to a second line, and a third, split
between entries, each follow-on line starting `[CDT]`. Past three lines the last
is cut, ending `...`. If the connection drops within 10 seconds of CDT sending to a
channel, a `[CDT debug]` note says what was sent and how long each line was.

To hide heal lines, see [Gags](#gags).

---

## Progress: tier, redo, flags

| Command | What it does |
|---|---|
| `cdt tier [token]` | a tier in 20-level groups (default: where you are now) |
| `cdt redo [token]` | one row per redo of a tier (token without a redo: `T9` or `T9R7`) |
| `cdt flags [token]` | XP and QP by PK flag: OPK, HARDCORE, neither, unknown |

**Tier tokens** (also filter every `cdt dmg` view and `cdt build compare`):

| Token | Means |
|---|---|
| `T9` | tier 9: every redo and remort |
| `T9R7` | tier 9, remort 7 (any redo) |
| `T9+3` | tier 9, redo 3 (every remort) |
| `T9+3R7` | tier 9, redo 3, remort 7 |

A remort alone (`R7`) is refused.

**Columns**: Time at those levels; Kills, Deaths, Quests, CPs, GQs at the level
you were then; XP/hr (kill XP with rare/bonus); QP (quests, campaigns and GQs);
Damage and DPS from the 30-minute slices.

**PK flags**: kills, level-ups, quests, campaigns, GQs and sessions record
whether you were OPK and/or HARDCORE, read from your whois at login. `unknown`
is anything before v3.9 or before the first whois.

**Older data**: deaths, damage and redos are recorded from v3.9 on. Older kills,
level-ups and quests get a tier/remort estimated from their session; a redo
filter leaves them out.

---

## Instinct, mastery and builds

### Instinct (`cdt instinct`)

Reads the game's `instinct` table, estimates what each count is worth, and
costs out training. Each capture that differs from the last is saved as a
snapshot and starts a new damage slice.

| Command | What it does |
|---|---|
| `cdt instinct` | capture and show your table |
| `cdt instinct report [channel]` | send your counts to a channel |
| `cdt instinct fullreport [channel]` | counts plus estimated benefits |
| `cdt instinct <from> <to> <skill>` | trains and gold to go from→to, and the benefit at each count |
| `cdt instinct benefit <skill>` | the benefit at every count |
| `cdt instinct project <pts> <skill>` | projected damage gain from adding pts, using your damage since the last snapshot |

- Sanctuary depends on wisdom: add `<N>wis` (default 600), e.g.
  `cdt instinct 100 150 sanct 400wis`.
- Skill names can be shortened (`deso` = desolation).
- **Benefit data** covers melee (second/third/fourth attack, dual wield, haste,
  knife) and percentage skills (sanctuary, enhanced damage, protection
  good/evil, incomplete healing, attack spells such as desolation and
  apocalypse). `project` covers damage instincts only.

### Mastery (`cdt mastery`)

Reads the game's `mastery` table (points per damage type), converts points to
damage %, and costs out training.

| Command | What it does |
|---|---|
| `cdt mastery` | capture and show your table |
| `cdt mastery report [channel]` | send your points to a channel |
| `cdt mastery fullreport [channel]` | points plus damage % |
| `cdt mastery <from> <to>` | QP and gold to go from→to |
| `cdt mastery benefit` | points → damage % table |
| `cdt mastery project <pts> <type>` | projected damage gain from adding pts to a damage type (e.g. `fire`) |

### Potential (`cdt potential`)

The QP it costs to raise potential, from the game's cost table (0 to 450).

| Command | What it does |
|---|---|
| `cdt potential <from> <to>` | QP to go from→to, e.g. `cdt potential 0 20` is 6,750 QP |

### Builds (`cdt build`)

Save instinct/mastery setups, compare them, and find the best use of a budget,
projected against the damage you've actually recorded.

| Command | What it does |
|---|---|
| `cdt build save <name>` | save your current values |
| `cdt build new <name> [race] [subclass]` | an empty build |
| `cdt build copy <from> <to>` | clone a build |
| `cdt build list` / `show <name>` | saved builds / a build's values and cost |
| `cdt build set <name> inst\|mast <skill> <n>` | set one value |
| `cdt build add <name> inst\|mast <skill> +n` | add to one value |
| `cdt build set <name> gaias <element>` | a shaman build's focus |
| `cdt build set <name> subclass\|race <x>` | which subclass/race it's for |
| `cdt build set <name> instinctbuilderstring <url>` | import an instinct-builder.com share URL (instincts and subclass) |
| `cdt build delete <name>` | delete (asks to confirm) |
| `cdt build compare <a> [b] [full]` | compare; `b` defaults to your current values |
| `cdt build optimize [trains] [save <name>]` | the best damage build for a trains budget |
| `cdt build subclass [name\|skill]` | which instincts each subclass has |

**Comparing**: each side is projected against damage recorded for its own race
and subclass, and the report says which basis it used (race + subclass,
subclass only, or multipliers only). It shows each build's cost and the cost of
switching. Add `<minlevel> <maxlevel>` and/or a tier token to compare on damage
at those levels.

---

## Snapshots, dtrack, gaias and wail

**Snapshots** save your instinct counts, mastery points or dtrack hits at a
moment, so damage before and after a change can be compared. One is taken at
login and on reload (skipped while AFK). Identical captures aren't saved again.

| Command | What it does |
|---|---|
| `cdt snapshot` | capture instinct, mastery and dtrack hits now |
| `cdt snapshot list [type]` | the last 50 (`instinct`, `mastery`, `dtrack_hits`) |
| `cdt snapshot show <id>` | one snapshot's values |
| `cdt snapshot compare <id1> <id2>` | two snapshots side by side |
| `cdt snapshot clear confirm` | delete every snapshot |

**`cdt dtrack`** sends the game's `dtrack hits` and saves it as a snapshot:
rounds, swings and hits (primary and secondary), hit and crit %, damage per
round, and swings per slot.

**`cdt unstick`**: snapshots and whois briefly hide game output while they read
it. If a capture is interrupted and output stays hidden, this turns every
capture and hiding trigger off.

**`cdt gaias [element|clear]`** (shaman): gaias fury and revenge damage lines
don't say their element, so tell the plugin your focus. Unfocused, crafter and
hunter gaias revenge count as an even mix. Recorded damage keeps its tag.

**`cdt wail`** (4.4.6): the harmer spell **void sermon** shows its damage as
`Your wail ...`, the same word a sonic weapon uses, so tell the plugin which
yours is:

| Command | What it does |
|---|---|
| `cdt wail` | the current setting |
| `cdt wail weapon` | all your wail is a sonic melee weapon hit (the default) |
| `cdt wail sermon` | all your wail is void sermon: negative spell damage, under its own name in every damage table, the meter and mastery |
| `cdt wail both` | each wail line is void sermon if your mana drops by a cast's cost (80, Eldar 60) by the next mana update, else a weapon hit |

- **`both` needs** nothing else giving you mana while you fight: consume
  (corpses for mana) or regen hides a cast, and another spell cast at the same
  moment can make a weapon hit count as void sermon.
- **Groupmates' wail**, and a mob's wail on you, are always a sonic weapon.
- **Recorded damage** keeps the name and type it was given.

---

## Automatic reports

An **event** is a line CDT prints by itself when something happens, like
`[CDT] [Quest | 15.00s | 44qp (+9tier+22daily) | 5,377g | ...]`. Each event can
be turned on/off and sent to a channel (`echo` = your screen only, or any game
command such as `gt`). New installs start with every event on, on echo.
Turning one off only hides the line; data is still recorded.

| Command | What it does |
|---|---|
| `cdt events` | every event's settings |
| `cdt events <event>` | one event's settings |
| `cdt events <event> on\|off` | print it or not |
| `cdt events <event> channel <ch>` | where it goes |
| `cdt events level <option>` / `powerup <option>` | which number the line shows |

| Event | Fires on |
|---|---|
| `level` | each level-up: time, trains, HP/MP/MV, bonus stats |
| `powerup` | each powerup: time, trains and bonus trains |
| `quest` | each quest: time, QP with bonuses, rewards |
| `cp` | each campaign: QP (+daily bonus), rewards, XP |
| `gq` | each GQ you finish: time, mob and win QP, rewards |
| `double` | double XP starting (with minutes, if known) and ending |
| `milestone` | a milestone reached |
| `goal` | a session goal reached |
| `session` | a tracked session's summary (sent only when the channel isn't echo) |
| `wrapup` | the day's totals when Aardwolf's day ends; if offline, up to 3 missed days on your next login |

**Level and powerup numbers**:

| Option | Powerup line | Level line |
|---|---|---|
| `remort` (default) | this remort's powerup number | the level you just reached |
| `lifetime` (powerup) | every powerup ever, from whois | |
| `total` (level) | | every level across all tiers, redos and remorts (e.g. T9+2 R7 level 100 = 16,783) |
| `daily` / `aarddaily` | powerups today, your midnight / Aardwolf's | level-ups today, same |
| `weekly` / `aardweekly` | powerups this week, your week start / Monday US Eastern | level-ups this week, same |

Day and week counts are read from the database, so they survive reloads. A
number that isn't known yet falls back to `remort`. Channel command echo is
hidden by default (`cdt config gag_channel_echo off` shows it).

---

## Channel query listener

Lets chosen players ask your plugin questions on a channel; it answers on the
same channel. Off for every channel until you turn one on, and only players on
the allowlist (or an allowed clan) get answers.

| Command | What it does |
|---|---|
| `cdt listen` | status, channels and allowlist |
| `cdt listen <channel> [on\|off]` | turn a channel on/off (flips if blank) |
| `cdt listen allow\|remove <name>` | add or remove a player |
| `cdt listen clanallow\|clanremove <clan>` | allow or remove a whole clan (reads `roster <clan>`) |

**Channels**: `ftalk`, `gtell`, `spouse`, `clantalk`, `say`.

**Queries players can send**:

| Query | Answer |
|---|---|
| `cdt help` | what they can ask |
| `cdt instinct <from> <to> [<skill>] [<N>wis]` | trains and gold, plus the benefit |
| `cdt mastery <from> <to>` | QP, gold and damage % |
| `cdt potential <from> <to>` | QP to raise potential |
| `cdt report <dps\|dmg\|crit\|healedme\|healedall\|healedgrp>` | your damage meter table (say, gtell, spouse only) |
| `cdt report player <name>` | one player's attacks and heals (say, gtell, spouse only) |
| `cdt customreport <a+b+...> player <name>` | only the figures asked for: dps, dmg, crit, avg, healedme, healedall (say, gtell, spouse only) |
| `cdt heal report player <name>` | whom `<name>` healed in the meter's count, by name (say, gtell, spouse only) |

`healedme` is the heals that landed on you; `healedall` everyone's total,
estimated where it must be (`~`); `healedgrp` the same with each healer on self,
the leader and others. Add `castsonly` to `cdt report healedme`, `cdt report
healedall` or `cdt report healedgrp` for just the numbers of casts, ranked.
Answers name you rather than saying "you".

---

## Gags

Hide kinds of game line from your screen. Hidden lines are still read and
recorded. `cdt gag` lists each category; `cdt gag <category>` flips it.

| Category | Hides |
|---|---|
| `xp` | kill, rare kill, bonus and superhero honour XP lines |
| `gold` | gold from corpses, crumbling, selling, fencing, haggling and splits |
| `loot` | items you get from a corpse |
| `sac` | gold given for a sacrifice |
| `tax` | clan tax taken from your gold |
| `groupdmg` | groupmates' damage lines, and `X is unaffected by <player>'s <attack>!` |
| `groupheal` | `You feel [greatly] healed by <groupmate>'s <spell>.` from a current groupmate only |
| `allheal` | that line from anyone, plus the three heals that name no one |
| `enemydmg` | mobs' damage against your groupmates (a mob hitting you always shows) |

Only one of `groupheal` and `allheal` can be on.

---

## Configuration

`cdt config` shows current settings: class, level, area, lifetime totals.

| Command | What it does |
|---|---|
| `cdt config gag_channel_echo [on\|off]` | hide (default) or show the command echo when a report goes to a channel |
| `cdt config update_check [on\|off]` | off (default): CDT never contacts GitHub by itself; on: look once, 20 seconds after loading, for a newer public version |
| `cdt config update_notice [on\|off]` | on (default): say once when a newer plugin file is on disk (re-read every 5 minutes); off: stop checking |
| `cdt config timezone [offset\|machine]` | show times in a fixed offset (`UTC-8`, `+5:30`, `0`; no daylight saving) or the computer's timezone |
| `cdt config weeklystart <day>` | the day weeks start, in full (`monday` by default); `aardweekly` ignores it |
| `cdt config dateformat <M/D/Y\|D/M/Y\|Y/M/D>` | the order for typed and shown dates (M/D/Y by default) |

Saved names (like "Double XP 2026-02-03") and exports always use
year-month-day.

---

## Data management

### Database

Everything lives in `cirrus_tracker.db` beside the plugin. Before each layout
upgrade the plugin makes a backup; if an upgrade fails it stops writing and
tells you how to recover.

| Command | What it does |
|---|---|
| `cdt db` | path, version and state |
| `cdt db backup` | a timestamped copy now |
| `cdt db backups` | the automatic pre-upgrade backups |
| `cdt db backups prune` | keep the newest 3 automatic backups (manual ones are never removed) |
| `cdt db restore [file]` | show what restoring a backup (newest by default) would do |
| `cdt db restore [file] confirm` | restore it; the current database is saved aside first |
| `cdt db keep` | clear an incomplete-upgrade warning |
| `cdt db verify [days\|all]` | look for rewards recorded twice and check session totals (last 30 days by default; changes nothing) |

Reload the plugin after a restore.

### Archive (`cdt archive`)

Moves old records into a separate SQLite file so the main database stays small
and quick. Archived records drop out of every report until rehydrated. Nothing
happens without `confirm`; you see the record counts first.

| Command | What it does |
|---|---|
| `cdt archive gains <date> [confirm]` | move gains records from before `<date>` |
| `cdt archive dmg <date> [confirm]` | the same for damage data |
| `cdt archive list` | archives made, with ids |
| `cdt archive rehydrate <id\|file>` | copy records back in (safe to run twice) |
| `cdt archive purge gains\|dmg <date> [confirm]` | delete without archiving (can't be undone) |
| `cdt archive path [folder\|default]` | show or set the archive folder |

- **gains** moves sessions, level-ups and powerups, kills, quests, campaigns,
  GQs, other earnings, gold, bloot, deaths, unfinished fights, double XP times,
  tracked sessions and daily totals.
- **dmg** moves attack and damage type slices, area combat, groupmates'
  profiles and attacks, and the pre-v3.9 estimates.
- **Stays**: snapshots, builds, milestones, mappings, lifetime totals, the
  subclass/race/combo/area summaries and your settings.
- `<date>` can't be later than the start of today or of this session.

### Export (`cdt export`)

Writes `cirrus_tracker_export.json` (export time, versions, session and
lifetime totals, and every row of each table).

| Command | What it does |
|---|---|
| `cdt export [table ...]` | every table, or only those named |
| `cdt export list` | the tables, with row counts |
| `cdt export csv [table ...]` | one CSV per table, plus `cirrus_tracker_export_totals.csv` |
| `cdt export path [folder\|default]` | where exports go (the folder must exist) |

Each export replaces the last file. Times in the rows are Unix time.

### Rebuild and reset

| Command | What it does |
|---|---|
| `cdt rebuild gains` | recalculate area stats and the subclass/race/combo gold and XP totals from the records |
| `cdt rebuild dmg` | re-apply current damage type mappings to every recorded attack and recalculate type totals (backup first; a failure is undone) |
| `cdt rebuild` | both |
| `cdt reset session` (or `cdt reset`) | start a new session now; saved data isn't touched |
| `cdt reset db` / `cdt reset db confirm` | show / permanently delete ALL tracked data (snapshots and settings are kept) |

Make a backup (`cdt db backup`) before `cdt reset db confirm` or any purge.

---

## Debugging

| Command | What it does |
|---|---|
| `cdt version [number\|all]` | versions, database path, and release notes |
| `cdt reload` | reload CDT, e.g. after a new version is deployed, instead of File -> Plugins |
| `cdt debug` | debug messages on/off (what the plugin reads from GMCP, triggers and records) |
| `cdt debug dump` | this session's counters and state |
| `cdt debug sql <query>` | run SQL on the database (a write query changes data for real) |
| `cdt debug rollup` | update today's daily totals now |
| `cdt debug worthtrace [on\|off\|report\|clear]` | time each worth update, reward claim and "other" booking to the millisecond |
| `cdt debug grouplog [on\|off]` | log every GMCP group and vitals update, game line and command to a file |
| `cdt debug double [on\|off]` | log each double XP, silently, from start to 20 s after CDT ends it |

---

## License

GPL-3.0, see [LICENSE](LICENSE).
