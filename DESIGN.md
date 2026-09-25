---
name: CairnAI
description: Design system for CairnAI, a local-first personal AI workspace. One application page with a sessions sidebar, no sign-in, plus two static legal pages.
version: 1.1.0
mode-default: light
mode-supported: [light, dark]
colors:
  canvas: "#F5F3EC"
  surface: "#FFFFFF"
  surface-sunken: "#EDF2F3"
  surface-hover: "#E3EBEE"
  border: "#D9E2E6"
  border-strong: "#B9C6CD"
  ink: "#0F2438"
  ink-secondary: "#3E5163"
  ink-muted: "#5B6B78"
  accent: "#3A86C8"
  accent-strong: "#1F5F99"
  accent-tint: "#E1EEF9"
  tide: "#BFE6F0"
  mist: "#E6F5F8"
  sand: "#D8C9AE"
  slate: "#6B7178"
  success: "#2E9E6B"
  warning: "#C77A12"
  danger: "#C93C33"
typography:
  display:
    fontFamily: Fraunces
    fontSize: 64px
    fontWeight: 400
    lineHeight: 0.96
    letterSpacing: -0.02em
  h1:
    fontFamily: Fraunces
    fontSize: 48px
    fontWeight: 400
    lineHeight: 1.0
    letterSpacing: -0.01em
  h2:
    fontFamily: Fraunces
    fontSize: 32px
    fontWeight: 400
    lineHeight: 1.04
  h3:
    fontFamily: Geist
    fontSize: 20px
    fontWeight: 600
    lineHeight: 1.3
  body:
    fontFamily: Geist
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.55
  small:
    fontFamily: Geist
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.5
  label:
    fontFamily: Geist Mono
    fontSize: 12px
    fontWeight: 400
    lineHeight: 1.3
    letterSpacing: 0.02em
rounded:
  xs: 4px
  sm: 6px
  md: 8px
  lg: 12px
  xl: 16px
spacing:
  base: 4px
  scale: [4, 8, 12, 16, 20, 24, 32, 40, 56, 80, 120]
---

# CairnAI design system

Visual authority for CairnAI. Where this file and any other document disagree on look and feel, this file wins. Where it is silent, make a decision consistent with the principles below and record it in `docs/09-UI-ARCHITECTURE.md`.

## 0. Scope and non-negotiables

### 0.1 Scope

- **One application page.** The product is a single page at `/`. The active session may be reflected in the URL query or hash so a reload restores it. That is state, not a page.
- **Two static legal pages.** `/privacy` and `/terms` are the only other routes. They are required before any public launch (section 17) and are linked from the footer strip and the Settings dialog. Nothing else gets its own route.
- **Two regions in the application page:** a **Sessions sidebar** and a **main canvas**. The main canvas has two states: **Landing** (empty session) and **Conversation** (session with messages).
- **No sign-in, sign-up, account menu, avatar, pricing, billing or upgrade prompts.** CairnAI runs locally and is open source. There is no user account.
- **No other pages.** Files, retrieval, tools (MCP), models and settings are **drawers, popovers and dialogs** on the application page. Nothing navigates away.
- **Everything the product does is reachable from the application page.** Sessions, streaming chat, file upload, document retrieval, MCP tools, skills, model selection, provider settings.
- **No external requests for design assets.** Self-host fonts and icons. No CDN fonts, no analytics, no remote images.

### 0.2 Never do this

These are absolute. If a design idea needs one of these, change the idea.

| Never | Details |
|---|---|
| Purple gradients | No purple, violet, indigo-to-pink, magenta or neon in any gradient or glow. |
| Generic gradient styling | No mesh gradients, aurora effects, blurred color blobs, glow orbs, gradient text, gradient buttons, gradient borders. Only the gradients in section 2.4 exist. |
| Pill-shaped buttons | No button, chip, badge, input, tab or switch uses a fully rounded (pill) or circular shape. Maximum control radius is 8px (see section 5). |
| Fake social proof | No testimonials, reviews, star ratings, quotes from people, "trusted by" rows, or company logos. |
| Fake metrics | No customer counters, user counts, download counts, uptime claims, speed or benchmark claims, or "N models supported" numbers. Only show numbers the running app actually computed. |
| Vague hero text | The hero must say what CairnAI is and what it does in concrete terms. See section 0.3. |
| Emoji as icons | No emoji anywhere in the interface or copy. Icons are Lucide only. |
| Em dashes | No em dash characters in any UI copy, legal page, README text or docs shown to users. Use a comma, colon, period or parentheses. No en dashes for ranges either: write "10 to 20". |
| Over-the-top scroll animation | No parallax, scroll-jacking, scroll-scrubbed sequences, or reveal-on-scroll effects. See section 13. |
| Cursor animation | No custom cursors, cursor followers, magnetic buttons, or spotlight or glow effects that track the pointer. |
| AI slop photos | No stock photography, generated images, 3D blobs, abstract "AI brain" or network art. Imagery is real UI screenshots or real components only. |
| AI slop copy | See the banned list in section 0.3. |
| Builder badges | No "Made with AI", "Built with", "Edit with" or similar tag, banner or watermark from any tool or host. |

### 0.3 Copy rules

Copy is part of the design. Write it plainly and specifically.

- Say what a thing is and what it does. Prefer a concrete noun and a plain verb over an abstract benefit.
- Every claim needs a mechanism in the product. If a feature has not shipped, do not mention it, or mark it with a "Soon" badge.
- Present tense. Sentence case. No exclamation marks. No trailing period on headlines.
- No rhythm-for-its-own-sake triplets such as "Fast. Private. Yours."
- Buttons name the action: **New session**, **Add files**, **Test connection**.
- **Banned words and phrases:** supercharge, unleash, unlock (as a metaphor), elevate, seamless, effortless, revolutionary, next-generation, cutting-edge, game-changing, empower, leverage, harness, delve, tapestry, journey, reimagine, "the future of", "AI-powered", "your AI companion", "magic".
- No em dashes (section 0.2).
- All numbers shown in the interface come from real data. Example content inside feature illustrations is labelled "Example" and contains no invented statistics.

