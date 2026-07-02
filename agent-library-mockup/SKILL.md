---
name: agent-library-mockup
description: >-
  Build an AI-agent chat-interface mockup in Figma from a topic, using Appier's real
  "Agent | Library" components (Chatroom, Response/Turn, Response/AI message,
  Response/User message, Chat history). Given a subject/scenario, generate a multi-turn
  human↔AI conversation and assemble it into a Figma mockup out of real library component
  instances with bound variant properties — not redrawn shapes. Use this whenever the user
  wants to mock up, prototype, or visualize an Appier AI agent / assistant chat experience,
  a conversation flow, a chat panel or full-page chat, or "用 agent library 組對話", "做一個 AI
  對話 mockup", "組一串人機對話畫面", even if they only give a topic and don't name the components.
  HARD REQUIREMENT: never create nodes until the user has given and confirmed a target Figma
  file + frame/location. NOT for: implementing the chat as React/production code (that is a
  code task), generic non-Appier chat UIs, or reading an existing Figma design into code.
---

# Agent Library Mockup

Assemble an Appier **AI-agent chat interface** in Figma from a topic. The job has two halves:
**(1) write a believable multi-turn human↔AI conversation** for the given subject, and
**(2) build it in Figma using the real `Agent | Library` components**, setting the right
variant properties so the result is a clean, editable mockup — not a pile of redrawn rectangles.

The full component spec (every property, value, and composition rule) lives in
[references/agent-library.md](references/agent-library.md). Read it before building — it is the
source of truth for what each component is and how the pieces fit. This SKILL.md is the workflow.

## Prerequisite: this is a Figma write task

Building places and edits nodes in Figma via the `use_figma` tool. Before the **first**
`use_figma` call you MUST load the **`figma-use`** skill — skipping it causes hard-to-debug
failures. When assembling multi-section views, the **`figma-generate-design`** skill's
incremental section-by-section approach pairs well with this one. Load both alongside this skill.

## Step 0 — Confirm the destination (hard gate, do this first)

Do not create, import, or move any node until the user has given **and confirmed** where the
mockup goes. Mockups must never land in a random or guessed location.

