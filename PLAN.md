# Party Island: Design and Implementation Plan

**Status:** Draft v1 for review. Decisions that need your call are tagged `[DECIDE]`.
**Target:** A 2 to 4 player, turn-based board game with mini-games in the style of classic Mario Party (Mario Party 1 to 3 and Mario Party Superstars rules), built in UEFN with Verse.

> IP note: the *mechanics* of a board-plus-mini-game party game are fair to use. Nintendo's names, characters, music, art, and board layouts are not. Everything below uses Fortnite-flavoured placeholder names, and no Nintendo asset or text may ship. Names are placeholders; rename freely.

---

## 1. Glossary (Mario Party concept → Party Island name)

| Mario Party | Party Island (placeholder) | Notes |
|---|---|---|
| Star | **Llama** (Golden Llama) | The win condition currency. Bought for 20 Bars. |
| Coins | **Bars** (Gold Bars) | Earned on spaces and in mini-games. Floor at 0, no cap. |
| Dice Block | **Dice** | Random 1 to 10, shown as a UI roulette. |
| Blue Space | **Blue Space** | +3 Bars (+6 in the last 5 turns). |
| Red Space | **Red Space** | -3 Bars (-6 in the last 5 turns). |
| Happening / Event Space | **Island Event Space** | Board-specific scripted event. Counts toward the Event bonus. |
| Chance Time | **Wheel of Fate** | Forced swap or transfer between two players. |
| Bowser Space | **Storm Boss Space** | A random bad outcome. |
| Battle / VS Space | **Showdown Space** | Everyone antes Bars, then a battle mini-game for the pot. |
| Item Space / Items | **Gadget Space / Gadgets** | Phase 2 (not in v1). |
| Boo | **The Gnome** | Pay to steal Bars or a Llama from a rival. |
| Koopa Bank | **The Vault** | Deposit 5 Bars when passing, collect everything when landing. |
| Item Shop | **Gadget Shop** | Phase 2. |
| Mini-game | **Mini-game** | 4-player, 2v2, 1v3, Battle, Duel. |
| Bonus Stars | **Bonus Llamas** | Awarded at the end for mini-game Bars, max Bars, and event landings. |
| Superstar | **Party Champion** | Winner. |

---

## 2. Rules

### 2.1 Players and match setup

- **Players:** 2 to 4 humans on one island instance. No AI players in v1 (UEFN has no scriptable board AI; see section 9 for a later option). Island max players is 4, minimum 2, join-in-progress off.
- **Turn count:** 10 turns by default. Options 5 (quick), 10, 15, 20. `[DECIDE]` whether the host picks in a lobby vote (M3) or the creator fixes it in an editable setting (v1).
- **Starting Bars:** 10 each. **Starting Llamas:** 0.
- **Turn order:** every player hits the Dice once in the lobby. Highest goes first; ties re-roll among the tied players only. Order is fixed for the whole match.
- **Board:** one board in v1 ("Party Island"), roughly 50 spaces, a main loop with 2 branch junctions, 4 Llama spots, 1 Gnome, 1 Vault, 3 to 4 Island Event spaces, 1 to 2 Storm Boss spaces, 1 Wheel of Fate, 1 to 2 Showdown spaces.
- **Llama placement:** one Llama is on the board at a time, at one of the 4 Llama spots. When it is bought, it moves to a different random spot.

### 2.2 A player's turn

1. **Turn start.** Camera focuses on the player. Banner "Player N's turn". Standings visible on the HUD.
2. **Pre-roll menu.** `Roll` or `View Board`. (Phase 2 adds `Use Gadget`.) Timeout 15 s, default `Roll`.
3. **Roll.** Dice gives 1 to 10. The result is shown to everyone.
4. **Move.** The player hops one space at a time along the board.
   - **Junctions:** the mover chooses a direction from a UI prompt. Timeout 10 s, default is the first listed direction (the main loop).
   - **Passing spaces** (Llama spot, Gnome, Vault, Gadget Shop) do **not** count as a step. Their events fire as the player passes or stops on them.
   - Movement cannot be undone. Rolls are never re-rolled except for ties in turn order and the final tiebreak.