## 1. Overview

CairnAI is a personal AI workspace, not a chatbot clone. The design should feel calm, private and deliberate: a quiet room you come back to, with a visible trail of where you have been.

**The name gives the metaphor.** A cairn is a stack of stones marking a trail. Sessions are the stones. The sidebar is the trail. The interface should make it feel safe to leave and safe to return.

**Two references, adapted rather than copied:**

| From | What is borrowed | What is deliberately not borrowed |
|---|---|---|
| Captions (`captions.ai`) | Airy light layout, one large central input as the hero, soft cool tones, the Split Studio macrostructure, the hero, feature-trio rhythm | Its proprietary typefaces, its neon accent palette, its pill shapes, its marketing chrome |
| Venice (`venice.ai`) | Deep navy foundations, one blue accent used sparingly, warm paper neutrals, an expressive serif for headlines with a plain sans for the interface, monospace for technical labels, "Ask anything" as the entry point, composed headlines | Its logo, wordmark and keys artwork, its proprietary typefaces, its palette values, any Venice branding |

**Tone words:** calm, private, composed, editorial, local, plain-spoken, unhurried.

**Personality rules**

- Softness comes from color, spacing and type, not from pill shapes or blur.
- One accent color. It marks focus, selection and state. It is never a large background.
- Quiet by default. Motion and decoration appear only where they carry meaning.
- Technical detail (model IDs, token counts, tool names) is shown in monospace so it reads as data, not prose.

## 2. Colors

All values are original tokens derived from the mood of the two references. Do not substitute either brand's exact palette.

### 2.1 Light theme (default)

| Token | Hex | Role |
|---|---|---|
| `canvas` | `#F5F3EC` | Page background. Warm paper. |
| `surface` | `#FFFFFF` | Cards, composer, popovers, drawers. |
| `surface-sunken` | `#EDF2F3` | Sidebar, code blocks, inset areas. |
| `surface-hover` | `#E3EBEE` | Hover fill on list rows and ghost buttons. |
| `border` | `#D9E2E6` | Default 1px hairlines. |
| `border-strong` | `#B9C6CD` | Input borders, emphasized dividers. |
| `ink` | `#0F2438` | Primary text and primary button fill. |
| `ink-secondary` | `#3E5163` | Secondary text. |
| `ink-muted` | `#5B6B78` | Metadata, placeholders, timestamps. |
| `accent` | `#3A86C8` | Focus ring, selection indicator, progress, icons. Fills and large elements only. |
| `accent-strong` | `#1F5F99` | Accent as text or link. Passes AA on white and canvas. |
| `accent-tint` | `#E1EEF9` | Selected session row, selected chip, user message bubble. |
| `tide` | `#BFE6F0` | Search-match highlight and text selection background. |
| `mist` | `#E6F5F8` | Suggestion chips, soft banners, top of the hero gradient. |
| `sand` | `#D8C9AE` | Warm stone tone. File and document chips. |
| `slate` | `#6B7178` | Neutral stone. Disabled and inert icons. |
| `success` | `#2E9E6B` | Connected, ready. Tint `#DDF3E8`. |
| `warning` | `#C77A12` | Degraded, needs attention. Tint `#FBEBD0`, text `#7A4A06`. |
| `danger` | `#C93C33` | Failed, destructive. Tint `#FBE1DE`, text `#9E2A23`. |

### 2.2 Dark theme

| Token | Hex |
|---|---|
| `canvas` | `#0B141C` |
| `surface` | `#121F2A` |
| `surface-sunken` | `#08111A` |
| `surface-hover` | `#182835` |
| `border` | `#22333F` |
| `border-strong` | `#33495A` |
| `ink` | `#EEF0E8` |
| `ink-secondary` | `#BAC5CD` |
| `ink-muted` | `#8C9BA7` |
| `accent` | `#5AA6E8` |
| `accent-strong` | `#8CC1F0` |
| `accent-tint` | `#142B3F` |
| `tide` | `#173847` |
| `mist` | `#10222E` |
| `sand` | `#B9A98F` |
| `slate` | `#8A9096` |
| `success` | `#4CC38A` |
| `warning` | `#E8A73A` |
| `danger` | `#F0776F` |

### 2.3 Color rules

- Follow the system theme by default, with a manual choice in Settings (Light, Dark, System). Apply the theme before first paint to avoid a flash (section 16).
- **Primary buttons use the `ink` token as the fill with `canvas` as the text color, in both themes** (navy on light, off-white on dark), not accent. `accent` with white text is only about 3.9:1 and fails AA for body-size text.
- Use `accent-strong` for any accent-colored text. Use `accent` for non-text indicators.
- Never rely on color alone for state. Pair every status color with an icon or a text label.
- No purple, violet or magenta anywhere in the palette.
- Contrast targets are WCAG AA (4.5:1 body text, 3:1 large text and UI). The main pairs were spot-checked, but run a contrast linter over the final CSS before shipping.

### 2.4 Gradients

CairnAI uses **three gradients in total.** They are quiet, low in chroma, and stay close in lightness so text contrast holds across the whole surface. They follow the product's own metaphors: light water meeting sand, and a single lamp over dark water. They are not decorative filler.

| Name | Theme | Use | Stops |
|---|---|---|---|
| **Shoreline** | Light | Backdrop of the Landing hero only | `mist #E6F5F8` at 0% to `canvas #F5F3EC` at 68%, vertical |
| **Deep water** | Dark | Backdrop of the Landing hero only | `#16303F` at 0% to `canvas #0B141C` at 70%, one radial light from above center |
| **Canvas fade** | Both | Functional. Softens content scrolling under the topbar and above the docked composer | `canvas` at 0% opacity to `canvas` at 100%, 32px tall |

