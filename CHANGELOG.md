# Changelog

Newest first. The database version is shown for each release; an upgrade backs the database up first.

## 4.5.1 - cdt potential

Database version: 39

### Upgrading

- No database change from 4.5.0.

### New

- cdt potential <from> <to> gives the QP cost of raising potential from one value to another, from the game's cost table (0 to 450): cdt potential 0 20 is 6,750 QP. Allowed players can ask it over cdt listen too, as they can cdt instinct and cdt mastery.

## 4.5.0 - cdt update gets the newest public version

Database version: 39

### Upgrading

- No database change from 4.4.9.

### New

- cdt update looks up the newest public version of CDT (the version.json in its GitHub repository) and says whether you have it. cdt update install downloads that version, checks it against the repository's checksum and that it is CDT, and puts it in place; your current file is kept beside it as cirrus_data_tracker.xml.<date>-update.bak. Type cdt reload to load it. It never reloads by itself, the web requests run in the background, and if anything does not check out or the file cannot be written, nothing changes.
- A copy running ahead of the public version (your own build) is left alone: cdt update install force installs the public one anyway.
- cdt config update_check [on|off]: on makes CDT look once, 20 seconds after it loads, and say if a newer version exists. Off by default, because it contacts GitHub.
- cdt help update explains it.

## 4.4.9 - Tells you when a new version is waiting

Database version: 39

### Upgrading

- No database change from 4.4.8.

### New

- A deploy replaces the plugin file on disk, but the client keeps running the copy it loaded, so nothing said a new version was waiting. CDT now reads its own file when it loads and again every 5 minutes. If the file on disk differs from the one that loaded, it says so once: 'CDT 4.4.10 is on disk; you're running 4.4.9. Type cdt reload to load it.', or, when the version number is the same (a fix inside a version), 'The CDT file on disk has changed since it loaded (still 4.4.9)'. cdt help repeats it under its header until you reload, or the file is put back. It never reloads anything by itself, and waits for the next check if you are in a fight.
- cdt config update_notice [on|off] turns this on (the default) or off. Off, the 5-minute timer is stopped and the file is not read again; the notice and the cdt help reminder go with it.

## 4.4.8 - Hiding never takes the wrong line

Database version: 39

### Upgrading

- No database change from 4.4.7.

### Fixes

- With cdt gag groupdmg, enemydmg, groupheal or allheal on, CDT hid a line by deleting the last line in the output. If another plugin had already hidden that line, the delete took the visible line above it instead. In a big group, with a damage line every moment, this ate the output from the bottom: text slid back up the screen and new lines never appeared, until a reconnect. The ASCII map's capture does this while it waits for the end of a map, so a missed map end set it off. CDT now deletes only when the last line in the output is the one it matched (all of its rows if it wrapped, where before only the last row went), and otherwise leaves the output alone and says so once a session.

## 4.4.7 - Regeneration, and scrolls by what they healed

Database version: 39

### Upgrading

- Database version 38 to 39: the groupmates' heals table gets a spell column (everything already in it is incomplete healing). A backup is made first, as with every upgrade.

### New

