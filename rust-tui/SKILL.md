---
name: rust-tui
description: Use when adding screens, changing layout, editing keybindings, styling, or any UI work in tools/port or tools/shingle. Covers brand palette, layout patterns, focus behavior, modals, async loading, new-screen checklist, and code review.
---

# ECIC Ratatui TUI Design System

Applies to both `tools/port/` (Port CIO TUI) and `tools/shingle/` (Shingle brand surface TUI).

---

## Brand Palette — Rust Constants

ECIC brand tokens map from `packages/design-tokens/src/tokens.css`. Both tools need a `src/palette.rs` that exports these constants. **Every rendering file imports from `palette` — never scatter inline `Color::Rgb(...)` literals.**

**Current reality:** Both tools have `palette.rs` but most render files don't import from it yet. Port has 186 inline `Color::Rgb(...)` calls across 15 files (per the 2026-04-19 design audit — see `docs/archive/2026-10-14/reviews/2026-04-19-port-design-audit.md`). Shingle uses ANSI named colors (`Color::Yellow`, `Color::Cyan`, etc.). When you touch either tool, migrate inline colors to `palette` imports as you go.

```rust
// src/palette.rs — create in both tools/port/ and tools/shingle/

use ratatui::style::Color;

// ── Brand primaries (from packages/design-tokens/src/tokens.css) ──
pub const TEAL:            Color = Color::Rgb(20, 184, 166);   // --color-teal (#14b8a6)
pub const TEAL_LIGHT:      Color = Color::Rgb(94, 234, 212);   // --color-teal-text-light (#5eead4)
pub const TEAL_DARK:       Color = Color::Rgb(15, 118, 110);   // --color-teal-dark (#0f766e)
pub const PURPLE:          Color = Color::Rgb(88, 28, 135);    // --color-purple (#581c87)
pub const PURPLE_BORDER:   Color = Color::Rgb(76, 21, 112);    // --color-purple-dark (#4c1570) — panel borders
pub const PURPLE_LIGHT:    Color = Color::Rgb(107, 70, 193);   // --color-purple-light (#6b46c1)
pub const AMBER:           Color = Color::Rgb(245, 158, 11);   // --color-amber (#f59e0b)
pub const DANGER:          Color = Color::Rgb(239, 68, 68);    // --color-danger (#ef4444)

// ── Terminal-only colors (no CSS token — dark bg needs deeper purples) ──
pub const PURPLE_DEEP:     Color = Color::Rgb(46, 16, 101);    // active tab bg, context line bg
pub const VIOLET:          Color = Color::Rgb(167, 139, 250);  // inactive tab labels, context text
pub const MODAL_BG:        Color = Color::Rgb(30, 10, 60);     // near-black purple — modal backgrounds

// ── Strategy semantic colors (Port only, but defined here for consistency) ──
pub const STRATEGY_GROWTH: Color = TEAL;                       // growth strategy = primary brand
pub const STRATEGY_INCOME: Color = Color::Rgb(100, 180, 255);  // income strategy — soft blue
pub const STRATEGY_DIVERS: Color = Color::Rgb(255, 165, 0);    // diversification — orange

// ── Semantic UI colors ──
pub const HINT:            Color = Color::DarkGray;            // footer keyhints, muted text
pub const BORDER_ACTIVE:   Color = TEAL;                       // focused pane border
pub const BORDER_IDLE:     Color = Color::DarkGray;            // unfocused pane border
pub const GAIN:            Color = Color::Green;               // positive / passing
pub const LOSS:            Color = Color::Red;                 // negative / failing / excluded
```

**Tick score color helper (Port)** — already exists in `tools/port/src/ui.rs:tick_color()`. Don't duplicate it; move it to `palette.rs` when creating the module.

**Shingle migration:** Shingle uses `Color::Yellow` for focus and ANSI named colors everywhere. These render differently in every terminal theme — a light theme makes Yellow invisible. When touching any Shingle file, replace ANSI focus colors with `BORDER_ACTIVE` / `TEAL` from palette.

**Why terminal-only colors?** The CSS design tokens target white/light backgrounds. Terminal TUIs use dark backgrounds that need deeper, more saturated purples for contrast. `PURPLE_DEEP` and `MODAL_BG` are intentionally darker than anything in the web palette.

---