```css
/* light */
--grad-hero: linear-gradient(180deg, #E6F5F8 0%, #F5F3EC 68%);
--grad-fade: linear-gradient(180deg, rgba(245, 243, 236, 0) 0%, #F5F3EC 100%);

/* dark */
--grad-hero: radial-gradient(90% 60% at 50% -10%, #16303F 0%, #0B141C 70%);
--grad-fade: linear-gradient(180deg, rgba(11, 20, 28, 0) 0%, #0B141C 100%);
```

Measured on the stop with the least contrast in each theme, `ink` is above 14:1 (light) and above 11:1 (dark), and `ink-muted` is about 4.9:1 (light) and about 4.8:1 (dark).

**Gradient rules**

- Only Shoreline, Deep water and Canvas fade may exist. Adding another needs a written reason in `docs/09-UI-ARCHITECTURE.md`.
- At most one decorative gradient is visible at a time. It is static: never animated, never parallaxed, never following the pointer.
- Gradients appear only as a page or section backdrop. Never on buttons, text, icons, borders, cards or the brand mark.
- Stops keep near-equal lightness (no more than 15 L* apart) and low saturation.

## 3. Typography

Both references pair an expressive serif for headlines with a clean sans for the interface. Their fonts are proprietary, so CairnAI uses open-licensed equivalents, **self-hosted**.

| Role | Family | Fallback stack |
|---|---|---|
| Display and headlines | **Fraunces** (variable, OFL). High optical size, `SOFT` 0, `WONK` 0, weight 400. | `"Fraunces", "Iowan Old Style", "Times New Roman", serif` |
| Interface and body | **Geist** (variable, OFL) | `"Geist", ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif` |
| Technical labels and code | **Geist Mono** (variable, OFL) | `"Geist Mono", ui-monospace, "SF Mono", Menlo, Consolas, monospace` |

Bundle with a self-hosting package such as Fontsource, and confirm exact package names when installing. Do not link to Google Fonts.

### 3.1 Type scale

| Role | Family | Desktop | Mobile | Weight | Line-height | Letter-spacing |
|---|---|---|---|---|---|---|
| display | Fraunces | 64px | 40px | 400 | 0.96 | -0.02em |
| h1 | Fraunces | 48px | 36px | 400 | 1.0 | -0.01em |
| h2 | Fraunces | 32px | 28px | 400 | 1.04 | 0 |
| h3 | Geist | 20px | 18px | 600 | 1.3 | 0 |
| body | Geist | 16px | 16px | 400 | 1.55 | 0 |
| chat body | Geist | 16px | 16px | 400 | 1.65 | 0 |
| small | Geist | 14px | 14px | 400 | 1.5 | 0 |
| label | Geist Mono | 12px | 12px | 400 | 1.3 | 0.02em |
| code | Geist Mono | 13.5px | 13px | 400 | 1.6 | 0 |

### 3.2 Typography rules

- Serif is for **headlines and expressive moments only**: the landing hero, empty states, section titles, legal page titles. Never for chat messages, buttons or form labels.
- Sentence case. No all-caps headings. No trailing period on headline-style text.
- Monospace is used **sparingly**: model IDs, token counts, tool names, file types, keyboard hints, code.
- Body text in chat is left-aligned with a reading width of 65 to 75 characters.
- Do not mix in extra families. Do not substitute fonts within the hierarchy.

## 4. Layout

### 4.1 App shell

```
+----------------+---------------------------------------------+
| Sessions       | Topbar: session title, model, context       |
| sidebar        +---------------------------------------------+
| 288px          |                                             |
|                |   Main canvas                               |
| + New session  |   (Landing state  or  Conversation state)   |
| Search         |                                             |
| Session list   |                                             |
| ...            +---------------------------------------------+
| Provider state |   Composer                                  |
+----------------+---------------------------------------------+
                                    Context drawer (right, 360px, on demand)
```

This is the Split Studio structure, adapted: fixed left rail plus one working canvas. The legal pages do not use it (section 17).

### 4.2 Dimensions

| Element | Value |
|---|---|
| Sidebar width | 288px expanded, 64px collapsed rail |
| Topbar height | 56px |
| Context drawer | 360px |
| Landing content width | up to 1080px |
| Composer width (landing) | up to 760px |
| Reading column (conversation and legal pages) | 720px max, centered |
| Section gap (landing, desktop) | 120px, 72px on mobile |

### 4.3 Spacing

Base step **4px**. Scale: `4, 8, 12, 16, 20, 24, 32, 40, 56, 80, 120`. Round any other value to the nearest step.

### 4.4 Breakpoints

| Range | Behavior |
|---|---|
| 1280px and up | Sidebar visible. Context drawer **pushes** the canvas. |
| 900 to 1279px | Sidebar collapses to a rail (toggle to expand). Drawer **overlays** the canvas. |
| 640 to 899px | Sidebar is an off-canvas drawer opened from the topbar. |
| Under 640px | Off-canvas sidebar. Context drawer becomes a **bottom sheet**. Composer stays pinned. |

## 5. Elevation and shape

**Shapes.** Small, deliberate radii. Nothing is fully rounded. This is a hard rule (section 0.2).

| Token | Value | Used for |
|---|---|---|
| `xs` | 4px | Inline code, keyboard hints |
| `sm` | 6px | Chips, badges, session rows, switch tracks, tab segments |
| `md` | 8px | **Buttons**, inputs, code blocks, popovers, the Send button |
| `lg` | 12px | Cards, tool-call cards, dialogs, message bubbles |
| `xl` | 16px | The composer, drawers (leading corners), sheets (top corners) |