- Regeneration is tracked. On you, 'A great healing power courses through your veins leaving you greatly regenerated. [N]' is the same line whoever cast it, so its caster is whoever's mana dropped by regeneration's cost (1,100, an Eldar's 825) in the next group update: you, a groupmate, or 'unknown' when no one's did; solo, it's yours. Yours on another player ('...courses through the veins of Bob leaving him greatly regenerated. [N]') is recorded like your incomplete healing on them. A groupmate's regeneration on anyone else shows no line: it's found in their mana, and counts the hp gain of the player it most likely healed (the biggest hp jump in their room in that update), or 4,600 when no one's hp jumped.
- Regeneration counts in the damage meter's Healing (with estimates), its detail view (Regen~ / HP~), healedall, healedgrp and castsonly, and in cdt heal report (a Regen~ column in healedall; their regeneration on others in player). Scrolls of regeneration are read like scrolls of incomplete healing.

### Improvements

- A heal scroll read on someone else counted 3,400 a heal, but a scroll's heals depend on the reader's stats. It now counts what its target's hp rose by in the next group update (for incomplete healing, at most 3,400 a heal), and only when that can't be measured (the target isn't in your group data), 3,400 a heal (regeneration 4,600). A scroll's heals on you were, and are, exact from their lines.
- A groupmate's heal size, worked out from the hp jumps their casts caused, now comes from their incomplete healing casts only.

### Fixes

- necrocide's instinct was left out of the damage estimates as an unmapped damage word, though its damage has been mapped since 4.4.3.
- The group's combined attack ('Your group's combined attack ...') showed in your own attacks in the damage meter's detail view and in cdt dmg meter report player, though it is the whole group's and was already left out of your damage and DPS. It no longer does, so your attacks' shares add up to your own damage.

## 4.4.6 - Void sermon or a sonic weapon

Database version: 38

### Upgrading

- No database change from 4.4.5. void sermon is added to the built-in damage type mappings (negative, spell) when CDT loads.

### New

- cdt wail: the harmer spell void sermon shows its damage as 'Your wail ...', the word sonic weapons use, so it was all counted as sonic melee. cdt wail weapon keeps that (the default); cdt wail sermon records all your wail as void sermon, negative spell damage, under its own name in every damage table, the meter and mastery; cdt wail both tells them apart by your mana: a wail line counts as void sermon when your mana drops by a cast's cost (80, Eldar 60) by the next mana update. 'both' needs nothing else to give you mana while you fight: consume (corpses for mana) or regen hides a cast, and another spell cast at the same moment can make a weapon hit count as void sermon. cdt wail on its own shows the setting.
- Groupmates' wail, and a mob's wail on you, are always a sonic weapon: there is no way to tell. Damage already recorded keeps the name and type it was given.
- void sermon's instinct is no longer left out of the damage estimates for an unmapped damage word.

## 4.4.5 - Solo and group damage

Database version: 38

### Upgrading

- Database version 37 to 38. The tables holding damage by attack, by damage type and damage taken are rebuilt with a solo/group column; a backup is made first, as with every upgrade, and it takes a moment on a large database.
- Damage already recorded is labelled solo or group where its 30-minute slice was all one or the other, and 'not split' where the slice mixed both or is from before v3.9. The group's combined attack is labelled group, and so is a groupmate's damage taken.

### New

- Add solo or group (or grouped) to a cdt dmg view to see only the damage recorded while you were solo, or in a group: cdt dmg combo gargoyle/necromancer solo, cdt dmg area 10 group, cdt dmg report solo, cdt dmg taken group. It goes with the level, date and tier filters, on report, subclass, race, combo, level, area, attacks, types and taken. It isn't for cdt dmg player or players, and the pre-v3.9 estimate and the instinct and mastery sections of report can't be split, so they're left out.
- What is exact: the totals in report, subclass, race, combo, level and area (damage, damage received and time) are exact for all your history, because the area table has always kept solo and group fights apart. Race and combo take each slice's race from its attack rows; a slice with two races (a race change partway) is shared between them by their damage.
- The breakdowns by attack, damage type and damage taken are exact from 4.4.5 on. Older damage in a slice that mixed solo and group fights can't be split, and is shown under the table as 'not split', with how much it is and what share of the total. New damage is always split.

## 4.4.4 - A lighter XP meter

Database version: 37

### Upgrading

- No database version change from 4.4.3. On first load CDT adds one index to the levels table, which takes a moment on a large database.

### Fixes

- The XP meter's Pup avg and Pups rows read the whole levels table on every refresh (every 3 seconds), and Pup avg for a window with no pups in it on every second. With many pups recorded and the database on a network drive this froze the client for a fraction of a second every few seconds. The levels table now has an index on type and end time, so each query reads only the pups in its window, and a window with no value is remembered until the next refresh like any other.
- CDT's database connection keeps 16 MB of pages in memory instead of SQLite's default 2 MB, so reports and the meter don't fetch the same pages from disk over and over.

## 4.4.3 - Heal reports for a channel

Database version: 37

### Upgrading

- No database change from 4.4.2.

### New

- cdt dmg meter auto [on|off]: the damage meter starts counting by itself when play begins, after a reload, a restart or a reconnect, instead of waiting for Start. Off by default and saved with the meter's settings; with no on/off it flips. Turned on while the meter is idle, it starts now. A meter already counting or paused is left alone.
- cdt reload reloads CDT from the command line, e.g. after a new version is deployed, instead of going to File -> Plugins.

### Improvements

- The damage meter's heal reports are three: cdt dmg meter report healedme [ch] sends the heals that landed on you, exact (titled Heals on <your name>); healedall [ch] sends everyone's heals in the meter's count, each healer's total and casts, biggest first (titled Heals w/ estimates, ~ where estimated); healedgrp [ch] sends each healer's heals on self, on the leader and on others as three figures, self/leader/others (titled Group heals). The leader is whoever leads the group when it's sent. Each takes castsonly. The older names (heal, healing, healest) are gone, and so is cdt heal report group.
- customreport takes healedme and healedall as figures, e.g. cdt dmg meter customreport healedall+dps player Bob: Heals: ~9.5k on self (2 casts), ~14.3k on leader (3 casts), ~23.8k on others (5 casts) | Attacks (dps): .... report player adds the same heals after the attacks. The sections are labelled and come in the order asked for.
- The damage meter's detail view by Healing splits a groupmate's heals by who they landed on: on self, on leader, on others - you and on others - not you (on leader - you when you lead); your own row, on self, on leader and on others. A cast's target is the biggest hp jump in the healer's room when their mana drops, as guessed since 4.4. Estimates show ~.
- cdt heal report's commands are named for what they show: healedme (who healed you; was players) and healedall (groupmates' healing, estimated from the group data; was group). The damage meter's count, each healer's heals on self, the leader and others, is cdt dmg meter report healedgrp (or cdt report healedgrp over cdt listen).
- cdt heal report sends one line to a channel, put last: your own heals; player <name>, their heals on self and on others; healedall, each groupmate's total healed and casts. Over the scope, all time by default. healedme stays on your screen. The tables on your screen are as before.
- Over cdt listen: cdt report healedme|healedall|healedgrp [castsonly], customreport with healedme and healedall, and cdt heal report player <name> (the meter's count, broken down by player healed: on you exactly, the rest by each cast's likely target, ~), on say, gtell and spouse. The cdt help reply lists them.
- castsonly: add it to a heal report that shows several players, sent as one line (to a channel, echo or over cdt listen), for just the numbers of casts, no hp healed: cdt dmg meter report healedme castsonly [ch] (Heal casts on <you>, each healer's casts on you and your own on yourself, with their share), cdt dmg meter report healedall castsonly [ch] (Heal casts w/ estimates: each healer's total casts), cdt dmg meter report healedgrp castsonly [ch] (Group heal casts: each healer's casts on self, on leader and on others), and cdt heal report healedall castsonly [scope] <ch> (Groupmates' heal casts). Ranked by casts; a ~ marks an estimated count (a groupmate's casts seen only in their mana; casts on you, your own and scrolls are exact). The word can come anywhere after the report name. The tables on your screen, and the single-player reports, don't take it. Allowed players can ask for it over cdt listen: cdt report healedme castsonly, cdt report healedall castsonly, cdt report healedgrp castsonly.
- A line CDT sends to a channel is at most 999 characters: a longer report goes on to a second line (and a third), split between entries, each follow-on line starting [CDT]; past three, the last is cut, ending .... A very long line can get you disconnected; if the connection drops within 10 seconds of CDT sending to a channel, a [CDT debug] note says what was sent and how long each line was.
- Reports name you rather than saying 'you', since whoever reads the channel, or asked over cdt listen, isn't you.
- cdt report and cdt customreport asked over cdt listen on say were never answered: CDT checked say against its report channels, which it isn't one of, and printed 'say is not a channel' instead. A listen query is now answered on its own channel.
- Clicking a player in the damage meter's group view opens their detail view on the same category: from Healing, their heals; from DPS, their DPS.
- Every line CDT sends ends with a colour reset, so a report's last colour no longer bleeds into what follows: $C on the game's channels (gtell, clantalk, ftalk, spouse), which puts the channel back to its own colour for everyone (help colors); @C on say, its bright cyan; @w, plain grey, everywhere else (echo, tells, a channel added with cdt config channels add), where $C would show as text.
- healedgrp reads Name: self/leader/others, e.g. 1.Bob: ~9.5k/~14.3k/~23.8k. Each figure is in the colour of its word in the header (self yellow, leader green, others cyan), names in bright white, always three figures with 0 where there were none, and no per-player total: shorter and easier to read. castsonly is the same with casts. A group of many players still splits between rows.

### Fixes

- necrocide is mapped to negative; its damage showed as unknown.
- A built-in damage type mapping added in a new version now reaches an existing database when CDT loads, and damage already recorded as unknown under that attack is filed under its type (the damage type totals follow). A mapping you set yourself with cdt dmg map is left alone.
- The million monsters double ('You hear a loud roar from <player> and the world appears calmer for a while. Aardwolf rejoices in the death of another MILLION monsters.') wasn't recognised: CDT looked for the sentence only at the start of a line. The hour then ran under whatever double came along next, and with a superhero's 15 minutes the double session ended at 15. A copy of the line in someone's whois history doesn't count.
- The million monsters' hour replaces any double running, as the game does: the double, and its session, now end an hour after it, not when the double it replaced would have (a superhero's 15 minutes ended the session at 15). Another double that runs longer than the one on moves the session's end out too.

