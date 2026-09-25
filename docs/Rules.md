# Rules

## 1. Core Principle
Work in modular decomposition only. This is mandatory, not a preference.

## 2. Stack and Dependencies
- Use only the stack chosen in docs/03-TECHNICAL-DECISIONS.md
- Adding a dependency needs a recorded decision
- Prefer fewer dependencies
- A dependency must be actively maintained and license-compatible with open sourcing
- Use Bun for all package management operations
- Run `bun run typecheck` after every change

### Libraries to Use
- Next.js 16 (App Router)
- React 19
- TypeScript
- Tailwind CSS
- shadcn/ui
- Lucide React (icons)
- Fraunces, Geist, Geist Mono (self-hosted fonts)
- react-hook-form + zod
- Bun

### Libraries to Avoid
- No purple gradients or any gradient outside DESIGN.md section 2.4
- No pill-shaped buttons, chips, badges, inputs, tabs, or switches
- No emoji as icons
- No icon fonts made of emoji
- No fake social proof (testimonials, reviews, star ratings, "trusted by")
- No fake metrics (user counts, download counts, speed claims, benchmark claims)
- No AI slop photos, generated images, 3D blobs, abstract AI art
- No over-the-top scroll animation (parallax, scroll-jacking, reveal-on-scroll)
- No cursor animation (custom cursors, followers, magnetic buttons, spotlight/glow effects)
- No builder badges ("Made with AI", "Built with", etc.)
- No external requests for design assets (no CDN fonts, no analytics, no remote images)
- No purple, violet, indigo-to-pink, magenta or neon in any gradient or glow
- No generic gradient styling (mesh gradients, aurora effects, blurred color blobs, glow orbs)
- No white text on accent for body-size text

## 3. Error Handling
- Use the normalized provider error states
- Never swallow an error
- Never show a generic "something went wrong"
- Every user-visible error has a plain message and an action
- Never leak keys or file contents into logs or messages
- Error states: unreachable, unauthorized, rate limited, quota exhausted, model not found, malformed stream, timeout
- Each error state has a distinct banner with icon, plain message, and action

## 4. Security
- Keys stay in the backend
- The frontend never calls a provider
- Uploaded documents and tool output are untrusted
- Validate files and paths
- No secrets in commits
- API keys never visible in UI after initial entry
- Never log API keys
- Never expose sensitive file contents
- No path traversal
- No redirect attacks with auth headers
- Skills cannot change tool permissions or provider settings

## 5. Performance
- The hot-path budgets in section 27 of SPEC.txt
- No buffering on the stream path
- No blocking work on the send path
- App-added latency must be under all budgets
- No artificial delays
- Streaming must be real (not faked)
- Context building is incremental and cached
- Retrieval runs in parallel with other request preparation
- File parsing, chunking, embedding run in background workers

## 6. Design and Copy
- Follow design.md exactly (section 0.2 bans apply to all UI, docs, and README text)
- No purple gradients, no pill shapes, no fake metrics, no emoji icons, no em dashes, and the rest of that list
- Self-host every font and icon
- Copy is plain and specific
- Say what a thing is and what it does
- Every claim needs a mechanism in the product
- Present tense, sentence case, no exclamation marks
- No em dashes (use commas, colons, periods, parentheses)
- All numbers shown in interface come from real data
- Example content inside feature illustrations is labeled "Example" and contains no invented statistics
- Banned words: supercharge, unleash, unlock (as a metaphor), elevate, seamless, effortless, revolutionary, next-generation, cutting-edge, game-changing, empower, leverage, harness, delve, tapestry, journey, reimagine, "the future of", "AI-powered", "your AI companion", "magic"

## 7. Scope
- Build only the current phase
- Do not build deferred features
- The only routes are /, /privacy and /terms
- No sign-in, sign-up, account menu, avatar, pricing, billing or upgrade prompts
- No other pages beyond /, /privacy and /terms
- Sessions, streaming chat, file upload, document retrieval, MCP tools, skills, model selection, provider settings are all reachable from the application page
- Everything the product does is reachable from the application page

## 8. Verification
- Do not claim a task is done without running the tests
- Do not claim provider support without a test
- Do not invent API behaviour, check the official documentation
- Run `bun run typecheck` after every change
- Do not run build unless explicitly requested
- Update PROGRESS.md at the end of every session
- Update MEMORY.md at the end of every session

## 9. Modules
- Work in modular decomposition only (section 1A)
- One module at a time
- Define the interface first
- Talk to other modules only through their public interfaces
- Keep files small and single-purpose
- Give each module its own README and tests
- No circular dependencies
- Modules talk only through public interfaces
- No reaching into another module's internals
- If a task cannot be split into module-sized pieces, stop and split it first
- Size guideline: a slice is about 400 changed lines or fewer, excluding tests and generated files

## 10. Working Style
- Read MEMORY.md first
- Do not re-read the whole codebase
- Make the smallest change that solves the problem
- Do not touch unrelated files
- Ask before destructive actions (deleting data, force pushes, mass rewrites)
- Do not edit design.md without asking
- Update the docs together with the code
- Update PROGRESS.md at the end of every session
- Create MEMORY.md at the moment implementation begins (right after Phase 0 approval)

## 11. Documentation
- Markdown is the canonical engineering-document format
- Create documentation directly inside the repository (docs/)
- Keep documentation synchronized with implementation
- Do not silently override design.md
- If requirements conflict, identify and document the conflict and make a reasoned engineering decision

## 12. Data Handling
- Sessions and files stored locally on the person's machine
- Requests go to the provider the person configures (for example a 9Router instance, or a provider's own API using the person's key)
- Those providers handle that data under their own terms
- Only state "no account", "no analytics" or "no telemetry" if that is true of the build being published
- Do not paste generic boilerplate that mentions accounts, cookies, payments, analytics, advertising or data collection that CairnAI does not have
- The terms should name the open-source license, include a no-warranty and limitation-of-liability statement, and give a real contact method
- Have the final text reviewed by a qualified person before launch

## 13. Launch Gate
Do not launch a public site until every item in DESIGN.md section 17 is done. Record status of each item in PROGRESS.md.

## 14. Portfolio Objective
The project should be capable of truthfully being presented as:
"Built CairnAI, a personal AI workspace with multi-model routing, persistent conversations and sessions, document RAG, streaming inference, file interaction, and MCP-based tool integration."

## 15. Attribution
Provider-level routing and fallback is done by 9Router. What this project builds is the gateway abstraction, model discovery and selection, sessions, RAG, file handling and MCP integration around it.

## 16. License
The project license is MIT.