Status dots are 8px circles. They are indicators, not controls, and are the only circular elements allowed.

**Elevation.** Mostly flat with hairline borders. Use shadow only to lift floating layers.

| Level | Light | Dark | Used for |
|---|---|---|---|
| 0 | none | none | Sidebar, canvas, messages |
| 1 | `0 1px 2px rgba(15,36,56,.06)` | `0 1px 2px rgba(0,0,0,.4)` | Cards, chips |
| 2 | `0 4px 16px rgba(15,36,56,.08)` | `0 4px 16px rgba(0,0,0,.45)` | Popovers, dropdowns, docked composer |
| 3 | `0 8px 32px rgba(15,36,56,.10)` | `0 8px 32px rgba(0,0,0,.5)` | Composer on Landing, dialogs, drawers |

## 6. Sessions sidebar

The sidebar contains **sessions only**. No navigation to other pages.

**Top**
- Brand: CairnAI mark and wordmark. Clicking it opens a new empty session and returns the canvas to the Landing state.
- Collapse toggle.
- **New session** button: full width, primary style, plus icon, `md` radius. Shortcut `Cmd+Shift+O` or `Ctrl+Shift+O`.
- **Search sessions** input: `surface` fill, `md` radius, magnifier icon, shortcut `Cmd+K` or `Ctrl+K`. Filters as you type by title and message content. Matches are highlighted with `tide`.

