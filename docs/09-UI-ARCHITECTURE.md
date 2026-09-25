# UI Architecture

## Overview
CairnAI provides a calm, private, and deliberate interface for personal AI work. The UI follows the design system in DESIGN.md and is organized into three regions: Sessions sidebar, main canvas, and optional context drawer.

## Core Layout
```
┌─────────────────────────────────────────────────┐
│                    TOPBAR                       │
├───────────────────┬───────────────────────────────┤
│                   │                              │
│    SESSIONS      │   MAIN CANVAS                │
│     SIDEBAR      │                              │
│                   │                              │
│                   │    (Landing state OR        │
│                   │     Conversation state)   │
│                   │                              │
├───────────────────┴───────────────────────────────┤
│                    BOTTOM BAR                    │
└─────────────────────────────────────────────────┘
```

## Three Main Regions

### 1. Sessions Sidebar
- **Width:** 288px expanded, 64px collapsed rail
- **Purpose:** Session navigation and management
- **Contents:** Brand, toggle, New session button, search, session list, provider status, settings, open source link
- **Features:** Collapse, search, session management, provider status, settings

### 2. Main Canvas
- **Purpose:** Primary workspace for conversation or landing content
- **States:**
  - **Landing:** Hero, composer, suggestion chips, model strip, feature trio, footer
  - **Conversation:** Messages, topbar, composer, context drawer toggle

### 3. Context Drawer (Right)
- **Width:** 360px
- **Trigger:** Topbar or composer
- **Tabs:** Files, Retrieval, Tools, Skills, Model

## Detailed Component Structure

### Sessions Sidebar

#### Top Section
- **Brand:** CairnAI cairn mark + wordmark
  - Clicking returns to Landing state with new empty session
- **Collapse Toggle:** Expands/collapses sidebar
- **New Session Button:** Full width, primary style, `md` radius, shortcut `Cmd+Shift+O`
- **Search Input:** `surface` fill, `md` radius, magnifier icon, shortcut `Cmd+K`

#### Session List
- **Grouping:** Today, Yesterday, Previous 7 days, Older
- **Headers:** `label` style in `ink-muted`
- **Rows:** 44px height, `sm` radius, show:
  - Session title (single line, truncated)
  - Model name + relative time (label style)
  - Paperclip icon with count when files attached
- **Active Row:** `accent-tint` fill, `ink` title, no border
- **Hover:** `surface-hover` fill, kebab button appears
- **Row Menu:** Rename, Archive, Delete (with confirmation)
- **Archived:** Collapsed group at bottom, muted, with Restore and Delete
- **Streaming Indicator:** Small pulsing accent dot for active rows

#### Bottom Section
- **Provider Status:** Dot + text for default provider
  - Connected (`success`), Checking (`slate`), Unreachable (`danger`), etc.
  - Summary count for other providers with problems
  - Opens Settings, Providers on click
- **Settings Icon Button:** Opens settings dialog
- **Open Source Link:** Repository link in `ink-muted`

### Main Canvas

#### Landing State (Empty Session)
The landing doubles as the project's front page, explaining the product without leaving the app.

##### Hero Section
- **Eyebrow:** `Open source · Runs locally` (label style)
- **Headline:** "A personal AI workspace that runs on your machine" (display serif, centered)
- **Supporting Text:** 60ch max, "Connect 9Router or another supported provider..."
  - Remove clauses for features that haven't shipped
  - Content from design.md section 7.1
- **Background:** Shoreline gradient (light) or Deep water (dark) from section 2.4
- **Suggestion Chips:** 3-4 `mist` chips under composer, clicking fills composer
- **Model Strip:** Chips listing models actually returned by configured providers
  - If no provider: honest line with "Add provider" button
  - If no backend: strip hidden

##### Composer (Center, 760px max, elevation 3)
- **Container:** `surface` fill, 1px `border-strong`, `xl` radius
- **Textarea:** Auto-grows from 2 lines to max 10, then scrolls
  - Placeholder "Ask anything" in `ink-muted`