## 4.4.2 - A tidier damage meter

Database version: 37

### Upgrading

- No database change from 4.4.1.

### Improvements

- Right-click the damage meter for 'Healing: Include Estimates'. With it on, the Healing tab adds each groupmate's estimated healing on others (their casts on others times their heal size, plus scrolls; shown with ~) to their heals on you, and your own heals on others to your row. It's off by default and remembered. Channel reports stay exact.
- The detail view shows only the lines for the tab you're on: Dmg / Time / DPS on DPS and Damage, Hits / Crits on Crit% and Avg/Hit, the heals on Healing. Taken / Per min shows on every tab. The figure in the header and the window's total follow the tab too: on Healing, the total is the healing.
- A detail page with nothing to list (a groupmate who healed nobody, or just after a reset) says 'No heals yet' or 'No damage yet' instead of an empty heading.
- The total at the bottom right no longer runs into the resize corner.
- cdt dmg's command list now includes heal as a meter report sort, and mentions the meter's right-click options.

## 4.4.1 - Healers without pretitles

Database version: 37

### Upgrading

- No database change from 4.4. The first time 4.4.1 loads, it puts right the heals already recorded (below).

### Fixes

- A healer or heal target with a pretitle ('Dark Lord Bob', 'Ranger Alice') was recorded under the whole pretitle, apart from the player's other heals, and their known heal size wasn't found. They're recorded by name now, and heals already recorded under a pretitle are moved to the name.
- Heals recorded in a quiet stretch with no fighting could be filed under an earlier half hour, which the level and tier filters go by. They're filed under the half hour they happened in now, and heals already recorded are put right.
- cdt heal report group's Likely healed list could run past 80 columns.
- With the gains meter open, CDT could go on seeing its database as it was a moment before, missing changes another program made to it (such as the one-off group log import) until reloaded, and its own records could fail to save meanwhile.