5. **Land.** The space the player stops on resolves immediately (section 2.3). The space's colour is remembered for mini-game selection.
6. **Turn end.** Camera returns to the board overview, next player.

### 2.3 Space effects (landing)

| Space | Colour for mini-game selection | Effect when landed on |
|---|---|---|
| Blue | Blue | +3 Bars (+6 during last 5 turns). Hidden-block check. |
| Red | Red | -3 Bars (-6 during last 5 turns). Hidden-block check. |
| Island Event | Green (roulette) | Board-specific event (section 7). Adds 1 to the player's Event count. |
| Wheel of Fate | Green (roulette) | Roulette picks Player A, Player B and an exchange: swap Bars, swap Llamas, A gives B 10/20/30 Bars, A gives B 1 Llama. The landing player is always one of the two. |
| Storm Boss | Green (roulette) | Roulette picks one: lose 20 Bars; lose 1 Llama (or 20 Bars if none); everyone loses 10 Bars; **Bar Shuffle** (all players' Bars pooled and split evenly, remainder to the lander). |
| Showdown | Green (roulette) | Roulette picks the ante: 5, 10, 20, 30 or 50 Bars (capped at the poorest player's Bars). All players pay in. A battle mini-game runs **immediately**. 1st takes 70% of the pot, 2nd takes 30%, odd Bars to 1st. |
| Gadget (phase 2) | Green (roulette) | Receive a random Gadget. |
| Start | Blue | Treated as Blue when landed on. |

**Hidden blocks.** Landing on a Blue or Red space has a 5% chance to spawn a hidden block: 80% of blocks hold 20 Bars, 20% hold a Llama. `[DECIDE]` keep or remove for competitive fairness.

### 2.4 Passing events

| Spot | On passing (or stopping) |
|---|---|
| **Llama spot (with the Llama present)** | If the player has 20 or more Bars: prompt `Buy Llama for 20 Bars?` (timeout 10 s, default Yes). On purchase the Llama relocates to a different spot. If it cannot be afforded, a "not enough Bars" message plays and movement continues. |
| **The Gnome** | Prompt: `Steal Bars (cost 5)`, `Steal a Llama (cost 50)`, `Pass`. Stealing Bars takes a random 5 to 15 Bars (or all if fewer) from a chosen rival. Stealing a Llama takes 1 Llama from a chosen rival who has one. Options the player cannot afford, or with no valid target, are hidden. Timeout 10 s, default `Pass`. |
| **The Vault** | Passing: deposit 5 Bars (or all if fewer) into the Vault. Stopping on the Vault space: collect the entire Vault balance. The Vault is a landable space (Blue colour for mini-game selection). |
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

3. **Selection roulette.** A random mini-game of that type, excluding games already played this match until the pool is exhausted.
4. **Briefing.** Rules screen with controls. Players press `Ready`; the game starts when all are ready or after 20 s.
5. **Play.** Players are teleported into the mini-game arena with the game's loadout. Hard time limit per game (30 to 90 s).
6. **Rewards.** Winner(s) receive 10 Bars each. FFA ties: all tied players win. 2v2: both teammates win. 1v3: the solo player wins 10, or each of the three wins 10. No reward on a full draw.
7. **Return.** Results screen 5 s, then everyone is teleported back to their board space.

Showdown (battle) mini-games use the FFA pool with the pot rules from 2.3 and are run at the moment the Showdown space is landed on, in addition to the normal end-of-turn mini-game.

### 2.6 Last 5 turns

At the start of the turn with 5 turns remaining:

- Announcer shows current standings.
- Blue and Red spaces are worth ±6 for the rest of the match.
- The player in last place receives 10 Bars. `[DECIDE]` keep this catch-up mechanic.

### 2.7 End of game

1. After the final mini-game, **Bonus Llamas** are awarded (1 Llama each, ties share, no award if nobody qualifies):
   - **Mini-game Llama:** most Bars earned from mini-games.
   - **Rich Llama:** highest Bar total held at any single moment during the match.
   - **Event Llama:** most Island Event spaces landed on.
   `[DECIDE]` bonus Llamas on or off by default (they add swinginess).
2. **Ranking:** most Llamas, then most Bars, then a dice roll between tied players.
3. The **Party Champion** is announced, the Fortnite victory screen fires for the winner, and the island ends.

### 2.8 Timeouts, leavers and edge cases

- **Every decision has a timeout and a default** (listed above) so one AFK player cannot stall the match.
- **A player leaves mid-match:** their piece freezes, their turns are skipped, they are removed from mini-games and standings. Remaining mini-game pools adapt to the new player count. If 1 player remains, the match ends and they win.
- **Negative Bars** are impossible. Any cost is clamped to what the player has, except purchases, which require the full price.
- **A Llama spot without the Llama** is just a passing node with no effect.
- **No join-in-progress.** Late joiners spectate from the board camera.
- **Mini-game disconnect** counts as last place for that player.

---

## 3. Game loop

```
LOBBY ──► TURN ORDER ──► ┌───────────── TURN t (1..N) ─────────────┐ ──► BONUS LLAMAS ──► RESULTS ──► END
                         │ if t = N-4: Last 5 Turns event          │
                         │ for each active player in order:        │
                         │     pre-roll ► roll ► move ► land       │
                         │ mini-game phase                         │
                         └─────────────────────────────────────────┘
```

**Per-phase timing budget** (used to size the match):

| Step | Budget |
|---|---|
| Pre-roll menu | ≤ 15 s (default auto-roll) |
| Dice animation | 2 s |
| Movement | 0.4 s per hop, average 5.5 hops ≈ 2.5 s |
| Junction / purchase / Gnome prompts | ≤ 10 s each |
| Landing effect | 3 to 15 s |
| **One player's turn** | **≈ 30 to 45 s** |
| Colour and selection roulettes | 10 s |
| Briefing | ≤ 20 s |
| Mini-game | 30 to 90 s |
| Results and return | 10 s |
| **One full turn (4 players)** | **≈ 4.5 to 5.5 min** |
| **10-turn match** | **≈ 45 to 55 min** |

A 5-turn quick match lands near 25 minutes. `[DECIDE]` the default turn count against how long you expect Fortnite players to stay.

### 3.1 Loop in Verse (illustrative)

```verse
party_game_manager := class(creative_device):
    @editable Settings : party_settings = party_settings{}
    @editable StartSpace : board_space_device = board_space_device{}
    @editable LlamaSpots : []board_space_device = array{}
    @editable Minigames : []minigame_device = array{}

    var Players : []player_state = array{}

    OnBegin<override>()<suspends>:void =
        set Players = WaitForLobby()
        set Players = RollForTurnOrder(Players)
        for (Turn := 1..Settings.TurnCount):
            if (Turn = Settings.TurnCount - 4):
                LastFiveTurnsEvent()
            for (P : Players, P.IsActive?):
                TakeTurn(P, Turn)
            RunMinigamePhase()
        AwardBonusLlamas()
        ShowResultsAndEnd()

    TakeTurn(P : player_state, Turn : int)<suspends>:void =
        FocusCamera(P)
        Choice := race:
            AwaitPreRollChoice(P)
            block:
                Sleep(Settings.PreRollTimeout)
                pre_roll_choice.Roll
        Steps := RollDice(P)
        MoveAlongBoard(P, Steps)     # handles junctions and passing events
        ResolveLanding(P)            # may run a Showdown mini-game
        ReleaseCamera(P)
```

Everything above is `<suspends>` code driven by `Sleep`, `race`, `sync` and `event.Await()`. Player input arrives through Verse UI button callbacks that signal events the loop awaits.

---

## 4. Mini-game roster (v1)

Eight mini-games, each 30 to 90 seconds, each with a dedicated arena on the island. All FFA games scale 2 to 4 players.

| # | Name (placeholder) | Type | Core devices | Rule |
|---|---|---|---|---|
| 1 | Shrinking Island | FFA | Prop Manipulators, Timer, Damage Volume | Platform tiles fall away over 60 s. Last player standing wins; others rank by elimination time. |
| 2 | Llama Loot Rush | FFA | Collectible Objects, Timer | Most collectibles grabbed in 45 s. |
| 3 | Target Blitz | FFA | Target Gallery / Damageable props, Item Granter, Timer | Most targets hit with a fixed weapon in 45 s. |
| 4 | Bus Dash | FFA | Race Checkpoints, Timer | First to the finish through a parkour course; others ranked by progress. |
| 5 | King of the Pad | 2v2 | Capture Area, Timer, Team Settings | Team with most capture time in 60 s. |
| 6 | Ball Brawl | 2v2 | Ball Spawner, Triggers (goals), Timer | Most goals in 90 s, golden goal if tied at 0. |
| 7 | Hunter | 1v3 | Item Granter, Player Spawners, Timer | The solo player has 60 s and a weapon to eliminate the other three, who are unarmed and fast. |
| 8 | Keep Away | 1v3 | Collectible / Trigger tag, Timer | The solo player holds a flag; the three try to tag them within 45 s. |

Showdown (battle) games draw from the FFA pool. **Duel** mini-games (1v1 stakes) are phase 2.

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

- About 50 countable spaces on a figure-eight main loop with two junctions, so routing to the Llama matters.
- Space mix target: 24 Blue, 10 Red, 4 Island Event, 1 Wheel of Fate, 2 Storm Boss, 2 Showdown, 1 Vault (landable), plus 4 Llama spots, 1 Gnome and the Start space as passing or special nodes.
- Island Events (board-specific, each triggered by a specific Event space):
  1. **Storm Surge:** every other player within the storm zone is pushed back 3 spaces (not past Start).
  2. **Supply Drop:** the lander gets 10 Bars, everyone else 3.
  3. **Battle Bus:** the lander is flown to the Start space (useful to loop back to a Llama).
  4. **Rift:** the Llama relocates to a new random spot immediately.
- Visual language: each space is a circular pad whose colour and icon state its type. Billboard devices label special spots. The player's own avatar is the game piece.

---

## 6. UEFN and Verse architecture

### 6.1 Devices and Verse classes

| Piece | Implementation |
|---|---|
| `party_game_manager` | `creative_device`. Owns the state machine, player list, settings, UI, cameras, and the mini-game runner. One per island. |
| `board_space_device` | `creative_device` placed once per space. `@editable Kind : space_kind`, `@editable Next : []board_space_device` (more than one entry makes it a junction), `@editable EventId : int`. Position is `GetTransform().Translation`. The board is a directed graph of these. |
| `player_state` | Verse class: agent, Bars, Llamas, current space, last landed colour, `IsActive`, stats (mini-game Bars, max Bars, event landings). |
| `dice` | `GetRandomInt(1, 10)` plus a UI roulette animation. |
| Movement | `fort_character.TeleportTo[Position, Rotation]` hop by hop with `Sleep(0.4)`. Players are held with `PutInStasis` during board phases so they cannot walk off their space; released in mini-games. |
| Cameras | `gameplay_camera_fixed_point_device` for the board overview, `gameplay_camera_orbit_device` following the active mover, `AddTo(Agent)` / `RemoveFrom(Agent)` as phases change. |
| UI | Verse UI (`canvas`, `stack_box`, `text_block`, `button_loud`, `texture_block`) through `GetPlayerUI[Player].AddWidget`. Per-player HUD (Bars, Llamas, standings), dice, prompts, briefing, results. `hud_message_device` for broadcast announcer lines. |
| Teams | `GetPlayspace().GetTeamCollection().AddToTeam[Agent, Team]` to form 2v2 and 1v3 sides before a mini-game; Team Settings & Inventory devices per team for mini-game rules. |
| Mini-game runner | Teleports players to arena spawn points, calls `Setup` / `Play` / `Teardown`, applies rewards, returns players. Class Designer or Item Granters supply loadouts. |
| Audio and VFX | Audio Player devices for dice, coins, win/lose stings; VFX Spawner devices on spaces. |
| End of match | `end_game_device` activated for the champion (victory screen), or custom results plus island end. |
| Debug | `@editable` toggles: fast mode (all sleeps ×0.1), forced dice value, forced mini-game, skip to last 5 turns. |

### 6.2 Data model

```
player_state   { Agent, Bars, Llamas, Space, LastColour, IsActive, Stats }
board_space    { Kind, Next[], EventId, Transform }
match_state    { Turn, TurnCount, Order[], LlamaSpot, VaultBalance, LastFiveActive, MinigamesPlayed[] }
```

All state lives on the server in the manager device. There is no persistence between matches in v1.

### 6.3 Constraints to design around

- **Verse runs server-side only.** All UI is driven from the server; input comes back as button events. Keep prompts few and large.
- **Controller and touch:** Verse UI buttons must be reachable by gamepad focus and touch. Every prompt needs a timeout default so unreachable UI never soft-locks the match.
- **Memory budget:** 8 arenas plus a board must fit the island memory calculator. Target under 80% in v1 by sharing arena shells and prop galleries.
- **Teleport failures:** `TeleportTo[]` can fail if the destination is blocked. Every space needs clearance, and movement retries once with a small offset.
- **Match length vs island time limit:** set the island time limit off and let Verse end the match. Target ≤ 55 minutes at 10 turns.
- **Late joiners and leavers:** handled in Verse via `PlayerAddedEvent` / `PlayerRemovedEvent` on the playspace.

---

## 7. Tuning sheet (all `@editable`)

| Parameter | Default |
|---|---|
| Turn count | 10 |
| Starting Bars | 10 |
| Dice range | 1 to 10 |
| Blue / Red value | +3 / -3 (±6 in last 5 turns) |
| Llama price | 20 |
| Gnome Bar steal | cost 5, takes 5 to 15 |
| Gnome Llama steal | cost 50 |
| Vault deposit | 5 |
| Mini-game win | 10 Bars |
| Showdown antes | 5 / 10 / 20 / 30 / 50 |
| Showdown split | 70 / 30 |
| Hidden block chance | 5% (80% Bars, 20% Llama) |
| Last place catch-up | 10 Bars at last 5 turns |
| Pre-roll timeout | 15 s |
| Prompt timeout | 10 s |
| Hop duration | 0.4 s |
| Briefing timeout | 20 s |

---

## 8. Milestones

| Milestone | Scope | Exit criterion |
|---|---|---|
| **M1 Playable loop** | Board graph device, player state, lobby and turn order, dice, hop movement with stasis and cameras, Blue and Red spaces, HUD, one FFA mini-game, end-of-match ranking. | 2 to 4 players can finish a 5-turn match. |
| **M2 Full board rules** | Junctions, Llama purchase and relocation, Gnome, Vault, Island Events, Wheel of Fate, Storm Boss, hidden blocks, colour roulette, 2v2 and 1v3 formation, 6 of 8 mini-games. | Every rule in section 2 except Showdown, last 5 turns and bonus Llamas works. |
| **M3 Match polish** | Showdown spaces, last 5 turns, bonus Llamas, leaver and AFK handling, briefing and results screens, audio and VFX, debug tools, remaining mini-games, lobby settings vote. | Full rules, 10-turn match under 55 minutes, no soft-locks in a 4-client test. |
| **M4 Phase 2** | Gadgets and Gadget Shop, Duel mini-games, second board, more mini-games. | As scoped later. |

**Testing:** UEFN Launch Session with 4 clients for every milestone. Fast mode and forced-dice toggles let one tester drive a full match in minutes. Each mini-game is tested standalone through a test harness device before joining the roster.

---

## 9. Out of scope for v1 and open questions

**Out of scope for v1:** Gadgets and the shop, Duel spaces, AI players, persistence or progression across matches, multiple boards, localisation beyond English, spectator mode for non-players.

**Open questions for you:**

1. Default turn count and whether the host votes on it in the lobby.
2. Keep hidden blocks, the last-place catch-up, and bonus Llamas on by default?
3. Should Showdown mini-games run immediately (faithful) or replace that turn's end-of-turn mini-game (shorter matches)?
4. Board theme and the placeholder names above.
5. Do you want Gadgets (items) in v1 after all? They roughly double the rule surface.
6. Publishing goal: private island for friends, or public with matchmaking (affects lobby and leaver handling priorities).