- **Attachment/Skill Row:** Above textarea, only when active
  - File chips with type icon, name, status (uploading/processing/ready/failed)
  - Skill chips (`mist` fill, `sm` radius) with skill name in `label` style
  - Each chip has remove button
- **Skill Picker:** Typing `/` at start opens popover listing enabled skills
  - Arrow keys navigate, Enter activates, Esc closes
  - Filters as user types
- **Toolbar:** Below textarea
  - Attach (paperclip) button: opens file picker
  - Tools button: icon + count of enabled tools (opens Tools tab)
  - Model picker: secondary button showing current model (`label` style)
  - Spacer
  - Send button: 40px square, `md` radius, primary style, arrow-up icon
    - Disabled when empty
    - Becomes Stop (square icon) while streaming
- **Keyboard:** Enter sends, Shift+Enter newline, Esc stops response, `Cmd+/` toggles drawer
- **Focus:** Container shows 2px `accent` ring with 2px offset

##### Model Strip
- Single row of `label`-style chips listing models actually returned by configured providers
- Rest behind model picker
- If no provider: one honest line with "Add provider" button
- If no backend: strip hidden
- Never placeholder names, provider logos, or counts

##### Feature Trio (3 cards, `lg` radius, `surface` fill, hairline border, elevation 1)
- **Sessions:** "Every conversation is saved on this machine. Switch sessions without losing the one you left."
- **Files:** "Attach PDFs, text, code and data. Ask questions and see which passages an answer used."
- **Tools:** "Connect MCP servers. You approve each tool call before it runs."
- Each card: h3 title, body copy above, one illustration from real components or screenshot
- Illustrations labeled "Example" and contain no invented statistics

##### How It Works (Horizontal strip)
- You, CairnAI on this machine, Your provider or 9Router, Models, joined by thin lines
- Caption underneath: "Conversations and files stay in a local workspace."

##### Footer Strip
- Open-source statement with license name, version, links to Repository, Privacy and Terms
- No sign-up call to action

#### Conversation State (Session with messages)
##### Topbar
- **Left:** sidebar toggle (small screens only)
- **Center-left:** Session title as inline-editable heading (Geist 600, 16px)
- **Right:** Model picker, then context drawer toggle (with file count badge)

##### Messages
- Single centered column, 720px max, 24px gap between messages
- **User message:** right-aligned bubble, `accent-tint` fill, `ink` text, `lg` radius, max 80% width
  - Attached files appear as file chips above text
- **Assistant message:** no bubble, plain text on `canvas`, with 24px cairn mark on first paragraph
- **Markdown:** headings, lists, tables (scroll horizontally), blockquotes, inline code (`xs` radius, `surface-sunken`), fenced code blocks (`md` radius, `surface-sunken`)
- **Streaming:** text appends with soft caret, announce completion not every token (`aria-live="polite"`)
- **Message footer:** `label` style, `ink-muted`, shown on hover/focus: copy, regenerate, model line
  - Model line: model that responded, includes provider name
  - If provider reports something different from requested: show both (`requested: combo-name, answered by: model-id`)

##### Retrieval Citations
- Answers that used documents show numbered citation chips (`sand` tint, `sm` radius, `label` text) inline
- Clicking opens popover with file name, location, retrieved passage
- Passage in quoted block labeled "Untrusted document content" with shield icon

##### Tool Calls (MCP)
- Each tool call is a card between message paragraphs: `lg` radius, `surface` fill, hairline border
- Header: tool name (`label` style), server name (`ink-muted`), status badge (running, succeeded, failed)
- Body collapsed by default
- Expanded: arguments and result as formatted JSON in code block
- Tool output labeled "Untrusted tool output"

##### Context Drawer
- Right-hand panel, 360px, `surface` fill, `xl` radius on leading corners, elevation 3
- Five tabs in segmented control (`sm` radius): Files, Retrieval, Tools, Skills, Model
- Content per session

##### Settings Dialog
- Centered dialog (`lg` radius, 560px), not a page
- Opened from sidebar footer or error banner

##### Composer (Dock state)
- Container: `surface` fill, 1px `border-strong`, `xl` radius, elevation 2
- Context meter: thin 2px bar with `accent` fill, warning above 80%
- Auto-grows from 2 lines to 10, then scrolls