## 4.4 - Groupmates' healing

Database version: 37

### Upgrading

- Your database upgrades to v37 the first time 4.4 loads, after a backup is made (cdt db restore rolls it back).

### New

- Groupmates' heals on other players are counted. A cast shows in the group data as their mana dropping by 950 (712 or 713 for an Eldar, learned per player, and a race change is noticed), or by close to it once they're known to heal: an estimate, as some are missed and wasted heals count. A healer's heal size is their heal on you, or else worked out from the hp jumps their casts cause. A heal scroll read in your room ('X recites a scroll of 3*incomplete healing on Y') counts exactly: 3 heals of 3,400.
- cdt heal report group [scope] lists each groupmate's casts, how many were on others, their heal size, their healing on others, and who each cast most likely healed (a best guess from the biggest hp jump in the healer's room, right about 3 times in 4). cdt heal report player <name> adds the same for one player, and the damage meter's detail view by Healing shows it per groupmate.
- A heal scroll's heals on you are recorded as 'incomplete healing (scroll)', apart from cast heals, and count in every total.

### Fixes

- A heal on you from a groupmate with a title before their name ('Dark Lord Bob') was recorded under the whole title, apart from that player's other heals.

## 4.3.1 - Bloot colours, and doubles that ended early

Database version: 36

### Upgrading

- No database change from 4.3.

### New

- Bloot tiers show in the game's colours and brackets in every bloot report, ((Godly)) included, and rank in the game's order: Mythical is now below Fabled.
- cdt debug double on|off: a diagnostic that records each double to a file, silently, from its start to 20 seconds after CDT decides it ended: every game line, and why CDT ended it.

### Fixes

- A superhero's double ('experience is doubled in honor of the new superhero') could end early in CDT, the sooner the faster you kill. Its bonus line ('... in honor of <player> reaching superhero') wasn't taken as the double's bonus, and with kills under half a second apart, one kill's bonus check could judge the next kill before its bonus line arrived. A double the game gave a length for is also no longer ended by kills without a bonus line before that time is up.
- A reload, a disconnect or cdt reset session no longer ends a double in CDT; the game's double carries on, and so does CDT's. If its time ran out while you were away, it's recorded as ending then. With no length known, your next kills decide.
- A double's end time was never saved: its record stayed open until the session ended. A double CDT ends because kills show no bonus is now recorded as ending at the last bonus.
- When CDT decided on its own that a double was over, its messages could cut a line that was still arriving in two, and neither half was then recognised (a bonus XP line went uncounted). They now wait for the line to finish.
- cdt reset session during a double no longer drops the double's tracked session unsaved.

## 4.3 - Heals and damage taken

Database version: 36

### Upgrading

- Your database upgrades to v36 the first time 4.3 loads, after a backup is made (cdt db restore rolls it back).

### New

- Incomplete healing on you is recorded, yours ('You are healed by your magic touch') and other players' ('You feel healed by <player>'s magic'). It is the only heal whose line names the caster.
- cdt heal report shows your own heals: total healed, casts, average per cast and highest heal. cdt heal report player <name> shows the same for <name>'s heals on you.
- cdt heal history all|self|player <name> [n] lists the last heals (10 by default, up to 100), with time, healer, spell and amount.
- cdt dmg player <name> shows what <name> healed you for.
- cdt heal report players lists everyone who healed you, most first. Every heal report takes a scope: session, day, week, month, year, or the Aardwolf ones.
- Damage taken is recorded by who hit you and with what: cdt dmg taken shows it by mob, attack, damage type, race and subclass. Hits on your groupmates are recorded too: cdt dmg taken player <name>, and cdt dmg taken players to see who takes the most.
- An attack the game prints capitalised ('Wrath', 'Colourful Spray') now finds its lower-case damage type mapping.
- The damage meter has a Healing category beside DPS, Total Dmg and Crit%: in the group view, each player's heals on you (yours on yourself on your row); in the detail view, a player's heals on you by spell, with casts. cdt dmg meter report heal sends it to a channel.
- cdt debug grouplog on|off: a diagnostic that writes every GMCP group update, game line and command to a file, to study whether groupmates' heals on others can be detected from their mana.
- Your heals on other players ('Bob is healed by your magic touch') are recorded too, and the meter's detail view by Healing lists them on your row, by player, under your heals on yourself.
- Two new gags for heals on you, no longer hidden by cdt gag groupdmg. cdt gag groupheal hides 'You feel healed by <player>'s <spell>' when the player is in your group. cdt gag allheal hides that line from anyone, plus 'The life within you is renewed', 'You feel your life restored' and 'A warm feeling fills your body'. Only one of the two can be on; cdt help gag lists exactly what each covers.