## Current State & Migration Debt

This skill describes the target state. Here's what's true *right now* and what
needs migrating — **re-verified against the tree 2026-08-13**; Port's palette
migration had completed without this table being updated:

| Gap | Port | Shingle | Fix |
|-----|------|---------|-----|
| `palette.rs` exists | **Done** — imported by 21 files | Yes — created, not yet imported by any file | Shingle only: import `use crate::palette::*` and replace inline colors on contact |
| 4-row chrome | Yes | No — 2-row (tab bar + body) | Add context line + footer when touching Shingle |
| Tab bar style | Correct (custom Paragraph) | Wrong (ratatui `Tabs`, Yellow highlight) | Replace with Port-style tab bar |
| Focus border color | Correct (TEAL) | Wrong (`Color::Yellow`) | Replace with `BORDER_ACTIVE` |
| Modal system | Full (`modal.rs`, 6 variants) | None | Copy `modal.rs` from Port when needed |
| `?` help overlay | Yes | No | Add per-screen help when adding modals |
| Footer keyhints | Yes | No | Add when upgrading to 4-row chrome |

**Migration rule:** Don't do a dedicated refactor PR. Fix these gaps incrementally when you're already touching a file for a feature or bugfix.

---

## Chrome Layout Pattern

Both tools use a 4-row vertical chrome. New screens must fit inside row 3 (body).

```rust
// tools/port/src/ui.rs — draw()
let chunks = Layout::default()
    .direction(Direction::Vertical)
    .constraints([
        Constraint::Length(1),  // [0] Tab bar    — "Ethical Capital  1:Triage  2:Portfolio …"
        Constraint::Length(1),  // [1] Context    — screen-specific breadcrumb / filter state
        Constraint::Min(0),     // [2] Body       — screen content
        Constraint::Length(1),  // [3] Footer     — contextual keyhints
    ])
    .split(f.area());
```

**Modal overlay** renders after `draw_body()` using `Clear` + centered rect. Port's `modal::draw_modal()` handles this — use it, don't reinvent.

Shingle currently has a 2-row chrome (tab bar + body). When adding complexity, migrate to the 4-row pattern.

---

## Tab Bar

```rust
// Tab bar background is always PURPLE
// Active screen: fg=TEAL, bg=PURPLE_DARK, BOLD
// Inactive screens: fg=VIOLET
// Label format: " N:ScreenName " (1-indexed number prefix)
```

Port's `draw_tab_bar()` in `src/ui.rs` is canonical. Shingle's `Tabs` widget should be migrated to match this style when touched.

---

## Focus and Border Pattern

Every focusable pane **must** change its border color when focused. This is non-negotiable — it's the terminal's only cursor.

```rust
fn border_style(focused: bool) -> Style {
    if focused {
        Style::default().fg(TEAL)
    } else {
        Style::default().fg(Color::DarkGray)
    }
}

// Usage:
Block::default()
    .borders(Borders::ALL)
    .border_style(border_style(app.left_pane_focused))
    .title("Feed")
```

**Tab** cycles focus between panes within a screen. **Number keys** (Port) or **Left/Right** (Shingle) switch screens.

---

## Footer Key Hints

The footer row shows 4–6 contextual hints for the active screen/pane. Update them on every focus change.

```rust
fn draw_footer(f: &mut Frame, area: Rect, app: &App) {
    let hints = match app.current_screen {
        Screen::Portfolio => "j/k:navigate  e:edit  f:filter  /:search  ?:help",
        Screen::Exclusions => "j/k:navigate  Enter:detail  e:edit  d:delete  ?:help",
        // …
    };
    let hint_widget = Paragraph::new(hints)
        .style(Style::default().fg(HINT));
    f.render_widget(hint_widget, area);
}
```

Hints format: `key:action  key:action`. No punctuation between pairs other than two spaces.

---

## Modal System (Port canonical — migrate Shingle toward this)

Port's `tools/port/src/modal.rs` provides the complete modal toolkit:

| Variant | Use case |
|---------|----------|
| `Modal::Help` | `?` overlay — list of key/description pairs |
| `Modal::Confirm` | Destructive actions — "y" confirm, "n"/"Esc" cancel |
| `Modal::TextInput` | Single-line text entry with cursor |
| `Modal::NumericInput` | Integer entry with min/max clamping |
| `Modal::SingleSelect` | Dropdown picker with j/k navigation |
| `Modal::MultiSelect` | Checkbox list with Space toggle |

