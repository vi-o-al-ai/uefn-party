# Party Island: Design and Implementation Plan

**Status:** Draft v2 for review. Decisions that need your call are tagged `[DECIDE]`.
**Target:** A 2 to 4 player, turn-based board game with mini-games in the style of classic Mario Party (Mario Party 1 to 3 and Mario Party Superstars rules), built in UEFN with Verse.

> IP note: the *mechanics* of a board-plus-mini-game party game are fair to use. Nintendo's names, characters, music, art, and board layouts are not. Everything below uses Fortnite-flavoured placeholder names, and no Nintendo asset or text may ship. Names are placeholders; rename freely.

**Revision notes (v1 → v2).** A five-lens review (rule fidelity, UEFN feasibility, edge cases, pacing, clarity) produced 25 upheld findings. The changes: the Vault is now a countable space (it was contradictorily listed as a non-counting node); Showdown antes, Wheel of Fate and Bar Shuffle now follow the Mario Party rules; the final-stretch event no longer fires on turn 1 of a 5-turn match; a lobby flow and a results sequence were added; late joiners use Fortnite's Spectate setting (there is no "join in progress off"); Verse UI input modes, camera targeting, team configuration, eliminations and respawns were corrected or added; leavers mid-turn and mid-mini-game are specified; idle players now have something to watch and do; the timing budget includes Showdowns, lobby and results; the mini-game roster was resized to the real type mix.

---

## 1. Glossary (Mario Party concept → Party Island name)

| Mario Party | Party Island (placeholder) | Notes |
|---|---|---|
| Star | **Llama** (Golden Llama) | The win condition currency. Bought for 20 Bars. |
| Coins | **Bars** (Gold Bars) | Earned on spaces and in mini-games. Floor at 0, no cap. |
| Dice Block | **Dice** | Random 1 to 10, shown as a UI roulette. |
| Blue Space | **Blue Space** | +3 Bars (+6 in the final stretch). |
| Red Space | **Red Space** | -3 Bars (-6 in the final stretch). |
| Happening / Event Space | **Island Event Space** | Board-specific scripted event. Counts toward the Event bonus. |
| Chance Time | **Wheel of Fate** | Forced swap or transfer between two players. |
| Bowser Space | **Storm Boss Space** | A random bad outcome. |
| Battle / VS Space | **Showdown Space** | Everyone antes Bars, then a battle mini-game for the pot. |
| Item Space / Items | **Gadget Space / Gadgets** | Phase 2 (not in v1). |
| Boo | **The Gnome** | Pay to steal Bars or a Llama from a rival. |
| Koopa Bank | **The Vault** | Deposit 5 Bars when passing over, collect everything when landing. |
| Item Shop | **Gadget Shop** | Phase 2. |
| Mini-game | **Mini-game** | Free-for-all, 2v2, 1v3, Showdown (battle), Duel (phase 2). |
| Last 5 Turns | **Final Stretch** | Doubled space values and a catch-up bonus. |
| Bonus Stars | **Bonus Llamas** | Awarded at the end for mini-game Bars, max Bars, and event landings. |
| Superstar | **Party Champion** | Winner. |

---

## 2. Rules

### 2.0 Lobby and match start

- **Island settings:** Max Players 4. Join In Progress = **Spectate** (UEFN offers Spawn, Spawn on Next Round, Spectate; there is no "off"). The lobby itself is run by Verse, not by the island's game-start countdown.
- **Lobby area:** players spawn on lobby spawn pads in a small enclosed lobby and can walk around. The lobby HUD shows the player list, the turn count, and the countdown.
- **Start condition:** when 2 or more players are present a 60 s countdown runs. It is cut to 10 s when the 4th player joins, and it pauses while fewer than 2 players are present. `[DECIDE]` for a private island, the first player to join is host, picks the turn count, and presses `Start now` instead of waiting.
- **Roster freeze:** when the countdown ends, the players present become the match roster. Lobby spawn pads are disabled, players are teleported to the Start space and held in stasis (section 6.1).
- **Late joiners:** anyone who connects after the roster freeze gets Fortnite's stock spectator view. They never receive a player state, HUD or prompt, and every rule below ignores them.
- **Turn order:** all players roll the Dice at the same time, with a 10 s window and auto-roll on timeout. Highest goes first; ties re-roll among the tied players only. A player who leaves during the roll is dropped. Order is fixed for the whole match.