### Fixes

- A heal from a groupmate was read as a damage line: it was recorded as their damage, and hidden by the groupdmg gag.
- After a new session started (a reload or relog) with the damage meter running, groupmates' rows disappeared from the meter until they dealt damage again.
- cdt reset session with the damage meter running, if the meter was started partway through the session: the meter read 0 until you had dealt more than the session had before Start. What it had counted now carries over.

## 4.2 - More reliable gains tracking; melee, spell and skill

Database version: 33

### Upgrading

- Your database upgrades to v33 the first time 4.2 loads, after a backup is made (cdt db restore rolls it back).

### New

- QP, TP, trains and pracs tracking is more reliable: rewards are matched to the game's updates however their messages are ordered or delayed.
- cdt db verify checks each session's QP, TP, trains and pracs against what the game reported, for sessions from 4.2 on.
- cdt dmg player <name> shows melee, spell and skill damage, each damage type's share, and the most recent day's damage lines and types.
- The damage meter shows your damage taken and damage taken per minute, in your Detail view and in cdt dmg meter report player.
- cdt debug worthtrace on|off|report|clear: a diagnostic that records how the game's reward messages are timed.

### Fixes

- The snapshot at load (instinct, mastery, dtrack hits) could give up on a slow reply and let it scroll by unsaved. It now waits up to 15 seconds, and if something still doesn't reply, it says what was missed and the command to retry.
- cdt dmg player counted mind force, mental energy and backstab as spell damage.
- Spells and skills are told apart. cdt dmg map <attack> <damtype> <category> corrects any attack.
- A spell you had mapped yourself (cdt dmg map ... spell) wasn't boosted by its own instinct in instinct estimates and projections.

### Data repaired on upgrade

- v32: attacks get their spell or skill category (backstab, spiral and counter strike added as skills), including damage already recorded. Mappings you made yourself are kept.

## 4.1.2 - Bloot by area, doubled rewards removed

Database version: 31

### Upgrading

- Your database upgrades to v31 the first time 4.1.2 loads (from 4.1, through v30 on the way), after a backup is made (cdt db restore rolls it back).

### New

- New cdt gains area bloot [scope]: bonus loot by area, with its best tier and drop rate, and under each area the mobs that dropped it, by tier. Every drop ever recorded has its mob.

### Fixes

- cdt gains best is faster again: the biggest kill comes out of the same pass over your kills as the best days, instead of reading them all a second time.
- cdt help gains was missing area quest, best and compare. They are listed, with area bloot.
- cdt gains compare ... full ended with two separator lines.
- cdt db verify also looks for a quest, campaign or GQ's TP, trains and pracs recorded a second time as 'other'. Its QP check matches within seconds instead of two minutes, like the others, and its trains check looks at level-ups only.

### Data repaired on upgrade

- v31: before 4.0, a quest, campaign or GQ's TP, trains and pracs could also be recorded as 'other' a moment before the reward line, counting them twice. v29 cleared the QP; the rest go now: an 'other' row the same size as the reward, booked from 3 seconds before it to 1 second after, one per reward, with the session and daily totals that counted it.

## 4.1.1 - Doubled level-up trains, and fixes to 4.1

Database version: 30

### Upgrading

- Your database upgrades from v29 to v30 the first time 4.1.1 or later loads, after a backup is made (cdt db restore rolls it back).

### Fixes

- Level-up and powerup trains and pracs were still sometimes recorded a second time as 'other': when the worth update came more than 1.5 seconds before the level line, it was booked before the level-up claimed it. Each gains line now claims its own amount as it arrives.
- cdt db verify ran for up to a minute on a large database. Its trains check now uses the time index and takes a moment.
- The day's wrap-up left out kills from the last half minute before Aardwolf's midnight (still unsaved), and could say nothing for a day that had only those.
- cdt gains best ignored a word after the scope (cdt gains best week gt), and cdt db verify read a count it couldn't make sense of as 30 days. Both show their usage now.
- cdt gains compare month: the last column's heading ran into Change.
- cdt gains area quest listed only the 15 most visited areas. It lists them all, with a total.
- cdt gains best lagged on a big database: it worked out a date for every kill. It now adds the kills up by quarter hour first.