**To open a modal:**
```rust
app.active_modal = Some(Modal::Confirm {
    title: "Delete exclusion".to_string(),
    message: "Remove Apple from exclusions?".to_string(),
});
```

**To handle modal result in `main.rs` event loop:**
```rust
if let Some(ref mut modal) = app.active_modal {
    if let Some(result) = modal.handle_key(key_code) {
        match result {
            ModalResult::Confirmed => { /* do the thing */ }
            _ => {}
        }
        app.active_modal = None;
    }
    continue; // modal eats all keys while open
}
```

Shingle doesn't have modals yet. Add `modal.rs` (copy from Port, adjust palette constants) when Shingle needs confirm dialogs or text input overlays.

---

## Async Data Loading

Both tools use tokio + tokio-postgres. Two patterns apply depending on whether the main loop is running yet.

### Startup: block in parallel until ready

At app start, nothing is rendering yet, so `tokio::try_join!` is the right tool — fetches run in parallel and we block until all resolve. Use this only for data needed before the first paint.

```rust
// tools/port/src/main.rs — startup
let (notes, prices, exclusions) = tokio::try_join!(
    db::fetch_notes(&client),
    db::fetch_prices(&client),
    db::fetch_exclusions(&client),
)?;
```

If startup load exceeds ~400ms, show a **boot splash** (centered card: "Port — loading…") before the `try_join!` resolves. See `ui_kits/terminal/loading.html` in the design kit for the visual.

### In-session: spawned tasks + mpsc channel + `tokio::select!`

Once the main loop is running, **never block rendering on a DB query.** The canonical shape is a `tokio::select!` event loop that arbitrates between (a) keystrokes, (b) a redraw tick, and (c) finished load results delivered via `mpsc::channel`:

```rust
// Post-ETH-802 canonical main loop shape

let (key_tx, mut key_rx) = tokio::sync::mpsc::channel::<KeyEvent>(32);
let (load_tx, mut load_rx) = tokio::sync::mpsc::channel::<LoadResult>(16);

// Spawn a blocking task that forwards crossterm key events into key_rx
tokio::task::spawn_blocking(move || {
    loop {
        if event::poll(Duration::from_millis(100)).unwrap_or(false) {
            if let Ok(Event::Key(k)) = event::read() {
                let _ = key_tx.blocking_send(k);
            }
        }
    }
});

// Redraw ticker — caps frame rate, drives lag-splash timer
let mut tick = tokio::time::interval(Duration::from_millis(100));

loop {
    terminal.draw(|f| ui::draw(f, &app))?;

    tokio::select! {
        // 1. User input
        Some(key) = key_rx.recv() => {
            if !handle_key(&mut app, key, &load_tx) { break; }
        }
        // 2. Finished background load
        Some(result) = load_rx.recv() => {
            app.apply_load_result(result);
        }
        // 3. Frame tick — redraws even when nothing changed (picks up lag-splash state)
        _ = tick.tick() => { /* fall through to redraw */ }
    }
}
```

To trigger an in-session load (refresh, fetch ticker chart, etc.), spawn a task that sends its result back on `load_tx`:

```rust
let tx = load_tx.clone();
let client = app.db.clone();
app.loading.insert(LoadKind::Notes);  // mark in-flight for status indicator
tokio::spawn(async move {
    let res = db::fetch_notes(&client).await;
    let _ = tx.send(LoadResult::Notes(res)).await;
});
```

Define `LoadKind` and `LoadResult` enums in `app.rs` listing every refreshable data source. `app.apply_load_result` matches on the variant, updates the relevant field, and clears the matching `loading` set entry.

### Lag-splash: the 400ms rule

Any in-session load that hasn't completed within **400ms** of its spawn must show a centered overlay with what's loading ("Loading notes…"). Below 400ms, show only a brief status-line indicator — no overlay (avoids flicker on fast queries). Source: `ui_kits/terminal/loading.html` in the design kit.

The redraw-on-tick arm of `tokio::select!` gives you the timer for this — each tick, check `Instant::now() - spawn_time` for every in-flight load and decide whether to render the splash.

### Rules