##### Model Picker Popover
- Searchable list, grouped by provider
- Each row: model/combo ID in `label` style, capability badges
- Unknown capabilities: nothing or muted "unknown"
- At bottom: Add model ID and Manage providers (opens Settings)

##### Skill Editor
- Dialog (`lg` radius, 640px)
- Fields: name, description with counter, instruction body in monospace with live token estimate
- Save and Cancel

## Component Details

### Buttons (All use `md` radius, no pills/circles)
- **Primary:** `ink` fill, `canvas` text, 40px high, 16px padding
- **Secondary:** `surface` fill, 1px `border-strong`, `ink` text
- **Ghost:** Transparent, `ink-secondary` text
- **Icon:** 36px square visual, 40px touch, 18px Lucide icon
- **Danger:** `danger` fill, white text

### Chips and Badges
- `sm` radius (6px), 28px high, `label` text
- Variants: neutral (`surface-sunken`), suggestion (`mist`), file (`sand` tint), status (semantic tint + icon), "Soon" (outlined)

### Inputs
- `surface` fill, 1px `border-strong`, `md` radius, 40px high
- Focus: 2px `accent` ring
- Error: `danger` border plus message below

### Dialogs, Popovers, Toasts
- Trapped focus, close on Esc
- Popovers close on outside click and Esc
- Toasts auto-dismiss after 5s

### Icons
- Lucide, 1.5px stroke, 18px default, 20px in composer
- No emoji, no icon fonts of emoji
- No decorative gradient icons

### Brand Mark
- Small cairn: three stacked rounded stones decreasing width, top stone in `accent`, lower two in `ink`
- Wordmark: "Cairn" (Fraunces 400) + "AI" (Geist 500)
- Draw as simple vector, not framework default

## Breakpoints
- **1280px+:** Sidebar visible. Context drawer pushes canvas
- **900-1279px:** Sidebar collapses to rail. Drawer overlays canvas
- **640-899px:** Sidebar is off-canvas drawer opened from topbar
- **Under 640px:** Off-canvas sidebar. Context drawer becomes bottom sheet

## Motion
- Hover/pressed/focus: 120ms `cubic-bezier(0.2, 0, 0, 1)`
- Popover/dropdown: 160ms
- Drawer/sheet/dialog: 240ms
- Compose-to-dock: 240ms

## Accessibility
- WCAG AA contrast (verified with linter)
- Every interactive element keyboard reachable
- Focus ring always visible
- Touch targets at least 40px
- Focus trap/restore for drawers and dialogs
- New assistant content announced politely (`aria-live="polite"`)
- Landmark structure: `nav` (sessions), `main` (canvas), `complementary` (context drawer)

## Copy Rules (Design.md section 0.3)
- Say what a thing is and what it does
- Every claim needs a mechanism in the product
- Present tense, sentence case, no exclamation marks
- Buttons name the action
- No em dashes (use commas, colons, periods, parentheses)
- All numbers shown in interface come from real data
- Example content inside feature illustrations is labeled "Example" and contains no invented statistics

## Component Library Dependencies
This UI architecture depends on:
- **DESIGN.md:** Visual design system (colors, typography, spacing, components)
- **Geist font family:** Interface and body font (self-hosted)
- **Fraunces font family:** Display and headline font (self-hosted)
- **Geist Mono:** Technical labels and code (self-hosted)
- **Lucide React:** Icon library
- **Tailwind CSS:** Utility-first styling
- **Radix UI:** Accessible component primitives
- **Sonner:** Toast notifications
- **Framer Motion:** Animations

## UI Architecture Constraints
- Follow design.md exactly (section 0.2 bans apply)
- No purple gradients, pill shapes, fake metrics, emoji, em dashes
- No scroll or cursor animations
- No parallax, scroll-jacking, reveal-on-scroll
- No custom cursors, cursor followers, magnetic buttons
- No stock photos, generated images, 3D blobs
- No "Made with AI" badges
- Everything reachable from application page
- Legal pages have minimal header/footer, no sessions sidebar