Ask for, then read back to confirm:
- The target **Figma file** (URL or fileKey).
- The exact **frame / page / location** within it (a node-id, a named frame, or "make a new
  frame named X here"). If ambiguous, propose a specific spot and ask for a yes.

If the user hasn't supplied a destination, stop and ask. Proceed only on an explicit confirmation.

## Step 1 — Scope the scenario

From the user's topic, settle these before writing anything. Offer sensible defaults and only
ask about what genuinely changes the build:

- **Subject / goal** of the conversation (e.g. "618 campaign marketing journey", "compare AOV of
  Line OA members vs guests").
- **Layout**: `Full-page` (immersive, content centered ~960) or `Side-panel` (narrow ~500 panel
  docked beside a feature page). This drives three coupled switches — see the layout table below.
- **Turns**: how many user↔AI rounds (default 2–3). One round → Chat thread `Type=Short`;
  multiple → `Type=Long`.
- **History rail**: show the `Chat history` (full-page = 300px second sidebar; side-panel =
  replaces the panel) or not.
- **Language** of the conversation copy (mirror the user's; default to theirs).
- **Richness**: does any AI turn carry a **widget** (chart card / table) or just text + quick replies?

## Step 2 — Write the conversation, then confirm it

Generate the full multi-turn script as plain text and show it to the user for sign-off **before**
touching Figma. Writing copy is cheap to revise; rebuilding Figma nodes is not.

Make it believable for the subject, and follow the **AI response anatomy** (this is how real
Agent responses are structured — see the reference for the worked example):

1. **Thinking** — a short "Show thinking" step label.
2. **Session 1 · Key Findings / Executive Summary** — the direct answer: narrative insight or an
   action summary, with a few bolded numbers or bullet highlights.
3. **Session 2 · Evidence / Data Visualization** — *only when it earns its place*: a chart card or
   structured table that backs the finding. Mark clearly where a widget goes.
4. **Guiding question + Quick Reply** — one forward-looking question ("Would you like to compare
   more?") plus 2–3 quick-reply chips.
5. **Reactions / Credit** — note whether the turn consumes credit.

Conversation rules that the build must respect:
- **Quick replies belong only to the latest AI turn.** Tapping one = sending that text as the next
  user message → a new turn. So earlier turns must NOT show quick replies.
- User turns may carry reference chips, target chips, or uploaded images when the scenario calls
  for it; otherwise keep them clean.

## Step 3 — Build in Figma from real library instances

Load `figma-use` first. The goal is a mockup made of **real library instances** (so variants and
tokens come for free), not redrawn shapes.

### Primary strategy: import a layout template → detach → retext (Path C)

The 4 canonical layouts are published as a component set (`[test]Template_agent_pages`), each variant
a complete page **including the app chrome** (nav bar + sidebar, and for side-panel the breadcrumb/
Cancel bar). This makes cross-file reuse trivial: import the right variant, **detach it**, and you get
a normal frame with every nested instance preserved (Chatroom, Response/Turn, chrome) — then retext.
Detach was verified to keep all nested instances intact. Keys are in
[references/agent-library.md](references/agent-library.md) §五.

```js
const dest = await figma.getNodeByIdAsync(DEST_NODE_ID);   // confirmed destination (page or frame)
const TEMPLATE_KEY = "f860d871f076be9e0bd4bbc4df2d7816020f19c5"; // full-page chat (see §五 for all 4)
const inst = (await figma.importComponentByKeyAsync(TEMPLATE_KEY)).createInstance();
dest.appendChild(inst);
inst.x = 0; inst.y = 0;
const page = inst.detachInstance();   // → FRAME; nested Chatroom/turns/chrome stay as instances
return { frameId: page.id };           // stable top-level anchor for all later edits
```

Then, anchored on that returned frame id:
1. **Adjust the turn count.** The template ships with 1 `Response/Turn`. For N turns, find it and
   `slot.appendChild(turn.clone())` N−1 times (same file now, so clone works). For fewer, remove extras.
2. **Set Chat thread `Type=Long`** for multi-turn (bottom-aligns so the latest turn sits above the input).
3. **Retext each turn** (rules below): user bubble, AI `Response content`, thinking steps; add a widget
   placeholder where Session 2 calls for one.
4. **Enforce the quick-reply rule:** hide `_Response/element/Quick response` on every turn **except the
   last**. Hide user-message `_*Chat context/Reference` / `…/Target` chips unless the scenario uses them.

**Requirements & gotchas:** the target file must have the `Agent | Library` enabled (else
`importComponentByKeyAsync` throws — ask the user to enable it). `figma.loadAllPagesAsync()` is NOT
available here; `getNodeByIdAsync`/`appendChild`/`clone`/`detachInstance` all work fine.

### Fallback: assemble from individual component keys (Path B)

Use only if the template set isn't available (not published / different library state), or you need a
bare Chatroom without chrome. Import `Chatroom` + `Response/Turn` by key, fill the (empty) `Chat thread`
slot, then add chrome (`*Navi-top bar`, `*sidebar 3/ primary sidebar`, side-panel adds `Viewer editing`)
— all keys + the full-page layout geometry (1512×915 frame, nav 1512×56 @0,0, sidebar 56-wide @0,56,
Chatroom inset to 56,56) are in [references/agent-library.md](references/agent-library.md) §五. Note a
freshly imported Chatroom defaults to side-panel (width 500, `Header` on, empty slot); for full-page set
`Header=false`, input `Type=Full page`, each Turn `Size=Full chat`. Then retext as above.

### Critical: how to address nodes inside instances (this is the #1 time-sink if ignored)

Deep instance-child ids (the `I…;…;…` form) are **not reliably resolvable** — `getNodeByIdAsync`
on them often returns `null`, and a `findOne`/`findAll` *predicate* can crash mid-traversal reading
a phantom node's `.name`. So:

- **Anchor on a stable top-level node** (the clone frame's simple id, e.g. `144:1023`) and
  `findAll` from there. Operate on the returned **node objects directly** — never round-trip a deep
  id back through `getNodeByIdAsync`.
- **Match targets by current content/name**, not by id: e.g. `if (n.characters.startsWith("Help me"))`
  for the user bubble, `=== "Action 1"` for a quick-reply label.
- **Beware smart quotes when matching default copy.** Freshly imported turns ship defaults like the
  user bubble `Text text text…` and the AI paragraph `I've created…` — that apostrophe is a curly `’`,
  so `startsWith("I've")` with a straight quote silently misses. Prefer matching by **structure** —
  the text node inside `Response/User message`'s bubble, and the one inside the AI message's
  `Response content` slot — or match loosely (e.g. on `Text text`). Verify with a screenshot after.
- **Guard every name/characters access in try/catch.** Use a `safe(n)` helper:
  `const safe = n => { try { return n.name } catch { return null } }`. A bare `findOne(n => n.name === …)`
  will crash the whole (atomic) script and roll back all your edits.
- **Distinguish duplicate turns by position:** sort instances by `absoluteTransform[1][2]` (abs Y) —
  topmost = Turn 1, bottom = latest.
- **Load fonts before any text edit** (`Inter` `Regular` and `Semi Bold`), then set `.characters`.

### Set variant / boolean properties via `setProperties`

Read the exact keys first from `instance.componentProperties`, then set them. Boolean keys carry a
`#id` suffix (e.g. `"Header#6120:1"`); variant keys are bare (`"Type"`, `"States"`).

- Layout switches must agree (full-page vs side-panel):

  | | Full-page | Side-panel |
  |---|---|---|
  | Chatroom `Header` | off | on |
  | Chat input field `Type` | Full page | Side panel |
  | Response/Turn `Size` | Full chat | Chat panel |
  | container width | 1456 (content ~960) | 500 |

- **Multi-turn → set Chat thread `Type=Long`** (bottom-aligns the thread so the latest turn sits
  just above the input). One turn → `Short` (top-aligned).
- **Input field `States=Enabled`** for the clean placeholder state (examples often ship as `Typing`).
- Set `Hint`, and input flags `Alert`/`Reference`/`Target`/`Uploaded files` only when a turn needs them.

### Widget placeholders

Chart cards / tables are not yet in this library. When Session 2 needs one, append a **clearly
labelled dashed placeholder card** into the AI message's `Response content` slot (a vertical
auto-layout: a "WIDGET PLACEHOLDER" tag, the chart title, and a grey chart area; set the card's
`layoutSizingHorizontal = "FILL"` after appending). Never fake a real chart — say it's a stand-in.

## Step 4 — Verify and iterate

Screenshot the result (`node.screenshot()` inline, or `get_screenshot`) and check against the
scenario: layout switches aligned, quick replies only on the last turn, alignment matches Chat
thread `Type`, copy reads naturally. Show the user and refine on request.

**Expected, not a bug:** the footer (input field) is a gradient overlay on top of the scrollable
thread. With `Type=Long` the latest turn bottom-aligns above the input and **overflow content
scrolls off the top**; the very bottom of the newest turn may tuck slightly under the input edge.
This mirrors real chat scrolling — do not try to "fix" it by shrinking copy or resizing the Chatroom.

## What good looks like

- Real, editable library instances with correct variant properties — not redrawn shapes.
- The three layout switches always agree (no full-page Turn inside a side-panel Chatroom).
- A coherent, on-topic conversation that follows the response anatomy and the quick-reply rule.
- Nothing created until the destination was confirmed.