### Data repaired on upgrade

- v30: the 'other' trains and pracs that doubled a level-up since v27 are removed, by v27's rule, with the session and daily totals that counted them. Powerups are left alone: they come seconds apart, too close to tell a copy from a real gain.

## 4.1 - Records, comparisons and the daily wrap-up

Database version: 29

### Upgrading

- No database change from 4.0. Coming from 3.9, the upgrade described under 4.0 runs first.

### New

- New cdt db verify [days|all]: looks for rewards recorded twice, and compares each session's saved totals with the records it holds. It changes nothing.
- New cdt gains best [scope]: your fastest quest, campaign, GQ win, powerup and level-up, your best day for XP, QP, gold, kills, powerups and quests, and your biggest kill and richest quest.
- The tables (cdt gains report qp 7day ...) have an Average row, per day or week you played, and show the best one in green.
- New cdt gains area quest [count|qp|time] [scope]: the areas your quests send you to, how often, their base QP, how long they take and what they paid.
- cdt gains history tp and cdt gains history trains: where each trivia point and train came from, newest first.
- New cdt gains compare <day|week|month|year|aard...> [full]: this period so far against the last one up to the same point, with the change and the last one's full total; full compares the last two finished periods.
- New event: wrapup, one line with the day's totals when Aardwolf's day ends (or when you next play, if you were offline). cdt events wrapup channel gt, or off.

### Fixes

- cdt dmg meter report <channel> sends a DPS report there; before, it printed locally and sent nothing. An unknown sort is refused instead of quietly becoming DPS.
- cdt gains meter named the Current Double filter 'Session -4' (and a saved track 'Session -3').

## 4.0 - Reports, automatic events, archive and export

Database version: 29

### Upgrading

- Your database upgrades from v23 to v29 the first time 4.0 or later loads, after a backup is made (cdt db restore rolls it back).

### New

- cdt events replaces cdt toggle and cdt channel: every automatic report in one list, each with on/off and a channel.
- Powerups are their own event, separate from levels. Your old settings carry over.
- Level and powerup lines can number by total, lifetime, daily or weekly count, in your time or Aardwolf's.
- Double XP alerts can go to a channel.
- cdt config holds gag_channel_echo, timezone (was cdt timezone), weeklystart and dateformat (M/D/Y, D/M/Y or Y/M/D), and lists where the other settings live.
- cdt gains history <type> [n] lists recent events. 'all' shows a kill with its gold, bloot and specials on one row.
- cdt flags compares XP and QP with and without OPK/HARDCORE, for any tier.
- cdt archive now works: move records before a date to an archive file, list, rehydrate and purge them, and pick the folder with cdt archive path.
- cdt export writes JSON, cdt export csv writes one CSV file per table, and cdt export path sets the folder.
- cdt build set <name> subclass|race <x> sets who a build is for.
- The damage meter window comes back after a reload if it was on, with its size, tab, sorts and minimalist mode. It comes back ready: press Start.
- Help covers every command.

### Reports

- Every gains report takes a scope: all (the default), session, day, week, month, year, aardday, aardweek, aardmonth or aardyear. (day was today; today still works.)
- A number before a scope shows a table of that many, newest first, with the total underneath: cdt gains report qp 7day, cdt gains report 12 month, cdt gains report xp 10aardweek.
- When one kill finished a campaign and won a GQ, the GQ's win rewards were counted as the campaign's. Each now gets its own.
- New cdt gains report tp and cdt gains report trains: where your trivia points and trains came from, with every scope and the tables.
- cdt gains xpmeter is gone: it only turned the same window on and off. Use cdt gains meter on, off or the new toggle.
- Automatic pre-migration backups are pruned to the newest 3; cdt db backups prune does it now. Backups made with cdt db backup are kept separately and never removed.
- cdt dmg meter customreport <dps+dmg+crit+avg> player <name> [channel] shows that player's attacks with only the figures you pick, in your order, the first sorting, instead of report player's full dmg/dps/crit/avg for each. avg shows damage per hit with the hit count in brackets. It works over a listened channel too, as cdt customreport.
- cdt config channels add <name> allows a channel of your own for reports, alongside the built-in echo, gtell, clantalk, ftalk and spouse. Type it in any case; the game gets the spelling you added, and cdt config gag_channel_echo applies as it does to the others.
- A GQ win's QP, TP and trains, or a campaign's, could also be recorded as 'other' when another gain had arrived a moment before, counting them twice. Each gain now waits its own full time.
- The same rewards were counted twice again when any kill landed between the reward and the check: a worth update that didn't touch a currency threw away what was owed on it. Credits now last until nothing has added to them for 20 seconds.
- cdt gains report <scope> is an overview of everything. All-time views end with period tables.
- The qp, quest, cp and gq reports break QP into its bonuses (daily blessing included) and show gold, TP, trains and pracs.
- The xp report splits kill, rare, double XP, blessing, honour and campaign XP.
- report pup is now report powerup. New report bloot counts bonus loot by tier.
- cdt gains report <type> <channel> sends that report's own one-line summary.

