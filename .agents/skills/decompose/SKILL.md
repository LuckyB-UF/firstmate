---
name: decompose
description: Turn a sharpened spec into dependency-ordered, vertical-slice backlog items before any work is dispatched. Use when the captain invokes /decompose (e.g. "/decompose", "break this spec into tickets", "slice this up"); firstmate also runs it in the section-7 intake path once a spec exists and the effort is too large to brief as one task. Reads a grill spec at data/spec-<project>-<slug>.md and writes backlog items via tasks-axi, each an independently shippable vertical slice sized to one crewmate, with blocked-by edges wired between slices. It plans and never builds: no branch, no dispatch, no code.
user-invocable: true
metadata:
  internal: true
---

# decompose

Turn a sharpened spec into work small enough to dispatch: a set of dependency-ordered, independently shippable vertical slices, recorded as backlog items with their `blocked-by` edges already wired.
This skill is the back of firstmate's planning path.
`/grill` sharpens the idea into a spec; `/decompose` breaks that spec into the tickets the ordinary brief and dispatch path of AGENTS.md section 7 then picks up.

`/decompose` plans, it never builds.
It creates no branch, dispatches no crewmate, and writes no code.
Its only outputs are backlog items in `data/backlog.md` (through `tasks-axi`) and their per-slice descriptions.

## When this fires, and when it must not

Load this skill when the captain invokes `/decompose`.
Firstmate also runs it, without being asked, in the section-7 intake path once a spec exists and the effort is plainly too big to brief as one crewmate task.

Do not load it for:

- Work that fits one crewmate as-is.
  If you could write a single brief now, write the brief now; a one-slice decomposition is just overhead.
- A request with no spec behind it.
  Decomposition consumes a spec; if the shape is still fuzzy, that is `/grill`'s job first, or a scout's, not this one.
- A bug, a scout question, or a trivial change.
  Those route straight through section 7.

If a spec is thin or its open questions are unresolved, say so and stop rather than inventing slices to fill the gaps.
A slice built on a guessed answer reads as settled to the crewmate who picks it up, and that is how a wrong assumption gets shipped.

## What a vertical slice is

A vertical slice is the smallest change that delivers real end-to-end value and can ship on its own.
It cuts top to bottom through whatever layers it needs, not a horizontal band across one of them.
"Add the database column", "wire the API", "build the form" are horizontal layers: none is shippable alone, and splitting this way front-loads integration risk to the end.
"A user can save and see one draft" is a slice: thin, but it works and ships.

Two tests decide whether a slice is right:

- **Shippable alone.** Merged by itself, it leaves the project better and nothing half-built.
- **Sized to one crewmate.** One worker can carry it through the project's delivery path in one focused effort. If it is too big to hold, split it; if two slices are too entangled to ship apart, merge them.

## What it does

1. **Read the spec and confirm the subject.**
   Read `data/spec-<project>-<slug>.md` in full.
   The project is named in the spec; confirm it against the registry the way AGENTS.md section 7 resolves a subject, and read the project's code and README enough to slice against how it is actually built, not how the spec imagines it.
   If no spec file is named, ask which spec, or offer `/grill` first; do not decompose from memory of a conversation.

2. **Find the slices.**
   Work down from the spec's solution shape and in-scope list to the thinnest sequence of vertical slices that delivers it.
   Fold the spec's out-of-scope exclusions into what you leave out.
   Prefer more, smaller slices over fewer, larger ones: a slice that is too big to hold is the most common decomposition mistake, and the easiest to fix by cutting again.
   For a large, foggy program the spec may not decompose cleanly yet; carve out the slices that are genuinely ready, and leave the rest as spec open questions or a scout task rather than forcing fog into fake tickets.

3. **Order by real dependency.**
   A slice is blocked by another only when it genuinely cannot start until the first has landed - a shared contract, a schema, a seam the later slice builds on.
   Do not invent dependencies from mere theme; slices that only touch related code but do not need each other's output are independent and should dispatch in parallel.
   Coarse overlaps that would collide in the same subsystem are a serialization concern for dispatch (section 7), not a `blocked-by` edge here.

4. **Write each slice as a backlog item.**
   `data/backlog.md` and its `tasks-axi` backend are owned by AGENTS.md section 10; this skill adds no backlog syntax of its own and defers to `tasks-axi add --help` for exact flags.
   For each slice, add a `--kind ship` item tagged `--repo <project>`, queued (the default), carrying a per-slice description as its body (`--body` or `--body-file`) rich enough to become a crewmate brief later: what ships, the acceptance test, and the scope edge.
   The `--blocked-by <id>` mechanism is native and is how the edges are recorded, but a referenced id must already exist, so **add slices in dependency order, upstream first**, and wire each slice's `--blocked-by` to the ids of the slices it truly depends on.
   Give each slice a short, stable id so later slices can reference it; the id outlives this session, so keep it descriptive of the slice, not of its order.

5. **Report and hand off.**
   Summarize in plain outcome language (AGENTS.md section 9): the slices in dispatch order, which are independent and which wait on others, and anything the spec left open that a slice still needs answered.
   Name the next step - dispatching the ready slices per section 7 - and stop.
   Do not dispatch anything yourself.
   The queue is now the durable record; a held or open-question slice waits for the captain, not for you to force it forward.

## Boundaries

- **Never writes to a project.** AGENTS.md section 1 is unconditional; `/decompose` reads projects and writes only backlog items under firstmate's own `data/`.
- **Never dispatches.** Slicing and dispatching are separate steps on purpose, so the captain can review the slices before any crewmate starts.
- **Never owns the backlog contract.** It emits items through section 10's backend and stops; retention, dependencies, and lifecycle stay the backlog's job. If you find yourself restating how the backlog works, stop and point at section 10 instead.