- **Never `.await` in the render path.** Use spawn + channel.
- **Startup `try_join!` is OK** if guarded by a boot splash and stays under ~1s in normal conditions.
- **Status indicator goes in the context line (row 1) or footer (row 3).** Never in the body — that shifts layout.
- **Every in-session load has a `LoadKind` variant.** This lets the UI show "what's loading" in the lag-splash.

```rust
// Brief in-flight indicator in the context line
if !app.loading.is_empty() {
    Span::styled("⠋ Loading…", Style::default().fg(TEAL))
}
```

---

## DB Write Pattern

Port's `tools/port/src/actions.rs` is canonical for write operations:

```rust
pub async fn set_tick(client: &Client, ticker: &str, score: i32, reason: &str) -> Result<()> {
    client.execute(
        "SELECT research.set_tick_score($1, $2, $3, $4)",
        &[&ticker, &score, &"tui", &reason],
    ).await.context("Failed to set tick score via RPC")?;
    Ok(())
}
```

- Raw SQL only — no ORM
- Use named PostgreSQL RPCs/functions where they exist (`research.set_tick_score`, etc.)
- Context string: always `"tui"` as the source identifier
- Return `anyhow::Result<()>` — propagate errors up to the event loop for display

---

## Adding a New Screen — Checklist

**Before writing any rendering code:**

- [ ] Add variant to `Screen` enum in `src/app.rs`
- [ ] Add label to `Screen::label()` match arm
- [ ] Add to `Screen::all()` array (sets tab order)
- [ ] Add state fields to `App` struct (grouped with a `// ── ScreenName ──` comment)
- [ ] Add data loading in `main.rs` startup block (parallel with existing loads)

**Layout (in new `src/screens/<name>.rs`):**

- [ ] Accept `(f: &mut Frame, app: &App, area: Rect)` signature
- [ ] Split `area` into content + status strip if needed: `Constraint::Min(0)` + `Constraint::Length(1)`
- [ ] All pane borders use `border_style(focused)` — not hardcoded colors
- [ ] No hardcoded `Color::Rgb(...)` — use palette constants

**Interaction:**

- [ ] Add match arm in `draw_body()` in `src/ui.rs`
- [ ] Add key handler block in `main.rs` event loop (guarded by `app.current_screen == Screen::NewName`)
- [ ] Add context line content in `draw_context_line()` (position / filter state)
- [ ] Add footer hints in `draw_footer()` for this screen

**Polish:**

- [ ] Selected row highlighted: `Style::default().fg(TEAL).add_modifier(Modifier::BOLD)` — or `TEAL_LIGHT` for list items in a panel with TEAL border
- [ ] Empty state: render a `Paragraph` with DarkGray text explaining why it's empty and what to do
- [ ] `?` opens `Modal::Help` with screen-specific keybindings

---

## Code Review Checklist

When reviewing any TUI change:

**Layout stability**
- [ ] No widget changes position when data updates (constraints are static)
- [ ] `Constraint::Length` for chrome rows (tab bar, footer, context) — never `Percentage` for fixed chrome

**Focus**
- [ ] Every focusable pane changes border color on focus gain/loss
- [ ] `Tab` key cycles focus correctly
- [ ] Focus state is serialized in `App` (e.g., `social_focus: SocialFocus`) — not derived at render time

**Colors**
- [ ] No new inline `Color::Rgb(...)` literals — all use named constants
- [ ] Shingle: any Color::Yellow for focus migrated to TEAL
- [ ] Semantic colors only: GAIN/LOSS for financial +/−, TEAL for selection, DANGER for errors