### 2.1 Players and match setup

- **Players:** 2 to 4 humans on one island instance. No AI players in v1 (UEFN has no scriptable board AI; see section 9 for a later option).
- **Turn count:** 10 turns by default. Options 5 (quick), 10, 15, 20. `[DECIDE]` whether the host picks in the lobby (M3) or the creator fixes it in an editable setting (v1).
- **Starting Bars:** 10 each. **Starting Llamas:** 0.
- **Board:** one board in v1 ("Party Island"); its composition and counts are in section 5.
- **Llama placement:** one Llama is on the board at a time, at one of the 4 Llama spots. When it is bought, it moves to a different random spot.

### 2.2 A player's turn

1. **Turn start.** The mover's camera switches to the orbit camera; everyone else watches from the board overview camera (section 2.9). Banner "Player N's turn" on every HUD. Standings visible on the HUD.
2. **Pre-roll menu.** `Roll` or `View Board`. (Phase 2 adds `Use Gadget`.) Timeout 10 s, default `Roll`.
3. **Roll.** Dice gives 1 to 10. The result is shown to everyone.
4. **Move.** The player hops one space at a time along the board.
   - **Countable spaces** are every space a player can stop on: Blue, Red, Island Event, Wheel of Fate, Storm Boss, Showdown, Vault, Start and (phase 2) Gadget. Each hop onto one consumes one step of the roll.
   - **Passing nodes** (Llama spots, the Gnome, the Gadget Shop) are not spaces. They consume no step and cannot be stopped on. Their prompts fire as the player moves through them (section 2.4).
   - **Moving over the Vault** without stopping deposits 5 Bars (section 2.4). Stopping on it collects the balance (section 2.3). A single move never does both.
   - **Junctions:** the mover chooses a direction from a UI prompt. Timeout 10 s, default is the first listed direction (the main loop).
   - Movement cannot be undone. Rolls are never re-rolled except for ties in turn order and the final tiebreak.
5. **Land.** The space the player stops on resolves immediately (section 2.3). The space's colour is remembered for mini-game selection.
6. **Forced movement** (Island Events, section 5) is a teleport along the board. It fires no passing events, does not resolve the destination space, and does not change any player's remembered colour. Moving backwards follows the path the player actually travelled this match; a player cannot be pushed behind Start.
7. **Turn end.** The mover's camera returns to the board overview, next player.

### 2.3 Space effects (landing)