**Session list**
- Grouped by recency: **Today, Yesterday, Previous 7 days, Older**. Group headers use the `label` style in `ink-muted`.
- Each row is 44px tall with `sm` radius and shows: session title (single line, truncated), and beneath it a `label`-style line with model name and relative time. A paperclip icon with a count appears when the session has files.
- **Active row:** `accent-tint` fill, `ink` title, no border stripe.
- **Hover:** `surface-hover` fill and a kebab button appears.
- Row menu: **Rename** (inline edit, Enter to save, Esc to cancel), **Archive**, **Delete** (confirmation dialog that states the session's messages and files will be removed from this machine).
- **Archived** is a collapsed group at the bottom of the list. Archived rows are muted and offer **Restore** and **Delete**.
- Switching sessions is instant and never discards the previous one. An in-flight response keeps streaming into its own session, and that row shows a small pulsing accent dot.

**Bottom**
- **Provider status:** dot plus text for the default provider, for example "Connected to 9Router". If other configured providers have problems, add a short count such as "1 provider needs attention". States: connected (`success`), checking (`slate`), unreachable (`danger`), key rejected (`danger`), no provider configured (`warning`), rate limited or exhausted (`warning`). Clicking opens Settings, Providers.
- **Settings** icon button.
- **Open source** link to the repository, in `ink-muted`.

No avatar, no username, no account.

## 7. Main canvas: Landing state

Shown whenever the active session has no messages. It doubles as the project's front page, so it explains the product without leaving the app. Everything scrolls inside the main canvas.

### 7.1 Sections, in order

1. **Hero**
   - Eyebrow in `label` style: `Open source · Runs locally`.
   - Headline in `display` serif, centered. Suggested copy: **A personal AI workspace that runs on your machine**
   - Supporting text in `body`, `ink-secondary`, 60ch max. Suggested copy: "Connect 9Router or another supported provider with your own API key, and chat with its models. Keep every session on your machine. Ask questions about your own files. Connect tools over MCP."
   - Remove any clause for a feature that has not shipped. The hero may only describe what works. "Supported" means listed as verified in `docs/13-PROVIDERS.md`, never a guess.
   - Background: the **Shoreline** gradient in light (the signature look of the product) and **Deep water** in dark (section 2.4). The gradient spans the whole hero block, from the eyebrow through the suggestion chips, and reaches plain `canvas` below it, so there is no visible edge where the hero ends. A reference render is in `cairnai_gradient_preview.png`, supplied with this file. Match its colors and softness.
2. **Composer** (the large central input, the Captions move and Venice's "Ask anything")
   - See section 9. Centered under the supporting text, 760px max, elevation 3.
3. **Suggestion chips**
   - Three or four `mist` chips (`sm` radius) under the composer. Clicking one fills the composer, it does not send. Examples: "Summarize a document", "Explain this code", "Compare two models", "Use a connected tool".
4. **Model strip**
   - A single row of `label`-style chips listing **models actually returned by the configured providers**. The rest are behind the model picker. If no provider is configured, show one honest line with an **Add provider** button. If there is no backend at all (a static public copy of the page), **hide the strip**. Never show placeholder names, provider logos or counts.
5. **Feature trio** (three cards, `lg` radius, `surface` fill, hairline border, elevation 1)
   - **Sessions:** "Every conversation is saved on this machine. Switch sessions without losing the one you left."
   - **Files:** "Attach PDFs, text, code and data. Ask questions and see which passages an answer used."
   - **Tools:** "Connect MCP servers. You approve each tool call before it runs."
   - Each card has an h3 title, the body copy above, and one illustration built from the **real components** (sidebar row, file chip with citation, tool-call card) or a real screenshot. Illustrations are labelled "Example" and contain no invented statistics.
   - Trim copy or add a "Soon" badge for any card whose feature has not shipped.
6. **How it works** (one horizontal strip)
   - `You`, `CairnAI on this machine`, `Your provider or 9Router`, `Models`, joined by thin lines. Caption underneath: "Conversations and files stay in a local workspace."
7. **Footer strip**
   - Open-source statement with the license name, version, and links to **Repository**, **Privacy** and **Terms**. No sign-up call to action.

### 7.2 Behavior

- All landing sections are visible immediately. Nothing fades or slides in on scroll.
- Sending the first message plays the **compose-to-dock transition** (section 13) and the canvas becomes the Conversation state. The landing sections collapse away.
- Opening a session that already has messages goes straight to the Conversation state.
- **New session** or the brand link returns to the Landing state.

## 8. Main canvas: Conversation state

### 8.1 Topbar (both states)

Left: sidebar toggle on small screens. Center-left: **session title** as an inline-editable heading (Geist 600, 16px). Right, in order: **model picker**, then **context drawer** toggle (with a count badge for attached files). The theme toggle lives in Settings, not here.

### 8.2 Messages

- Single centered column, 720px max, 24px gap between messages.
- **User message:** right-aligned bubble, `accent-tint` fill, `ink` text, `lg` radius, max 80% width. Attached files appear as file chips above the text.
- **Assistant message:** no bubble, plain text on `canvas`, with the 24px cairn mark in the left gutter on the first paragraph only.
- **Markdown:** headings, lists, tables (scroll horizontally inside their own container), blockquotes, inline code (`xs` radius, `surface-sunken`), fenced code blocks (`md` radius, `surface-sunken`, language label in `label` style, copy button top right).
- **Streaming:** text appends incrementally with a soft caret. The Send button becomes **Stop**. Announce completion, not every token, to screen readers (`aria-live="polite"`, throttled).
- **Message footer** in `label` style, `ink-muted`, shown on hover or focus: copy, regenerate, and the model line. The model line shows the model that responded. Include the provider name. When the provider reports something different from what was requested (for example a 9Router combo), show both: `requested: combo-name, answered by: model-id`. Show only what the provider actually reports.

### 8.3 Retrieval citations

- Answers that used documents show numbered **citation chips** (`sand` tint, `sm` radius, `label` text) inline.
- Clicking a chip opens a popover with the file name, location (page or section) and the retrieved passage. The passage sits in a quoted block labelled **Untrusted document content** with a shield icon, because retrieved text must never be presented as instructions.

### 8.4 Tool calls (MCP)

- Each tool call is a card between message paragraphs: `lg` radius, `surface` fill, hairline border.
- Header: tool name in `label` style, server name in `ink-muted`, status badge (running, succeeded, failed).
- Body is collapsed by default. Expanded, it shows arguments and result as formatted JSON in a code block. Tool output is labelled **Untrusted tool output**.
- **Permission request** (inline, before execution): tool name, server, arguments, and three buttons: **Allow once**, **Allow for this session**, **Deny**. Execution does not start until the person chooses.

### 8.5 Error and edge states

Each is a distinct inline banner with an icon, a plain message, and one action. Never a generic "Something went wrong".

| State | Message intent | Action |
|---|---|---|
| Provider unreachable | The provider isn't responding at the configured address | Open Settings, Retry |
| API key rejected | The provider refused the key | Open Settings, Providers |
| Model unavailable | The selected model was not found on that provider | Change model |
| No provider configured | There is nothing to send the message to | Add provider |
| 9Router has nothing to route to | 9Router is running but has no upstream provider set up | Open the 9Router dashboard |
| Rate limited or exhausted | The upstream provider hit a limit | Retry, Change model |
| Stream interrupted | The response stopped early | Continue, Regenerate |
| File failed to process | Name and reason | Retry, Remove |
| Skill too large | The skill body is over the token limit | Edit skill |
| Context nearly full | Older messages will be trimmed | Start new session |

## 9. Composer

The single most important component. It is the hero on Landing and a docked bar in Conversation.

- **Container:** `surface` fill, 1px `border-strong`, `xl` radius (16px), elevation 3 on Landing, elevation 2 when docked.
- **Textarea:** auto-grows from 2 lines to a maximum of 10, then scrolls. Placeholder "Ask anything" in `ink-muted`.
- **Attachment and skill row** (above the textarea, only when files are attached or skills are active): file chips with type icon, name, and a status that moves **uploading, processing, ready**, or **failed**, and skill chips (`mist` fill, `sm` radius, skill name in `label` style). Each chip has a remove button.
- **Skill picker:** typing `/` at the start of the textarea opens a popover listing enabled skills: name, one-line description and `~N tokens` in `label` style. Arrow keys move, Enter activates, Esc closes. The picker filters as you type.
- **Toolbar** (below the textarea, left to right):
  - **Attach** icon button (paperclip). Opens the file picker. The whole window also accepts drag-and-drop, showing a dashed `accent` drop overlay.
  - **Tools** icon button with a count of enabled tools. Opens the Tools tab of the context drawer.
  - **Model picker**: a `md`-radius secondary button showing the current model in `label` style with a chevron.
  - Spacer.
  - **Send** button: 40px square, `md` radius, primary style, arrow-up icon. Disabled when empty. Becomes **Stop** (square icon) while streaming.
- **Keyboard:** Enter sends, Shift+Enter inserts a newline, Esc stops a running response, `Cmd+/` or `Ctrl+/` toggles the context drawer.
- **Focus:** the container shows a 2px `accent` ring with 2px offset when the textarea is focused.
- **Model picker popover:** searchable list, **grouped by provider**, covering every enabled provider. Each row shows the model or combo ID in `label` style, and capability badges (tools, vision, context size) **only when the provider or the app's metadata layer actually knows them**. Unknown capabilities show nothing, or a muted "unknown", and are never assumed. At the bottom: **Add model ID** (for providers that do not list a model) and **Manage providers**, which opens Settings. The model can be changed at any time, including in the middle of a session, and the change applies to the next message.
- **No provider configured:** Send is disabled and a banner above the composer reads "Add a provider to start chatting" with an **Add provider** button.
- **Context meter:** a thin 2px bar along the bottom edge of the composer, `accent` fill, showing estimated context usage. It turns `warning` above 80%. The token estimate is shown on hover in `label` style. The bar is a rectangle, not a rounded track.

## 10. Context drawer

Right-hand panel, 360px, `surface` fill, `xl` radius on its leading corners, elevation 3, opened from the topbar or composer. Five tabs in a segmented control (`sm` radius segments) at the top: Files, Retrieval, Tools, Skills, Model. Each segment has an icon and a label. If the labels do not fit at 360px, show icons with tooltips and the label for the active tab only. Content is per session.

| Tab | Contents |
|---|---|
| **Files** | Files in this session with type icon, size, and processing status. Per-file switch **Use in answers** (rounded-rectangle track, `sm` radius). **Add files** button. **Import from another session** is an explicit action that opens a picker, because files never cross sessions automatically. |
| **Retrieval** | Mode control: **Off**, **Selected files**, **Auto**. Below it, the passages retrieved for the last answer, each with file, location and relevance shown as a small rectangular bar rather than a raw number. |
| **Tools** | MCP servers as rows: name, transport, status dot with text. Expanding a server lists its discovered tools with a per-tool permission control (**Ask**, **Allow**, **Deny**). **Add server** opens a small dialog. **Remove** asks for confirmation. |
| **Skills** | Skills as rows: name, one-line description, and two token estimates in `label` style (`~N tokens` in the index, `~N tokens` when active). Each row has an enable switch, an **Activate for this session** control, **Edit** and **Delete**. Active skills are listed first. At the bottom, a `label` line shows the total tokens used by the index and by active skills. **New skill** and **Import skill** open dialogs. Inline warnings appear for descriptions or bodies over the limits. |
| **Model** | Session model (provider and model), and default model. Generation parameters (temperature and similar) only for the parameters the selected provider accepts. Context window meter with the estimate. A **Compact session** button appears when the context is above about 60% full. It shows the estimated tokens before and after and offers **Undo**. |

### Skill dialogs

- **Skill editor:** a dialog (`lg` radius, 640px). Fields: name, description with a character counter (about 200 max), and the instruction body in a monospace text area with a live token estimate. **Save** and **Cancel**.
- **Import skill:** opens a file picker for a skill folder, then shows the **full content** for review under the label **Untrusted until reviewed**. The skill stays disabled until I press **Enable**.
- Skills never appear as their own page.

## 11. Settings dialog

A centered dialog (`lg` radius, 560px), not a page. Opened from the sidebar footer or an error banner.

- **Providers:** a list of configured providers. Each row shows name, adapter type, base URL, a status dot with text, an enable switch, and Edit and Remove actions. One row carries a **Default** badge. **Add provider** opens a dialog with: a preset selector (9Router, OpenAI-compatible, Anthropic, Google Gemini) that only prefills the form, a name, the base URL, the API key (masked, **write-only**: once saved it is never displayed again, only "Key saved"), optional extra headers under an "Advanced" section, a **Test connection** button with an explicit result, and **Fetch models**. Each provider also offers **Add model ID** for models it does not list. Keys are never shown, exported or logged.
- **Default model.**
- **Default skills:** the skills that start active in every new session (none by default).
- **Appearance:** Light, Dark, System.
- **Data:** shows the local workspace location, **Export workspace** and **Open data folder** actions. One line explaining that nothing leaves this machine except requests to the providers you configured. Links to **Privacy** and **Terms**.

## 12. Components

### 12.1 Buttons

All buttons use the `md` radius (8px). No button is a pill or a circle.

| Variant | Style |
|---|---|
| Primary | `ink` fill, `canvas` text (both themes), 40px high, 16px horizontal padding. Hover: 8% lighter. Pressed: 8% darker. |
| Secondary | `surface` fill, 1px `border-strong`, `ink` text. Hover: `surface-hover`. |
| Ghost | Transparent, `ink-secondary` text. Hover: `surface-hover`. |
| Icon | 36px square visual size with a 40px minimum hit area on touch, 18px Lucide icon. |
| Danger | `danger` fill, white text, used only in delete confirmations. |

All buttons have visible focus (2px `accent` ring, 2px offset), a disabled state at 45% opacity, and a loading state that keeps width stable. Sizes: 40px default, 32px compact.

### 12.2 Chips and badges

`sm` radius (6px), 28px high, `label` text. Variants: neutral (`surface-sunken`), suggestion (`mist`), file (`sand` tint), status (semantic tint with icon), "Soon" (outlined, `ink-muted`).

### 12.3 Inputs

`surface` fill, 1px `border-strong`, `md` radius, 40px high. Focus: 2px `accent` ring. Error: `danger` border plus message below in `danger` text with an icon.

### 12.4 Dialogs, popovers, toasts

Dialogs trap focus, close on Esc, return focus to the trigger. Popovers close on outside click and Esc. Toasts appear bottom center, auto-dismiss after 5s, never for errors that need a decision (those are banners).

### 12.5 Icons

Lucide, 1.5px stroke, 18px default and 20px in the composer. Bundled, no icon CDN. No emoji, no icon fonts made of emoji, no decorative gradient icons.

### 12.6 Brand mark and favicon

A small **cairn**: three stacked rounded stones of decreasing width, slightly offset, the top stone in `accent` and the lower two in `ink`. Flat fills, no gradient. Draw it as a simple original vector. Wordmark is "Cairn" in Fraunces 400 followed by "AI" in Geist 500, same baseline, no effects. Do not reuse or imitate any Venice or Captions logo.

The favicon is this mark (section 17 lists the required files).

## 13. Motion

| Purpose | Duration | Easing |
|---|---|---|
| Hover, press, focus | 120ms | `cubic-bezier(0.2, 0, 0, 1)` |
| Popover, dropdown | 160ms | `cubic-bezier(0.2, 0, 0, 1)` |
| Drawer, sheet, dialog | 240ms | `cubic-bezier(0.2, 0, 0, 1)` |
| Compose-to-dock (Landing to Conversation) | 240ms | `cubic-bezier(0.2, 0, 0, 1)` |

- Compose-to-dock: the composer moves from the center to the bottom while the hero fades out. One short, continuous movement.
- Streaming caret pulses gently at 1s intervals.
- Use `transform` and `opacity` only. No layout-shifting animation.
- **No scroll-driven animation of any kind.** No parallax, no scroll-jacking, no sticky scrub sequences, no reveal-on-scroll, no counters that count up. Content is visible on load and scrolls natively.
- **No cursor effects.** No custom cursor, follower, trail, magnetic hover or pointer-tracking glow. The system cursor is always used.
- **Perceived speed.** No animation may delay or block input. The user message appears instantly on send and the assistant placeholder right after. No artificial delays, no full-page loaders. Use a skeleton only for a list that takes longer than 200ms to load. Stopping a response takes effect within one frame.
- **Respect `prefers-reduced-motion`:** replace movement with instant or opacity-only changes and stop the caret pulse.

## 14. Accessibility

- WCAG AA contrast, verified with a linter on the final build.
- Every interactive element is keyboard reachable in a logical order. The focus ring is always visible.
- Touch targets are at least 40px.
- Sidebar drawer, context drawer and dialogs trap focus and restore it on close.
- Status is never color-only.
- The message list is a labelled region. New assistant content is announced politely and in batches.
- Landmark structure: `nav` (sessions), `main` (canvas), `complementary` (context drawer). Legal pages use `header`, `main`, `footer`.

### Shortcuts

| Action | Keys |
|---|---|
| New session | `Cmd+Shift+O` or `Ctrl+Shift+O` |
| Search sessions | `Cmd+K` or `Ctrl+K` |
| Toggle sidebar | `Cmd+B` or `Ctrl+B` |
| Toggle context drawer | `Cmd+/` or `Ctrl+/` |
| Send | `Enter` |
| Newline | `Shift+Enter` |
| Stop response or close layer | `Esc` |

## 15. Do and don't

**Do**
- Keep the product on one application page. Use drawers, popovers and dialogs.
- Let the composer be the visual center of the Landing state.
- Use `accent` sparingly, and `accent-strong` for text.
- Show real provider state: real models, real status, real errors.
- Label retrieved documents and tool output as untrusted.
- Make session switching obvious and safe.
- Write specific copy that states what the product does.
- Self-host every font and icon.

**Don't**
- Don't add sign-in, sign-up, account, pricing or upgrade UI.
- Don't add routes beyond `/`, `/privacy` and `/terms`.
- Don't use pill or circular buttons, chips or inputs.
- Don't use purple, or any gradient outside section 2.4.
- Don't add testimonials, reviews, counters, star counts or "trusted by" rows.
- Don't use emoji, stock photos, generated images or 3D blobs.
- Don't use em dashes.
- Don't animate on scroll or react to the cursor.
- Don't ship a builder or "Made with" badge.
- Don't use Venice's or Captions' logos, wordmarks, fonts or exact palettes.
- Don't hardcode provider logos, provider counts or model lists.
- Don't use white text on `accent` for body-size text.
- Don't show a marketing claim for a feature that is not implemented.

## 16. Implementation tokens

```css
:root {
  color-scheme: light;

  /* color */
  --canvas: #F5F3EC;
  --surface: #FFFFFF;
  --surface-sunken: #EDF2F3;
  --surface-hover: #E3EBEE;
  --border: #D9E2E6;
  --border-strong: #B9C6CD;
  --ink: #0F2438;
  --ink-secondary: #3E5163;
  --ink-muted: #5B6B78;
  --accent: #3A86C8;
  --accent-strong: #1F5F99;
  --accent-tint: #E1EEF9;
  --tide: #BFE6F0;
  --mist: #E6F5F8;
  --sand: #D8C9AE;
  --slate: #6B7178;
  --success: #2E9E6B;   --success-tint: #DDF3E8;
  --warning: #C77A12;   --warning-tint: #FBEBD0;  --warning-ink: #7A4A06;
  --danger: #C93C33;    --danger-tint: #FBE1DE;   --danger-ink: #9E2A23;

  /* gradients (the only three that exist) */
  --grad-hero: linear-gradient(180deg, #E6F5F8 0%, #F5F3EC 68%);
  --grad-fade: linear-gradient(180deg, rgba(245, 243, 236, 0) 0%, #F5F3EC 100%);

  /* type */
  --font-display: "Fraunces", "Iowan Old Style", "Times New Roman", serif;
  --font-sans: "Geist", ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-mono: "Geist Mono", ui-monospace, "SF Mono", Menlo, Consolas, monospace;

  /* shape (no pill radius exists) */
  --radius-xs: 4px;  --radius-sm: 6px;  --radius-md: 8px;
  --radius-lg: 12px; --radius-xl: 16px;

  /* elevation */
  --shadow-1: 0 1px 2px rgba(15, 36, 56, 0.06);
  --shadow-2: 0 4px 16px rgba(15, 36, 56, 0.08);
  --shadow-3: 0 8px 32px rgba(15, 36, 56, 0.10);

  /* layout */
  --sidebar-w: 288px;  --sidebar-rail-w: 64px;  --topbar-h: 56px;
  --drawer-w: 360px;   --reading-w: 720px;      --composer-w: 760px;
  --landing-w: 1080px; --section-gap: 120px;

  /* motion */
  --ease: cubic-bezier(0.2, 0, 0, 1);
  --dur-fast: 120ms; --dur-base: 160ms; --dur-panel: 240ms; --dur-dock: 240ms;
}

:root[data-theme="dark"] {
  color-scheme: dark;

  --canvas: #0B141C;
  --surface: #121F2A;
  --surface-sunken: #08111A;
  --surface-hover: #182835;
  --border: #22333F;
  --border-strong: #33495A;
  --ink: #EEF0E8;
  --ink-secondary: #BAC5CD;
  --ink-muted: #8C9BA7;
  --accent: #5AA6E8;
  --accent-strong: #8CC1F0;
  --accent-tint: #142B3F;
  --tide: #173847;
  --mist: #10222E;
  --sand: #B9A98F;
  --slate: #8A9096;
  --success: #4CC38A;   --success-tint: #12291F;
  --warning: #E8A73A;   --warning-tint: #33240C;  --warning-ink: #F2C67A;
  --danger: #F0776F;    --danger-tint: #331715;   --danger-ink: #F5A29C;

  --grad-hero: radial-gradient(90% 60% at 50% -10%, #16303F 0%, #0B141C 70%);
  --grad-fade: linear-gradient(180deg, rgba(11, 20, 28, 0) 0%, #0B141C 100%);

  --shadow-1: 0 1px 2px rgba(0, 0, 0, 0.4);
  --shadow-2: 0 4px 16px rgba(0, 0, 0, 0.45);
  --shadow-3: 0 8px 32px rgba(0, 0, 0, 0.5);
}

@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

**Applying the theme before first paint.** Theme is driven only by the `data-theme` attribute, so there is a single set of dark tokens. Set the attribute with a tiny inline script in `<head>` so the page never flashes the wrong theme. "System" means no stored choice.

```html
<script>
  (function () {
    var t = null;
    try { t = localStorage.getItem("cairnai-theme"); } catch (e) {}
    if (t !== "light" && t !== "dark") {
      t = window.matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light";
    }
    document.documentElement.setAttribute("data-theme", t);
  })();
</script>
```

The stored preference may live in the app's local settings store instead of `localStorage`, as long as it is read before first paint.

## 17. Public launch gate and legal pages

**We do not launch a public site until every item below is done.** This applies to any publicly hosted copy of CairnAI, such as a portfolio demo or project page. Record the status of each item in `PROGRESS.md`.

### 17.1 Checklist

| Item | Requirement |
|---|---|
| Custom domain | Connected, served over HTTPS, one canonical hostname. The host's default subdomain redirects to it. Domain availability has not been verified. |
| Favicon | Designed from the cairn mark, not a framework default. Provide an SVG that adapts to light and dark with `prefers-color-scheme`, a 32px PNG or ICO fallback, and a 180px Apple touch icon. Confirm the Vite, Next or other template icon is gone. |
| Builder tags removed | No "Made with AI", "Built with", "Edit with" or similar tag, banner or watermark on any page, in the page source, or injected by a script. Check the built output, not only the source. |
| Privacy policy page | Live at `/privacy`, linked from the footer strip and the Settings dialog. |
| Terms and conditions page | Live at `/terms`, linked from the footer strip and the Settings dialog. |
| Page metadata | Real `<title>` and description. The social preview image is a real screenshot of the app, never a generated image. |
| No placeholder text | No lorem ipsum, no bracketed placeholders such as "[Company Name]" or "[Date]", no unfinished sections. |
| Content check | Section 0.2 and 0.3 checked against every page: no purple gradient, no pill shapes, no fake reviews or counters, no emoji icons, no em dashes, no scroll or cursor effects, no generated photos. |

### 17.2 Legal page design

The two pages share one layout.

- **No sessions sidebar.** A minimal header with the brand at left and a link back to the application at right. A simple footer with Repository, Privacy and Terms links.
- Single reading column, 720px max, centered, on `canvas`. No hero gradient.
- Title in `h1` (Fraunces). Below it, "Last updated" in `label` style with a **real date** set at publication.
- Section headings in `h2` or `h3`. Body in `body`. Lists and tables use the same styles as chat markdown. For pages longer than a few screens, add a short plain list of anchor links at the top.
- Print friendly: no shadows, ink on white, links show their URL.
- Both pages follow the copy rules in section 0.3.

### 17.3 Legal page content rules

The project owner writes and owns this text. It must be **accurate to the software as shipped**.

- Describe what CairnAI actually does. Sessions and files are stored locally on the person's machine. Requests go to the provider the person configures (for example a 9Router instance, or a provider's own API using the person's key), and those providers handle that data under their own terms.
- Only state "no account", "no analytics" or "no telemetry" if that is true of the build being published.
- Do not paste generic boilerplate that mentions accounts, cookies, payments, analytics, advertising or data collection that CairnAI does not have.
- The terms should name the open-source license, include a no-warranty and limitation-of-liability statement, and give a real contact method such as the repository issue tracker or a real email address.
- Have the final text reviewed by a qualified person before launch. This design file does not provide legal advice.

## 18. Notes for the agent

- **Adapt, don't copy.** Both references informed mood and structure. Neither brand's fonts, logos, palette values or pill shapes appear here, and none should be added.
- If a front-end framework with a token system is used (for example Tailwind), map these CSS variables into its theme instead of duplicating raw values. Do not add a full-radius utility to the theme.
- If any decision here conflicts with the project spec, document the conflict in `docs/09-UI-ARCHITECTURE.md` and `docs/03-TECHNICAL-DECISIONS.md` rather than silently overriding either. Known refinements of the spec's conceptual sketch: the right-hand Context area is an **on-demand drawer**, not a permanent third column, and two static legal routes exist beside the single application page.
- When unsure whether something is allowed, check section 0.2. If it is listed there, it is not allowed.
- Assumptions to revisit: the Fraunces and Geist pairing (chosen for licensing, not because it matches the references exactly), the suggested Landing copy, and the cairn mark (a simple original placeholder, to be redrawn if desired).

---

*Inspired by the Captions design system (Split Studio, light, central input) and the Venice brand kit (deep blue foundations, warm paper neutrals, serif display with plain sans). All tokens, copy and marks in this file are original to CairnAI.*