**Footer**
- [ ] Footer updates when focus changes to a different pane on the same screen
- [ ] Hints cover the 4–5 most likely next actions (not exhaustive — that's what `?` is for)

**Modals**
- [ ] Destructive actions (delete, override) go through `Modal::Confirm` before executing
- [ ] Modal eats all key events while open (`continue` after `handle_key`)
- [ ] `active_modal = None` after any `ModalResult`

**Async**
- [ ] No `.await` in the synchronous render path
- [ ] In-session loads spawn via `tokio::spawn` + send result on `load_tx`
- [ ] Event loop uses `tokio::select!` — keystrokes, load results, and redraw tick as separate arms
- [ ] Every refreshable data source has a `LoadKind` + `LoadResult` variant
- [ ] Loads >400ms trigger a centered lag-splash overlay naming what's loading (`ui_kits/terminal/loading.html`)
- [ ] Brief in-flight indicator in context line or footer for <400ms loads

**Accessibility / degradation**
- [ ] Works at 80×24 (minimum terminal size) without truncation errors
- [ ] No use of Unicode box-drawing that degrades badly in 16-color terminals (stick to `Borders::ALL` and `Borders::NONE`)

---

## Architecture: Flat App vs. Component

**Current state:** Both tools use a flat `App` struct. This is intentional and appropriate at current scale.

**Keep flat when:** A screen's state is simple (one list, one selection index, one edit buffer). The majority of screens qualify.

**Migrate a screen to a struct when:**
- The screen has >5 state fields of its own
- The screen has its own initialization logic (e.g., needs to parse data on first render)
- Two screens share overlapping field names in `App` causing confusion

**Component migration pattern (Port already uses this informally for modals):**

```rust
// Before: scattered in App
pub exclusions_list: Vec<Exclusion>,
pub exclusions_selected: usize,
pub exclusions_filter: String,
pub exclusions_edit_mode: bool,
pub exclusions_edit_buffer: String,

// After: screen-owned struct
pub struct ExclusionsScreen {
    pub items: Vec<Exclusion>,
    pub selected: usize,
    pub filter: String,
    pub edit_mode: bool,
    pub edit_buffer: String,
}
// In App: pub exclusions: ExclusionsScreen
```

The `draw()` function and key handler call into the screen struct's own methods. Don't adopt full Flux — unidirectional data flow is valuable at web scale, not at a single-user CIO tool.

**Do not prematurely add** `reratui`, `tundra`, `ratatui_router`, or `ratatui-form`. Evaluate each only when the specific problem it solves (reactive rerenders, modal state management, multi-screen routing, form validation) becomes painful in practice.

---

## Keybinding Layers (both tools)

| Layer | Keys | Rule |
|-------|------|------|
| Universal | `Esc` cancel/back, `Enter` confirm, `Tab`/`Shift+Tab` cycle focus | Never reassign these |
| Navigation | `j`/`k` or `↑`/`↓` list movement, `g`/`G` top/bottom | Consistent across all list views |
| Contextual | `e` edit, `d` delete, `c` create, `r` refresh, `n` new, `/` filter | Single-char mnemonics, only active in relevant pane |
| Global | `?` help modal, `Ctrl+C`/`q` quit | Override any pane context |
| Screen switch (Port) | `1`–`9` direct screen jump | Number corresponds to `Screen::all()` index + 1 |

**Conflicts to avoid:**
- Don't use `q` for "cancel" inside a pane — it quits the whole app
- Don't use `e` in a text-input mode — it types the letter
- `Ctrl+P` is reserved for publish in Shingle's Social screen

---

## File Locations

| What | Port | Shingle |
|------|------|---------|
| App state | `src/app.rs` | `src/app.rs` |
| Top-level draw | `src/ui.rs` | `src/ui.rs` |
| Screen modules | `src/screens/<name>.rs` | `src/screens/<name>.rs` |
| DB queries | `src/db.rs` | `src/db.rs` |
| DB writes / RPCs | `src/actions.rs` | _(add when needed)_ |
| Modal system | `src/modal.rs` | _(copy from Port when needed)_ |
| Data models | `src/note.rs`, `src/prices.rs` | `src/models.rs` |
| Main event loop | `src/main.rs` | `src/main.rs` |
| Brand palette | `src/palette.rs` | `src/palette.rs` |

---

## Common Anti-Patterns to Reject

- **Hide-and-hover**: important data behind 3+ keystrokes. Surface it in context line or split pane.
- **Monolithic match arms**: if a screen's key handler exceeds ~50 lines, extract to `screen.handle_key(&mut app, key)`.
- **Hardcoded dimensions**: `centered_rect(52, 7, area)` is fine for modals; for content panes always use layout constraints so the terminal can resize.
- **Silent failures**: if a DB write fails, surface the error in `app.status_msg` — never `let _ = result` in the event loop.
- **Borrow fights from large App**: if you're fighting the borrow checker to get two mutable refs from `App`, that screen's state belongs in its own struct.