### Data repaired on upgrade

- v24-v25: the first campaign of the day's bonus QP was recorded as 'other'. Past bonuses are moved onto the right campaigns.
- v26: session counters ran on between sessions, so session rows and daily totals were inflated. Both are corrected.
- v27: level-up and powerup trains counted twice are removed and damage type times are repaired. XP by source, kill specials and GQ joins are recorded from now on.
- v28: level-ups and powerups saved 0 for the XP and kills earned during them. They are rebuilt from your kills.
- v29: rewards that were also recorded as 'other' QP are removed, with the session and daily totals that counted them.

### Fixes

- All-time DPS read far too low: a fight's time was counted once per attack used.
- The gains and damage meters no longer blink when one is on top of the other.
- A damage meter running when a new session started (a reload, relogging, or Start before login) read 0 until you out-damaged the old session. It now carries its count over.
- cdt snapshot while AFK stops straight away, and an unchanged dtrack capture isn't saved twice.
- cdt listen clanallow no longer allowlists an unrelated line when no roster comes back.
- cdt flags no longer errors on deaths, which now record OPK/HARDCORE.
- cdt reset session ends the running session first, and cdt reset db clears every table.
- cdt gains track retention no longer errors, and a track name ending in a number keeps it.
- The xp, gold, loot, sac and tax gags hide their lines, and a goal line obeys cdt events goal off.
- cdt rebuild dmg re-applies damage type mappings instead of emptying the damage tables.
- Odd characters shown in place of dashes are plain now.
- A tracked session that ends with nothing earned says so, instead of an empty rates heading.
- A word that isn't a channel is no longer sent to the game as a command: 'cdt gains report qp daily' ran the game's own daily command with the report as its argument, invisibly. It now says so and suggests the scope you meant.
- A mistyped command repeats what you typed and suggests the nearest one.
- Quests, campaigns and GQs are timed to a fraction of a second, as levels and powerups already were. Durations that were only ever measured in whole seconds no longer print a misleading .0.
- Campaigns were never timed at all: every one recorded 0. They are timed from the moment the campaign giver hands one out (whoever yours is), and the CP event line shows how long it took.
- The time left on campaign info, cp check or cp brief names the campaign you are on (now + time left is its deadline), so looking again never restarts its clock, and a different one is recognised. A campaign whose start was never seen records an unknown duration rather than a guess.
- CDT says on screen when it sees a quest, campaign or global quest start and end, as well as sending the event.
- Joining a global quest you are already on doesn't restart its clock or lose its mob QP, and joining a different one records the one you left.
- The million monsters double ('Aardwolf rejoices in the death of another MILLION monsters.') starts double XP tracking and its auto session, as the other doubles do. So does 'For the next N minutes all experience is doubled!!'.
- Someone mentioning double exp on a channel ('is double exp on?') no longer starts double XP tracking: only the game's own announcements do.
- A global quest that never finishes (the quest ended with mobs left, or you disconnected) stayed open for good, and its mob QP counted as 'from the GQ running now' in every report since. It is closed 20 minutes after joining and recorded as unfinished.
- Every table fits 80 columns with full numbers, and the report, history, dmg, tier, build, snapshot, listen, archive, export and config screens line up.

## 3.9 - Damage by level and time, tier tracking, fight tracker

Database version: 23

### New

- Damage is kept per character level (superhero as 201) in 30-minute slices, for your attacks, damage types, areas and groupmates.
- Every cdt dmg view takes <minlevel> <maxlevel> and <from> <to> filters. New cdt dmg level.
- Damage from before v3.9 has no level or time: filtered views show an estimate for it in a separate, marked table.
- Damage, kills, levels, powerups, quests, campaigns, GQs and rewards are stamped with tier, remort and redo. Older records get an estimate from their session.
- New cdt tier (a tier in 20-level groups) and cdt redo (redos side by side). T9, T9R7, T9+3 and T9+3R7 also filter every cdt dmg view and build compare.
- cdt build compare takes a <minlevel> <maxlevel> range.
- Deaths are recorded.
- A fight tracker records each kill's mob, room, damage, rounds, combat time and estimated mob level. Kills with no XP count, and fights you leave are logged with why they ended.
- Quests record the OPK and Hardcore bonus QP, shown in the quest report.
- Records note whether you were OPK or HARDCORE at the time, so rates with and without can be compared.

### Fixes

