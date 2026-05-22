# Logic Puzzle Game

Logic Puzzle Game is a static web game for language practice through logic grid puzzles.
Players read clues in the target language, mark matches and exclusions in a grid,
and solve each puzzle by deduction.

The current app is a Next.js static export. Puzzle content comes from JSON files in
the repo; there is no database, login, API server, or remote progress sync.

## What The Game Does Today

- Shows one interactive logic grid puzzle at a time.
- Supports language, level, category, and puzzle navigation from the puzzle screen.
- Lets players cycle cells through empty, NO, and YES.
- Auto-fills eliminated NO cells after a YES match.
- Starts a timer on the first grid interaction.
- Checks the answer automatically once the expected number of YES cells is filled.
- Shows an incorrect modal with the number of mistakes.
- Shows a completion modal, wave animation, confetti, and a next-puzzle action.
- Saves completed puzzle IDs in `localStorage`.
- Includes a two-part first-run tutorial, also tracked in `localStorage`.
- Reads clues aloud with the browser Web Speech API.
- Shows optional grammar notes when a puzzle provides one.

## Stack

- Next.js 14 App Router
- React 18
- TypeScript
- Tailwind CSS
- Static export via `next.config.mjs`
- JSON puzzle files loaded at build time from `content/puzzle/`

## Getting Started

```bash
pnpm install
pnpm dev
```

Open [http://localhost:4321](http://localhost:4321).

Useful commands:

```bash
pnpm dev      # run the local dev server on port 4321
pnpm build    # create the static export
pnpm lint     # run Next.js linting
python3 scripts/validate_puzzles.py
```

## Routes

| Route | Behavior |
|---|---|
| `/` | Redirects to the first available English puzzle, preferring `PRE` then `A1`. |
| `/<lang>` | Redirects to the first puzzle for the first available level in that language. |
| `/<lang>/<level>` | Redirects to the first puzzle for that language and level. |
| `/<lang>/puzzle/<id>` | Renders the puzzle player. |

Examples:

- `/en/puzzle/en-pre-animals-and-colors-1`
- `/pt/puzzle/pt-a1-casa-e-rotina-1`

Puzzle IDs are generated deterministically from `languageCode`, `levelCode`, and
`title`, so changing a title changes the URL.

## Current Content

Puzzle files live in `content/puzzle/`. As of this README, the repo contains 94
puzzles:

| Language | Levels |
|---|---|
| English (`en`) | `PRE`, `A1`, `A2`, `B1` |
| Portuguese (`pt`) | `PRE`, `A1`, `A2` |

The UI translation layer currently has strings for English and Portuguese. The
audio hook maps a few additional language codes for Web Speech, but only
languages with JSON content appear in the app.

## Puzzle Format

Each JSON file contains a `puzzles` array:

```json
{
  "puzzles": [
    {
      "title": "Rotina Diária (1)",
      "themeSlug": "Rotina-Diaria",
      "levelCode": "A1",
      "languageCode": "pt",
      "gridSize": 3,
      "narrativeIntro": null,
      "grammarFocus": ["present tense"],
      "culturalNote": null,
      "categories": [
        { "label": "Pessoas", "emoji": "👥", "order": 0 },
        { "label": "Atividades", "emoji": "🏃", "order": 1 },
        { "label": "Horários", "emoji": "🕒", "order": 2 }
      ],
      "items": [
        { "categoryIndex": 0, "label": "Ana", "emoji": "👩" },
        { "categoryIndex": 1, "label": "correr", "emoji": "🏃" },
        { "categoryIndex": 2, "label": "sete horas", "emoji": "🕖" }
      ],
      "clues": [
        { "text": "Ana corre às sete horas.", "clueType": "positive" }
      ],
      "solution": [
        { "itemLabels": ["Ana", "correr", "sete horas"], "value": "YES" }
      ],
      "grammarNote": "Use \"às sete horas\" to connect an action with a time."
    }
  ]
}
```

### Solution Formats

`PRE` puzzles use two categories and explicit cell values:

```json
{ "rowItemLabel": "cat", "colItemLabel": "red", "value": "YES" }
{ "rowItemLabel": "cat", "colItemLabel": "blue", "value": "NO" }
```

`A1`, `A2`, and `B1` puzzles use one YES row per solved set. The app infers the
NO cells:

```json
{ "itemLabels": ["Ana", "correr", "sete horas"], "value": "YES" }
```

Supported clue types in the existing content are:

- `positive`
- `negative`
- `relational`

## Adding Or Editing Puzzles

1. Add or edit a JSON file in `content/puzzle/`.
2. Keep `languageCode`, `levelCode`, `themeSlug`, `title`, `gridSize`,
   categories, items, clues, and solution internally consistent.
3. Run the validator:

```bash
python3 scripts/validate_puzzles.py
python3 scripts/validate_puzzles.py content/puzzle/pt-a2.json
```

The validator checks structural consistency, solution references, some clue
contradictions, numeric ordering clues, and category-pair coverage. Some checks
depend on English clue parsing or optional config that is not currently present
in the repo, so do not treat the validator as a complete proof of puzzle
uniqueness.

For fuller authoring rules, see [content/AUTHORING.md](content/AUTHORING.md).

## Pre-Commit Hook

The repo includes a hook script that validates puzzle JSON when committing
content changes:

```bash
cp scripts/pre-commit-hook.sh .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

## Project Structure

```text
app/
  page.tsx                    # root redirect
  [lang]/
    layout.tsx                # syncs the html lang attribute
    page.tsx                  # language redirect
    [level]/page.tsx          # level redirect
    puzzle/[id]/page.tsx      # puzzle player route
components/
  PuzzlePlayer.tsx            # page shell, filters, timer, modals, progress
  PuzzleGrid.tsx              # grid interaction and auto-fill behavior
  TutorialOverlay.tsx         # two-part onboarding tutorial
  ClueList.tsx                # clue display and speech playback
  LangSync.tsx                # client-side html lang updater
content/
  puzzle/                     # source JSON puzzle files
  AUTHORING.md                # puzzle writing guidelines
hooks/
  useAudio.ts                 # Web Speech API wrapper
lib/
  puzzles-static.ts           # loads and indexes puzzle JSON at build time
  translations.ts             # UI strings for en/pt
scripts/
  validate_puzzles.py         # content validator
  pre-commit-hook.sh          # optional git hook
```

## Known Limits

- Progress is stored only in the browser's `localStorage`.
- There is no user account, backend, database, dashboard, or teacher workflow.
- Web Speech API voice availability depends on the user's browser and OS.
- `narrativeIntro`, `grammarFocus`, and `culturalNote` exist in the content shape
  but are not rendered by the current puzzle player.
- The root route defaults to English content.
