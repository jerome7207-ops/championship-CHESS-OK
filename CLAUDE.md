# CLAUDE.md

## Project Overview

**V2 Chess Championship** is a single-page browser chess game where the player fights through 10 AI opponents of increasing difficulty to claim a championship title. The entire application — HTML, CSS, and JavaScript — lives in a single file: `index.html`.

There is no build system, no bundler, no framework, and no dependencies beyond Google Fonts loaded via CDN. The game runs entirely client-side with localStorage for persistence and optional Supabase integration for online leaderboards.

## Repository Structure

```
championship-CHESS-OK/
├── index.html   # The entire application (HTML + CSS + JS, ~575 lines)
├── index        # Legacy file (just contains the repo name)
└── CLAUDE.md    # This file
```

## Architecture

Everything is in `index.html`. The file is organized into clearly commented sections:

### CSS (lines 8–115)
- CSS custom properties in `:root` for theming (dark theme with gold accents)
- Abbreviated class names throughout (e.g., `.scr` = screen, `.btn` = button, `.sq` = square, `.bd` = board)
- Fonts: `Orbitron` (headings), `Russo One` (labels), `Crimson Pro` (body)

### JavaScript Sections (lines 119–573)

| Section | Lines | Description |
|---|---|---|
| **Chess Engine** | 119–135 | Board representation, move generation, minimax with alpha-beta pruning, AI move selection |
| **Terminology** | 137–153 | Rank tiers, foresight/precision labels and display helpers |
| **Opponents** | 155–167 | Array of 10 opponent objects with personality, AI config, chat lines |
| **Learning Engine** | 169–175 | Tracks player patterns and adjusts AI difficulty after defeats |
| **Player Tag System** | 177–189 | Profile management, stats, beaten opponents, best move records |
| **Supabase Integration** | 191–212 | Optional online leaderboard via Supabase REST API |
| **State** | 214–216 | Global state object `S` and app root `$` |
| **Chat System** | 218–221 | In-game opponent chat with pattern-matched responses |
| **Render Functions** | 223–515 | Screen rendering (title, register, ladder, prematch, game, victory, defeat, champion, leaderboard, settings, records) |
| **Game Flow** | 517–573 | Game logic: move handling, AI turn, win/lose detection, progression |

### Key Global Variables

- **`S`** — Global state object holding current screen, board state, selection, move history, messages, stats
- **`$`** — Reference to `#app` DOM element (all rendering targets this)
- **`OPP`** — Array of 10 opponent configurations
- **`RANKS`** — Rank tier definitions (Pawn through Legend)
- **`SYM`** — Unicode chess piece symbol map
- **`VAL`** — Piece value table for evaluation
- **`INIT`** — Initial board layout (8x8 array)

### Chess Engine Details

- Board is an 8x8 2D array; `null` = empty, uppercase = white, lowercase = black
- `isW(p)` — checks if piece is white (uppercase)
- `gM(b,r,c,p)` — generates legal moves for a piece at position (r,c)
- `aM(b,pl)` — generates all moves for a player
- `ap(b,f,t)` — applies a move, returns new board (with auto-promotion to queen)
- `ev(b,w)` — evaluates board position using material + piece-square tables, weighted by opponent config `w`
- `mm(b,d,a,bt,mx,w)` — minimax with alpha-beta pruning
- `aiMv(b,depth,rng,w)` — selects AI move (with random move probability `rng`)

### Opponent System

Each opponent in `OPP[]` has:
- `depth` — minimax search depth (1–3)
- `rng` — random move probability (0.35 for easiest, 0 for hardest)
- `w` — evaluation weights: `{mat, cen, dev, ks, pc}` (material, center control, development, king safety, piece-specific multipliers)
- `chat` — response templates for various situations
- `scout` — scouting report text shown before the match

### Learning Engine

After the player beats an opponent:
- The engine records opening sequences, piece usage patterns, and attack zones
- On rematch, `getAdjusted(opp)` increases depth (up to +2) and reduces randomness
- The AI also counter-weights the player's most-used pieces and attack zones

### Data Persistence (localStorage keys)

| Key | Purpose |
|---|---|
| `cchv5` | Array of beaten opponent IDs |
| `cch-profile` | Player profile (name, country, state, tag) |
| `cch-stats` | Attempt counts per opponent, total moves |
| `cch-best-{id}` | Best move count record per opponent |
| `cch-learn-{id}` | Learning data per opponent (patterns, game count) |
| `cch-supa-key` | Supabase anon API key for leaderboard |

### Screens / Navigation

The app uses a state-machine pattern. `S.screen` drives which render function runs:

```
title → register → ladder → prematch → game → victory/defeat → champion
                                                   ↕
                                              leaderboard / settings / records
```

## Development Workflow

### Running Locally

Open `index.html` directly in any browser. No server required. For development with live reload, any static file server works:

```bash
python3 -m http.server 8000
# or
npx serve .
```

### Making Changes

Since everything is in one file:
1. Edit `index.html`
2. Refresh the browser
3. Game state persists in localStorage between reloads

### Testing

There is no automated test suite. Testing is manual:
- Play through opponents to verify chess logic
- Check localStorage values in browser DevTools
- Test responsive layout at various viewport sizes

### Code Conventions

- **Extreme minification style**: Variable and function names are very short (e.g., `gM` = get moves, `rLad` = render ladder, `mk` = make move)
- **Render functions** are named `r` + abbreviated screen name: `rTitle`, `rReg`, `rLad`, `rPM`, `rGame`, `rVic`, `rDef`, `rChamp`, `rLB`, `rRec`, `rSet`
- **CSS classes** are abbreviated: `.scr` (screen), `.fi` (fade-in), `.sq` (square), `.bd` (board), `.oc` (opponent card)
- **Template literals** with `innerHTML` for all rendering — no virtual DOM or framework
- **No semicolon-free style** — semicolons are used consistently
- **No modules/imports** — everything is in global scope

### Supabase Leaderboard Setup

The game optionally connects to a Supabase project for online rankings. Requires:
- A Supabase project with tables: `chess_leaderboard`, `chess_ghosts`
- The anon/public API key pasted into Settings

## Common Modification Tasks

### Adding a new opponent
Add an entry to the `OPP` array (index.html ~line 156) with `id`, `name`, `emoji`, AI config (`depth`, `rng`, `w`), `scout` text, and `chat` responses. Add a matching entry to `RANKS`. Update the total count references (currently hardcoded as 10 in multiple places).

### Adjusting AI difficulty
Modify the opponent's `depth` (search depth 1–4), `rng` (random move chance 0–1), and `w` weights in the `OPP` array.

### Changing the board appearance
Modify `.sl` (light square), `.sd` (dark square), `.ss` (selected), `.sv` (valid move) CSS classes near line 65.

### Modifying the chat system
Update the `chat` object on each opponent, and pattern-matching logic in `getResp()` at line 219.