| Space | Colour for mini-game selection | Effect when landed on |
|---|---|---|
| Blue | Blue | +3 Bars (+6 during the final stretch). Hidden-block check. |
| Red | Red | -3 Bars (-6 during the final stretch). Hidden-block check. |
| Start | Blue | Treated as Blue when landed on. |
| Vault | Blue | Collect the entire Vault balance; the Vault resets to 0. Replaces the Blue +3; no hidden-block check. An empty Vault does nothing. |
| Island Event | Green (roulette) | Board-specific event (section 5). Adds 1 to the player's Event count. |
| Wheel of Fate | Green (roulette) | Three roulettes: Player A (any active player), the exchange, Player B (any other active player). The lander is **not** guaranteed to be involved, as in Mario Party. Exchanges: swap Bars, swap Llamas, swap both, A gives B 10/20/30 Bars, A gives B 1 Llama. If A cannot pay in full or has no Llama, A gives everything they have. With 2 players, A and B are the two players. |
| Storm Boss | Green (roulette) | Roulette picks one: lose 20 Bars; lose 1 Llama (or 20 Bars if none); everyone loses 10 Bars; **Bar Shuffle** (all active players' Bars pooled and split evenly, any remainder is kept by the Storm Boss and removed from play). All losses clamp at 0. |
| Showdown | Green (roulette) | Roulette picks the ante: 5, 10, 20, 30 or 50 Bars. Every active player pays the ante, or all of their Bars if they have fewer. If the total pot is 0, the Showdown is skipped with a "nobody can ante" message. Otherwise a battle mini-game runs **immediately** `[DECIDE, open question 3]`. Payout from the result's tie groups: a sole 1st takes 70% rounded down and the 2nd group splits the rest evenly, any leftover Bar going to a random member of the 2nd group; two or more tied for 1st split the whole pot evenly (odd Bars to a random tied player) and nobody else is paid; a full draw refunds every ante. Pot winnings count as mini-game Bars for the Mini-game Llama. |
| Gadget (phase 2) | Green (roulette) | Receive a random Gadget. |

**Hidden blocks.** Landing on a Blue or Red space has a 5% chance to spawn a hidden block: 80% of blocks hold 20 Bars, 20% hold a Llama. `[DECIDE]` keep or remove for competitive fairness.

### 2.4 Passing events (fire when moved through, never on the final step)

| Node | On passing through |
|---|---|
| **Llama spot (with the Llama present)** | If the player has 20 or more Bars: prompt `Buy Llama for 20 Bars?` (timeout 10 s, default Yes). On purchase the Llama relocates to a different spot. If it cannot be afforded, a "not enough Bars" message plays and movement continues. |
| **The Gnome** | Prompt: `Steal Bars (cost 5)`, `Steal a Llama (cost 50)`, `Pass`. Stealing Bars takes a random 5 to 15 Bars (or all if fewer) from a chosen active rival. Stealing a Llama takes 1 Llama from a chosen active rival who has one. Options the player cannot afford, or with no valid target, are hidden. Timeout 10 s, default `Pass`. |
| **The Vault** (countable space, see 2.3 for landing) | Deposit 5 Bars (or all if fewer) into the Vault balance. |
| **Gadget Shop** (phase 2) | Buy one Gadget. |

### 2.5 Mini-game phase (end of every turn)

After all active players have moved:

1. **Colour roulette.** Players who landed on a Green space spin to be assigned Blue or Red.
2. **Type by colour split:**

| Colour split (4 players) | Mini-game type |
|---|---|
| 4 same | 4-player free-for-all |
| 2 and 2 | 2v2 (same colour = same team) |
| 3 and 1 | 1v3 (the odd one out is solo) |

With 3 players: 3 same → 3-player FFA, 2 and 1 → 1v2. With 2 players: always 1v1 from the FFA pool. Every FFA mini-game must work with 2 to 4 players; 1v3 games must work as 1v2.

Given the section 5 board mix, a player's colour is Blue about 68% of the time and Red 32%, so in a 4-player match roughly 49% of turns are 1v3, 29% are 2v2 and 22% are free-for-all. 3-player matches are about 65% 1v2. 2-player matches are 100% free-for-all. The roster in section 4 is sized to this.

3. **Selection roulette.** A random mini-game of that type, excluding games already played this match until the pool is exhausted.
4. **Briefing.** Rules screen with controls. Players press `Ready`; the game starts when all are ready or after 15 s.
5. **Play.** Players are teleported into the mini-game arena with the game's loadout. Hard time limit per game (30 to 90 s), including any overtime rule.
6. **Rewards.** Winner(s) receive 10 Bars each. FFA ties: all tied players win. 2v2: both teammates win. 1v3: the solo player wins 10, or each of the three wins 10. No reward on a full draw.
7. **Return.** Results screen 5 s, then everyone is teleported back to their board space.

Roulettes and the results screen end early, with a 2 s floor, when every active player presses `Ready`.

**Showdown (battle) mini-games** draw from the FFA pool with the pot rules from 2.3 and run at the moment the Showdown space is landed on, in addition to the normal end-of-turn mini-game. `[DECIDE, open question 3]` the alternative is that a Showdown replaces that turn's end-of-turn mini-game, which shortens matches (section 3).

### 2.6 Final stretch (the Last 5 Turns event)

- **Length:** the final stretch is the last `min(5, floor(TurnCount / 2))` turns: the last 5 turns of a 10, 15 or 20 turn match, the last 2 turns of a 5-turn quick match. It begins at the start of turn `TurnCount - FinalStretch + 1`.
- **Announcement:** the announcer shows current standings.
- **Doubled spaces:** Blue and Red spaces are worth ±6 for the rest of the match.
- **Catch-up:** the player in last place (fewest Llamas, then fewest Bars) receives 10 Bars. If several players are tied for last, each of them receives 10 Bars; if every active player is tied, nobody does. `[DECIDE]` keep this catch-up mechanic.

### 2.7 End of game

1. After the final mini-game, **Bonus Llamas** are awarded (1 Llama each, ties share, no award if nobody qualifies):
   - **Mini-game Llama:** most Bars earned from mini-games, Showdown pots included.
   - **Rich Llama:** highest Bar total held at any single moment during the match.
   - **Event Llama:** most Island Event spaces landed on.
   `[DECIDE]` bonus Llamas on or off by default (they add swinginess).
2. **Ranking:** most Llamas, then most Bars, then a dice roll between tied players.
3. **Results sequence:** each Bonus Llama is revealed one at a time (3 s each) with the recipient's piece highlighted; a final standings screen (10 s) shows every player ranked with Llamas and Bars and plays any tiebreak roll; the **Party Champion** is announced with a short celebration on the board; then the End Game device fires for the champion so Fortnite shows its victory screen to them and placements to everyone else, and the island ends. Fortnite returns players to its lobby. There is no rematch flow in v1.

### 2.8 Timeouts, leavers and edge cases

- **Every decision has a timeout and a default** (listed above) so one AFK player cannot stall the match. **AFK escalation:** after two consecutive timed-out decisions, a player's prompt timeouts drop to 3 s until they answer any prompt.
- **A player leaves mid-match:** their piece freezes, their later turns are skipped, they are removed from mini-games and standings. Remaining mini-game pools adapt to the new player count.
- **Leaving during your own turn:** the turn aborts at once (the turn routine is raced against that player's leave event). Open prompts are cancelled and do **not** fire their defaults; no purchase, steal, landing effect or colour is recorded for the leaver; the next player starts immediately.
- **Leaving as a target:** Gnome and Wheel of Fate targets are drawn from active players only. If the chosen target leaves before the effect resolves, the action is cancelled and any cost refunded.
- **Leaving inside a mini-game** ranks last for that game. If a side is left with no players (the solo in 1v3, a whole team in 2v2, either player in 1v1), the mini-game ends immediately and the other side wins. A Showdown leaver's ante stays in the pot.
- **Active player count** is checked after every turn and every mini-game. With 1 player remaining the match ends and they are champion. With 0 remaining the island ends immediately.
- **Negative Bars** are impossible. Any cost is clamped to what the player has, except purchases, which require the full price.
- **A Llama spot without the Llama** is just a passing node with no effect.
- **Ties** anywhere not covered above are shared (everyone tied receives the benefit) except final ranking, which is broken by a dice roll.

### 2.9 What non-active players see and do

Three quarters of a 4-player board phase is spent watching someone else's turn, so idle players need a show:

- **Camera:** everyone except the mover watches from the board overview camera. Each gameplay camera device attaches to the agent it is added to, so an orbit camera cannot follow another player's character; a true follow camera for spectators is M3 polish (section 6.1).
- **Stasis with expression:** idle players are held in stasis with turning and emotes allowed, so they can look around and emote. They cannot walk off their space.
- **Broadcast everything:** every dice result, junction choice, Llama purchase, Gnome action and landing effect is announced to all players with the mover's name, and the mover's decision countdown is shown on everyone's HUD.
- **Skip the dead air:** roulettes and results end early when everyone presses `Ready`.
- **Readable board:** each player gets a floating P1 to P4 marker in a fixed seat colour matching their HUD standings row, so four arbitrary skins can be told apart from the overview. Fortnite's minimap, health and inventory HUD are hidden during board phases.

---

## 3. Game loop

```
LOBBY ──► TURN ORDER ──► ┌──────────────── TURN t (1..N) ────────────────┐ ──► BONUS LLAMAS ──► RESULTS ──► END
                         │ if t = N - FinalStretch + 1: Final Stretch event │
                         │ for each active player in order:                 │
                         │     pre-roll ► roll ► move ► land                │
                         │     (a Showdown landing runs a battle now)       │
                         │ mini-game phase                                  │
                         │ check active player count                        │
                         └──────────────────────────────────────────────────┘
```

**Timing budget.** Two columns: players who answer promptly (3 s to roll, 4 s per prompt), and every timeout hit.

| Step | Prompt players | Every timeout hit |
|---|---|---|
| Lobby countdown and turn-order roll | 1.5 min | 2 min |
| Pre-roll menu | 3 s | 10 s |
| Dice animation | 2 s | 2 s |
| Movement (0.4 s per hop, average 5.5 hops) | 2.5 s | 2.5 s |
| Junction / purchase / Gnome prompt | 4 s | 10 s |
| Landing effect | 3 to 15 s | 3 to 15 s |
| **One player's turn** | **15 to 27 s** | **28 to 40 s** |
| Colour and selection roulettes | 4 s | 10 s |
| Briefing | 5 s | 15 s |
| Mini-game | 30 to 90 s | 30 to 90 s |
| Results and return | 7 s | 10 s |
| **One full turn (4 players)** | **≈ 2 to 4 min** | **≈ 3 to 4.5 min** |
| Showdown when landed on (ante roulette, briefing, game, results, two teleports) | 60 to 120 s each | 70 to 130 s each |
| Expected Showdowns per 10-turn 4-player match (2 of 45 spaces, 40 landings) | ≈ 1.8, so +2 to 4 min | +2 to 4 min |
| Bonus Llama reveal, standings, victory | 1 min | 1.5 min |
| **10-turn match** | **≈ 25 to 45 min** | **≈ 35 to 55 min** |

A 5-turn quick match lands near 15 to 25 minutes. If Showdowns replace the end-of-turn mini-game instead (open question 3), subtract the Showdown rows. `[DECIDE]` the default turn count against how long you expect Fortnite players to stay.

### 3.1 Loop in Verse (illustrative)

```verse
party_game_manager := class(creative_device):
    @editable Settings : party_settings = party_settings{}
    @editable StartSpace : board_space_device = board_space_device{}
    @editable LlamaSpots : []board_space_device = array{}
    @editable Minigames : []minigame_device = array{}

    var Players : []player_state = array{}

    OnBegin<override>()<suspends>:void =
        set Players = RunLobby()              # 2.0: countdown, roster freeze, stasis
        set Players = RollForTurnOrder(Players)
        FinalStretchStart := Settings.TurnCount - Settings.FinalStretch + 1
        for (Turn := 1..Settings.TurnCount):
            if (Turn = FinalStretchStart):
                FinalStretchEvent()
            for (P : Players, P.IsActive?):
                race:
                    TakeTurn(P, Turn)
                    AwaitLeave(P)             # 2.8: aborts the turn, cancels prompts
                if (not EnoughPlayers[]) { return EndEarly() }
            RunMinigamePhase()
            if (not EnoughPlayers[]) { return EndEarly() }
        AwardBonusLlamas()
        ShowResultsAndEnd()

    TakeTurn(P : player_state, Turn : int)<suspends>:void =
        FocusCamera(P)
        Choice := race:
            AwaitPreRollChoice(P)
            block:
                Sleep(PromptTimeout(P))       # AFK escalation lives here
                pre_roll_choice.Roll
        Steps := RollDice(P)
        MoveAlongBoard(P, Steps)     # countable vs passing kinds, junctions, Vault deposit
        ResolveLanding(P)            # may run a Showdown mini-game
        ReleaseCamera(P)
```

Everything above is `<suspends>` code driven by `Sleep`, `race`, `sync` and `event.Await()`. Player input arrives through Verse UI button callbacks that signal events the loop awaits.

---

## 4. Mini-game roster (v1)

Ten mini-games, each 30 to 90 seconds with a hard cut-off, each with a dedicated arena on the island. The four free-for-all games also serve as 2v2 games by scoring team totals, so 2v2 has six games and no extra arenas. All FFA games scale 2 to 4 players; all 1v3 games work as 1v2.

| # | Name (placeholder) | Type | Core devices | Rule |
|---|---|---|---|---|
| 1 | Shrinking Island | FFA / 2v2 | Prop Manipulators, Timer, Damage Volume | Platform tiles fall away. Last player (or last team with a survivor) standing wins; others rank by elimination time. Ends when one player or team remains, or at 60 s with survivors tied. |
| 2 | Llama Loot Rush | FFA / 2v2 | Collectible Objects, Timer | Most collectibles grabbed in 45 s (team total in 2v2). |
| 3 | Target Blitz | FFA / 2v2 | Target Gallery / damageable props, Item Granter, Timer | Most targets hit with a fixed weapon in 45 s (team total in 2v2). |
| 4 | Bus Dash | FFA / 2v2 | Race Checkpoints, Timer | First to the finish through a parkour course; others ranked by progress. Ends 10 s after the first finisher or at 75 s. Best finisher per team in 2v2. |
| 5 | King of the Pad | 2v2 | Capture Area, Timer, Team Settings | Team with most capture time in 60 s. |
| 6 | Ball Brawl | 2v2 | Ball Spawner, Triggers (goals), Timer | Most goals in 75 s. If tied, golden goal for at most 15 s, then a draw. |
| 7 | Hunter | 1v3 | Item Granter, Player Spawners, Timer | The solo player has 60 s and a weapon to eliminate the others, who are unarmed and fast. Eliminated players wait in the arena pen. |
| 8 | Keep Away | 1v3 | Trigger tag, Timer | The solo player holds a flag; the others try to tag them within 45 s. |
| 9 | Beacon Run | 1v3 | Triggers, Prop Manipulators, Timer | The three must light 3 beacons within 60 s while the solo extinguishes them (2 beacons as 1v2). |
| 10 | Bus Stop | 1v3 | Capture Item / Trigger, Timer | The solo carries a prop to a goal within 45 s while the others block and tag. |

Expected plays per 10-turn 4-player match from the section 5 board mix: about 2.2 free-for-all turns plus 1.8 Showdowns across games 1 to 4, about 2.9 2v2 turns across games 1 to 6, and about 4.9 1v3 turns across games 7 to 10. With 3 players about 6.5 of 10 turns are 1v2. **Duel** mini-games (1v1 stakes) are phase 2.

Every mini-game implements the same Verse interface so the manager does not know their internals:

```verse
minigame_result := struct:
    Ranking : [][]agent      # tie groups, best first
    WinningTeam : ?int

minigame_device := class<abstract>(creative_device):
    @editable Kind : minigame_kind = minigame_kind.FreeForAll
    @editable TimeLimit : float = 60.0
    Setup(Participants : [][]agent)<suspends> : void   # teams as arrays, FFA = one agent per array
    Play()<suspends> : minigame_result
    Teardown()<suspends> : void
```

---

## 5. Board design (v1: "Party Island")

- **45 countable spaces** on a figure-eight main loop with two junctions, so routing to the Llama matters: 24 Blue, 10 Red, 4 Island Event, 1 Wheel of Fate, 2 Storm Boss, 2 Showdown, 1 Vault, 1 Start. Plus **5 passing nodes**: 4 Llama spots and 1 Gnome. (The Gadget Shop is added in phase 2.)
- Island Events (board-specific, each triggered by a specific Event space). All movement here is forced movement per 2.2 step 6:
  1. **Storm Surge:** every other active player is pushed back 3 spaces along their own travelled path, not behind Start.
  2. **Supply Drop:** the lander gets 10 Bars, everyone else 3.
  3. **Battle Bus:** the lander is flown to the Start space (useful to loop back to a Llama). Start is not resolved; the lander keeps Green.
  4. **Rift:** the Llama relocates to a new random spot immediately.
- Visual language: each space is a circular pad whose colour and icon state its type. Billboard devices label special nodes. The player's own avatar is the game piece, with a floating P1 to P4 seat-colour marker above it.

---

## 6. UEFN and Verse architecture

### 6.1 Devices and Verse classes

| Piece | Implementation |
|---|---|
| `party_game_manager` | `creative_device`. Owns the state machine, player list, settings, UI, cameras, and the mini-game runner. One per island. |
| `board_space_device` | `creative_device` placed once per space or node. `@editable Kind : space_kind`, `@editable Next : []board_space_device` (more than one entry makes it a junction), `@editable EventId : int`. Position is `GetTransform().Translation`. `space_kind` separates **countable** kinds (Blue, Red, IslandEvent, WheelOfFate, StormBoss, Showdown, Vault, Start, Gadget) from **passing** kinds (LlamaSpot, Gnome, GadgetShop). `MoveAlongBoard` decrements the remaining steps only on countable kinds, fires passing events and the Vault deposit on move-through, and can therefore only end on a countable kind. |
| `player_state` | Verse class: agent, Bars, Llamas, current space, `Path` (spaces visited this match, for backwards forced movement), last landed colour, `IsActive`, consecutive timeouts, stats (mini-game Bars, max Bars, event landings). |
| `dice` | `GetRandomInt(1, 10)` plus a UI roulette animation. |
| Movement | `fort_character.TeleportTo[Position, Rotation]` hop by hop with `Sleep(0.4)`. During board phases players are held with `PutInStasis(stasis_args{AllowTurning := true, AllowFalling := false, AllowEmotes := true})` so they can look around and emote but not walk; released on entering a mini-game. |
| Cameras | Board overview: `gameplay_camera_fixed_point_device` with a fixed rotation, added to everyone between turns (paired with the gameplay controls device the docs require). Active mover: `gameplay_camera_orbit_device.AddTo(Mover)`. Gameplay camera devices attach to the agent they are added to and cannot track another player, so non-movers stay on the overview in v1. M3 polish option: a second fixed-point camera device the manager teleports to an offset behind the mover on each hop and adds to the non-movers, with a per-player `Overview / Follow` HUD toggle. |
| UI | Verse UI (`canvas`, `stack_box`, `text_block`, `button_loud`, `texture_block`) through `GetPlayerUI[Player].AddWidget`, in two layers. **(a) Persistent HUD** (Bars, Llamas, standings, turn banner, mover's countdown, mini-game timer) is added with `player_ui_slot{InputMode := ui_input_mode.None}` so it never captures input. **(b) Prompt widgets** (pre-roll, junction, Buy Llama, Gnome, Ready, roulettes, results) are added with `InputMode := ui_input_mode.All`, which shows a cursor and consumes the player's game input while visible; every prompt is removed with `RemoveWidget` before releasing stasis or teleporting into a mini-game. `hud_message_device` for broadcast announcer lines; a HUD controller device hides minimap, health and inventory during board phases. |
| Teams | Island Settings: Teams = 4 (not cooperative), so `GetPlayspace().GetTeamCollection().GetTeams()` returns four team handles. Verse cannot create teams at runtime; `AddToTeam[]` only moves agents between existing teams. Default and FFA: player i on team i. 2v2: `AddToTeam[Agent, Teams[0]]` / `Teams[1]`. 1v3: solo on `Teams[0]`, the rest on `Teams[1]`. `Teardown` moves everyone back to their own team before rewards are applied. Team Settings & Inventory devices per team for mini-game rules. Verify in M2 that the Team Size setting does not cap `AddToTeam`. |
| Mini-game runner | Teleports players to arena spawn points, calls `Setup` / `Play` / `Teardown`, applies rewards, returns players. Class Designer or Item Granters supply loadouts. Each arena has Player Spawners that the runner enables only during its mini-game, plus a spectator pen. The runner subscribes to `fort_character.EliminatedEvent` and the playspace spawn event for every participant: an eliminated player's time feeds the ranking and on respawn they are teleported into the pen and put in stasis until `Teardown`. Outside mini-games any unexpected respawn is teleported back to the player's board space and put back in stasis. |
| Audio and VFX | Audio Player devices for dice, Bars, win/lose stings; VFX Spawner devices on spaces; a seat-colour marker (billboard or VFX beam) above each player. |
| End of match | After the results sequence (2.7), `end_game_device` activated for the champion, which shows the victory screen and ends the island. |
| Debug | `@editable` toggles: fast mode (all sleeps ×0.1), forced dice value, forced mini-game, forced landing kind, skip to final stretch. |

### 6.2 Data model

```
player_state   { Agent, Bars, Llamas, Space, Path[], LastColour, IsActive, ConsecutiveTimeouts, Stats }
board_space    { Kind, Next[], EventId, Transform }
match_state    { Turn, TurnCount, FinalStretch, Order[], LlamaSpot, VaultBalance, FinalStretchActive, MinigamesPlayed[] }
```

All state lives on the server in the manager device. There is no persistence between matches in v1.

### 6.3 Constraints to design around

- **Island settings (to verify in the editor at M1):** Max Players 4; Join In Progress = Spectate; Teams = 4; Time Limit none (Verse ends the match); Game Start Countdown off or minimal (the Verse lobby runs the real countdown); Respawn on with a short respawn time and no spawn limit; no "last standing ends game" style end condition, because mini-games eliminate players and only `end_game_device` may end the island. For public matchmaking (open question 6): Minimum Players 2, Overtime Player Target 4, and queue durations tuned so matches start at 2 to 3 players rather than waiting for 4.
- **Verse runs server-side only.** All UI is driven from the server; input comes back as button events. Keep prompts few and large.
- **Controller and touch:** interactive Verse widgets require `ui_input_mode.All`, and players click with a cursor (mouse, stick-driven cursor on gamepad, tap on touch). There is no focus navigation. Board-phase prompts always run while the player is in stasis, so consumed movement input never matters. Every prompt needs a timeout default so unreachable UI never soft-locks the match.
- **Eliminations and respawns:** handled per the mini-game runner row in 6.1. The board itself has no enabled Player Spawner after the lobby.
- **Memory budget:** ten arenas plus a board must fit the island memory calculator. Target under 80% in v1 by sharing arena shells and prop galleries, and by keeping arenas small.
- **Teleport failures:** `TeleportTo[]` can fail if the destination is blocked. Every space needs clearance, and movement retries once with a small offset.
- **Match length:** target ≤ 55 minutes at 10 turns with every timeout hit (section 3).
- **Leavers:** `PlayerRemovedEvent` on the playspace marks the player inactive and aborts their running turn (2.8). Late joiners are not handled in Verse; the Spectate setting handles them.

---

## 7. Tuning sheet (all `@editable`)

| Parameter | Default |
|---|---|
| Turn count | 10 (options 5, 10, 15, 20) |
| Final stretch length | min(5, floor(turns / 2)) |
| Starting Bars | 10 |
| Dice range | 1 to 10 |
| Blue / Red value | +3 / -3 (±6 in the final stretch) |
| Llama price | 20 |
| Gnome Bar steal | cost 5, takes 5 to 15 |
| Gnome Llama steal | cost 50 |
| Vault deposit | 5 |
| Mini-game win | 10 Bars |
| Showdown antes | 5 / 10 / 20 / 30 / 50, pay all if short |
| Showdown split | 70 / 30, leftover Bar to a random 2nd-place player |
| Hidden block chance | 5% (80% Bars, 20% Llama) |
| Last place catch-up | 10 Bars at the start of the final stretch |
| Lobby countdown | 60 s, cut to 10 s on the 4th player |
| Turn-order roll window | 10 s |
| Pre-roll timeout | 10 s |
| Prompt timeout | 10 s |
| AFK escalation | 3 s timeouts after 2 consecutive timeouts |
| Hop duration | 0.4 s |
| Briefing timeout | 15 s |
| Early-end floor for roulettes and results | 2 s |

---

## 8. Milestones

| Milestone | Scope | Exit criterion |
|---|---|---|
| **M1 Playable loop** | Island settings, lobby flow and roster freeze, board graph device with countable and passing kinds, player state, turn order, dice, hop movement with stasis and cameras, Blue and Red spaces, two-layer HUD and prompts, one FFA mini-game with elimination handling, end-of-match ranking and results. | 2 to 4 players can finish a 5-turn match, and a player leaving in any phase does not stall it. |
| **M2 Full board rules** | Junctions, Llama purchase and relocation, Gnome, Vault, Island Events with forced movement, Wheel of Fate, Storm Boss, hidden blocks, colour roulette, 2v2 and 1v3 team formation, at least 2 mini-games per type (FFA, 2v2, 1v3). | Every rule in section 2 except Showdown, final stretch and bonus Llamas works; no colour split ever draws from an empty pool. |
| **M3 Match polish** | Showdown spaces, final stretch, bonus Llamas, AFK escalation, briefing and results sequence, idle-player broadcast and seat markers, optional follow camera, audio and VFX, debug tools, remaining mini-games, lobby settings vote. | Full rules. A 4-client 10-turn match finishes in ≤ 45 min with prompts answered within 3 s and ≤ 55 min with every timeout hit and one forced Showdown, fast mode off, with no soft-locks. |
| **M4 Phase 2** | Gadgets and Gadget Shop, Duel mini-games, second board, more mini-games, custom spectator UI. | As scoped later. |

**Testing:** UEFN Launch Session with 4 clients for every milestone. Fast mode and forced-dice toggles let one tester drive a full match in minutes. Each mini-game is tested standalone through a test harness device before joining the roster. Leaver tests disconnect a client in each phase: lobby, own turn, as a Gnome target, inside a mini-game, during a Showdown.

---

## 9. Out of scope for v1 and open questions

**Out of scope for v1:** Gadgets and the shop, Duel spaces, AI players, persistence or progression across matches, multiple boards, localisation beyond English, custom spectator UI for late joiners (they get Fortnite's default spectate), rematch flow.

**Open questions for you:**

1. Default turn count, and whether the host votes on it in the lobby. Section 3 gives the match lengths.
2. Keep hidden blocks, the last-place catch-up, and bonus Llamas on by default?
3. Showdown timing: run immediately when landed on (faithful; adds about 2 to 4 minutes to a 10-turn match) or replace that turn's end-of-turn mini-game (shorter, one fewer mini-game that turn)?
4. Board theme and the placeholder names above.
5. Do you want Gadgets (items) in v1 after all? They roughly double the rule surface.
6. Publishing goal: private island for friends (host `Start now` button, fixed roster) or public with matchmaking (queue settings, matches often start at 2 to 3 players and cannot refill).