- Instinct/mastery estimates, build compare and the optimizer read damage since the snapshot from slices, which fixes subclass and race filtering.
- A quest, CP or GQ reward is no longer counted twice when its worth update comes first. Old duplicates are removed and the totals they inflated are corrected.
- With autologin off, nothing is sent until you are in the game (commands at the login prompts were read as your name or password).
- Fresh installs no longer log 'no such table: area_stats'.

## 3.8.1 - Damage meter DPS fixes

Database version: 17

### Fixes

- The meter's clock counted live combat twice: DPS could read 0 for minutes, then spike 8-100x.
- Group rows no longer divide a burst by an invented second, and no row shows DPS under 5s of combat (--).
- The Group tab percentage measures whatever the bars measure.
- cdt dmg reset re-baselines the meter instead of leaving it stuck at zero.
- The area-change auto-reset never ran. It does now, and says when it discards data.
- The player channel report shows Avg/Hit instead of a line count, and help no longer offers a hits sort.
- The whois gag is cleared on load, connect, disconnect and save, so a reload mid-whois can't leave output hidden.
- A possessive target ("...to Orzbuk's right arm!") no longer makes your damage look like the enemy's or a groupmate's.
- Build show and compare report trains, QP and gold cost. Compare falls back to subclass-only or multiplier-only data and says which basis it used.
- A dtrack read taken just after the in-game tracker reset (all zeros) isn't stored, and saved ones are removed.
- Snapshot compare spots a dtrack reset between two snapshots and blanks the cumulative deltas.
- Attack names are no longer cut at the wrong verb ("mind force does UNBELIEVABLE things"). DB v17 merges rows already stored.
- The groupdmg gag also hides "X is unaffected by <player>'s <attack>!".
- The meter's Detail tab shows each attack's share beside its bar when sorted by damage or DPS.

## 3.8 - Snapshot gag fixes

Database version: 16

### Fixes

- The snapshot gag no longer swallows output: narrow patterns replace the catch-all, an AFK refusal stops the chain, and the gag is cleared on load, connect and disconnect. New cdt unstick.
- Meter snapshots carry true hit counts (Avg/Hit was always blank).
- Dropped the Hits sort and the bar percentage, which contradicted the bars.
- Pre-migration backups are size-checked.

## 3.7 - Table layout

Database version: 16

### Changes

- Tables declare each column once and values never overflow (23 views converted).
- Avg/Hit in cdt dmg subclass, race, combo and types, and in the meter's Detail tab.
- New cdt dmg race and cdt dmg combo.
- Fixed the meter's group sort and bars, and fake per-attack DPS.

## 3.6 - True hit counts

Database version: 16

### Changes

- Damage lines record true hits (Lines vs Hits vs Avg/Hit). Avg/Hit only uses data from v3.6 on; older rows show -.

## 3.5 - Gaias focus

Database version: 15

### Changes

- Gaias focus for shaman (cdt gaias). Unfocused gaias damage counts as an even mix of the four elements.

## 3.4 - Build optimizer

Database version: 14

### Changes

- cdt build optimize finds the best damage build for a trains budget.
- Migration backup and restore guard (cdt db).
- Fixed dtrack slot capture and the melee swing multiplier. Physical and unmapped instincts are left out rather than counted as zero.

## 3.3 - Builds

Database version: 14

### Changes

- cdt build saves and compares instinct and mastery builds, and imports instinct-builder.com links.
- Subclass instinct map, shared instinct pools, race on attack stats, and one shared damage projection engine.
- 'class' is shown as 'subclass' everywhere.

## 3.2 - Instinct and mastery estimates

Database version: 13

### Changes

- Damage estimates from instinct and mastery, scoped to each snapshot. Duplicate snapshots are merged.
- New cdt instinct project, cdt mastery project, cdt dmg map usage and cdt dmg purge.
- The epic attack is left out of DPS, and the DPS share sort is fixed.

## 3.1 - Snapshots and listen

Database version: 11

### Changes

- One snapshot system for instinct, mastery and dtrack: cdt dtrack and cdt snapshot (list, show, compare, clear), taken automatically at login.
- cdt listen answers questions on channels for allowed players and clans.
- Player lookups ignore case.

## 3.0 - Command restructure

Database version: 9

### Changes

- Commands grouped under cdt gains, cdt dmg and cdt archive.
- Damage tracking, archive and purge.

## 2.2 - Gold and XP by subclass

Database version: 8

### Changes

- Gold and XP in subclass, race and combo stats, an area summary table, and a rebuild command.

## 2.1 - Tracked sessions

Database version: 5

### Changes

- Tracked sessions, whois lifetime totals, automatic double XP tracking and a database wipe command.

## 2.0 - Milestones and QP report

Database version: 4

### Changes

- Future milestones, a QP report, race and combo tracking, precise timing and channel toggles.

## 1.0 - First release

Database version: 3

### Changes

- XP, gold, quest, CP, GQ and subclass tracking, estimates, and solo/group splits.
