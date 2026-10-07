# Capabilities section ("One team · four disciplines") — full redesign plan

**Intent:** Redesign the *entire* Capabilities band on the echo11 homepage — every sub-component (kicker, claim heading, argument column, hairline, the four discipline rows), its layout, spacing, and its loading/entry choreography — while staying inside the site's house style and the section's typography-led art direction.

- **Target:** `http://localhost:3001/` → section `#capabilities`
- **Source of truth:** `/Users/devendrashahithakuri/Work/echo11/src/components/sections/Capabilities.tsx` (used by `src/app/page.tsx`, between `Results` and `FeaturedWork`)
- **Status:** superseded — v1 (this plan) and the v2 four-column index were both rejected; the shipped design is **v3 "ghost wall"** (compact claim+paragraph header, then four full-width outline-type titles that fill solid on hover with sibling spotlight dimming). Final proof: `browser/contact-GHOST-V5-4308717b6e91-1791410281.png`.

---

## 1. Current state (audit)

### What ships today
An interim pass from this session already landed in `Capabilities.tsx` (typecheck + eslint pass, **not yet visually verified**):

- Mono kicker `One team · four disciplines` with a small rule tick
- Claim heading split from the paragraph at `lg:` with an **accent full-stop** (`.` pops in cyan ~1.5 s after the word reveal)
- A **self-drawing hairline** (`scaleX 0→1`) between header and list
- Four rows on a `md:` 12-col grid (title 5 cols / body 6 cols from col 7), body measure capped at 60ch on mobile
- Row entrances: `opacity + y 24 + blur 6 → 0`, staggered 90 ms
- Hover: cyan rule sweeps under the title, scope ticks to accent, arrow steps out, row bg washes
- CTA hover now ticks to accent

### Problems observed in the "before" capture (`browser/browser-capture-localhost-2026-10-07T05-46-11-660Z.png`)
1. **Tablet stacking + dead space** — below `lg`, the header collapses to one column with a large empty band before the list; rows stack body text under titles at full width (~90ch measure).
2. **Flat row rhythm** — hierarchy relies only on color (80/55/40 % white); hover is easy to miss; no structural motion.
3. **Generic entry motion** — independent `whileInView` fades, no choreography spine; nothing reads as a "load-in" moment.
4. **Spacing inconsistency** — header→list gap differs across breakpoints; row padding steps (7/9/10) don't map to the section rhythm.

### Hard constraints (do not violate)
- **House motion split:** GSAP via `RevealText` owns word reveals (heading + row titles); framer-motion owns everything else. They never target the same nodes.
- **House easing:** `EASE_OUT = [0.16, 1, 0.3, 1]` from `src/lib/animations.ts` — no new easings.
- **Accent discipline:** single brand accent `#00e5ff` (`text-accent`); used sparsely (interactive feedback + the claim's full-stop).
- **Art direction for this band:** typography-led — no numerals, no rails, no rules *between* rows, no panels/cards.
- **Reduced motion:** every new animation must stand down (`initial={reduce ? false : …}` / CSS policy in `globals.css`).
- `npx tsc --noEmit` and `npx eslint src/components/sections/Capabilities.tsx` must stay green.

### Known open items (not caused by this work)
- `TODO(investigate)`: pre-existing dev overlay **"1 Issue"** badge — present in the user's capture from *before* this session's edit. Optional; capture console output if time allows.

---

## 2. Design spec — section anatomy

### S0 · Section shell
- `id="capabilities"`, `relative`, **spacing rhythm:** `py-24 md:py-36` (matches sibling sections — keep).
- Background: stays page background (no panel, no gradient). `TODO(user)` if a very subtle top/bottom fade is wanted — default: none.

### S1 · Kicker (eyebrow)
- House pattern (cf. `ClientLogos`): `font-mono text-[11px] uppercase tracking-[0.24em] text-white/40`, flex row with a `w-10 h-px bg-white/25` tick.
- Copy: `One team · four disciplines` (already in place — keep unless user rewrites).
- **Entry:** fade + y 10, 600 ms, first in the choreography.

### S2 · Claim heading
- **Type:** `font-sans font-bold leading-[1.04] tracking-[-0.04em] text-foreground`.
- **Scale:** `clamp(2.2rem, 5.2vw, 4.2rem)` now → **proposal:** bump to `clamp(2.4rem, 5.6vw, 4.6rem)` to match the `Statement` headline scale. `TODO(user)` — keep vs bump.
- Wrap: `max-w-[17ch]`, `lg:col-span-7`; rendered `inline` so the accent `.` flows as the final glyph.
- **Accent full-stop:** `text-accent`, pop-in `scale 0.3→1 + opacity`, 450 ms at delay **1.5 s** (lands as the last word of the RevealText reveal finishes). Screen readers get the full string via `ariaLabel` (with the period).

### S3 · Argument column (right)
- `lg:col-span-5 lg:justify-self-end`, aligned to the heading's baseline band (`items-end`).
- Paragraph: `max-w-[42ch] text-base/17px leading-[1.75] text-white/55`.
- CTA `See what each one includes` → `/services`: existing underline + arrow micro-move; hover ticks text/border to accent (already landed).
- **Entry:** fade + y 20, 700 ms, delay 0.1 s.

### S4 · Drawing hairline
- Single full-width 1 px line, `bg-white/[0.14]`, `origin-left scaleX 0→1`, 1100 ms, delay 0.15 s. Replaces the old static `border-t`.
- Spacing above: `mt-12 md:mt-16`. Below: rows' own padding provides the opening gap (`TODO(user)` if you want a bigger breath before row 1 — option: `mt-14 md:mt-20`).

### S5 · The four discipline rows (the core sub-components)
**Grid** (fixes the tablet problem):
| Breakpoint | Title | Body block |
|---|---|---|
| < 768 | full width, stacked | full width, `max-w-[60ch]` |
| md (768–1023) | `col-span-5` | `col-span-6 col-start-7` |
| lg (≥1024) | `col-span-6` | `col-span-5 col-start-8` |

**Vertical rhythm:** rows `py-7 md:py-9 lg:py-10` → **proposal:** uniform `py-8 md:py-10` so every row steps on one value. Internal: title → underline `mt-2`; body → scope `mt-2`; `gap-y-4` when stacked.

**Type hierarchy (unchanged scale, tightened roles):**
- Title: `clamp(1.85rem, 4.4vw, 3.4rem)` bold, `text-white/80 → white` on hover (GSAP RevealText per row, blur 3).
- Body: `15px/1.75` `text-white/55 → white/75` hover.
- Scope: `13px` `text-white/40 → accent/90` hover — the row's "metadata voice".
- Arrow: `ArrowUpRight` `white/25 → accent`, translate `(-2px, 2px)` on hover/focus.

**Hover choreography (one cue, consistently):** cyan 1 px rule under the title sweeps `scaleX 0→1` origin-left, 500 ms `EASE_OUT`; row bg wash `white/[0.02]`; all color ticks 500 ms. Keyboard: same states via `group-focus-visible`, ring via `ROW_CUE`.

**No** rules between rows, no numerals, no card chrome (art direction).

### S6 · Optional closing affordance
- `TODO(user)`: rows already link to `/services` and the header carries the CTA — default is **no** second CTA. Add a bottom link only if wanted.

---

## 3. Loading / entry choreography (the motion spine)

One coordinated ~2.6 s sequence when the section enters view (single viewport trigger, `once: true`):

| t (s) | Element | Animation |
|---|---|---|
| 0.00 | kicker | fade + y10, 600 ms |
| 0.10 | heading words (GSAP `RevealText`) | word-mask rise + blur 4, stagger 55 ms |
| 0.10 | argument column | fade + y20, 700 ms |
| 0.40 | hairline | scaleX draw, 1100 ms |
| 1.50 | accent full-stop | pop `scale 0.3→1`, 450 ms |
| 0.55 + i×0.09 | rows 1–4 | `opacity + y24 + blur6 → 0`, 800 ms; titles' own RevealText rides inside |

**Implementation:** convert the `<ol>` to a framer container with `variants` + `staggerChildren` (house `staggerContainer` pattern in `src/lib/animations.ts`) so the rows are one choreographed group instead of four independent `whileInView`s. Kicker/header/rule keep their own triggers (they sit above the list). GSAP stays the sole owner of word reveals.

**Reduced motion:** container and items render static (`initial={false}` / CSS stand-down); no stagger, no blur, no draw.

**Loading note:** the page is static HTML — "loading animation" = this first-paint/scroll-in sequence. No async data is involved.

---

## 4. Spacing & responsive summary

- Section padding: `py-24 md:py-36` (house).
- Kicker → heading `mt-6 lg:mt-7`; heading grid `gap-y-7 lg:gap-x-16`.
- Header grid → hairline `mt-12 md:mt-16`; hairline → rows: row padding only.
- Mobile: single column everywhere; measure capped; nav-safe (fixed header overlaps ≤ 64 px — `scroll-margin-top` not required, no deep links used).
- Verify no horizontal scroll at 360 / 430 / 768 / 1024 / 1440.

---

## 5. Acceptance checks

1. `npx tsc --noEmit` → clean; `npx eslint src/components/sections/Capabilities.tsx` → clean.
2. Visual: desktop (1440) + mobile (430) captures show — kicker, 2-col header with accent `.` at lg, drawn hairline, four rows in two columns from md, no dead band, no clipped/overlapping type.
3. Entry sequence plays once on scroll-in; reduced-motion renders the full section static and legible.
4. Hover/focus on a row: cyan sweep + scope→accent + arrow step, visible at all breakpoints (on touch: static end-state only).
5. No new raw hex, no new easing curves, no rules/numerals/cards between rows.
6. Cleanup (see §6) — no temp files left in `echo11/public`.

---

## 6. Verification method & cleanup

Headless verification was blocked this session: fragment scroll is ignored, `whileInView` never fires under virtual time at scroll 0, and native `scrollTo` inside an iframe harness is fought by Lenis.

**Plan:**
1. Reuse the same-origin harness `echo11/public/__qa-caps.html` (iframe `/`, script scrolls the inner window). Scroll **must** go through `w.__lenis.scrollTo(top, {immediate: true})` (`window.__lenis` is exposed by `SmoothScroll.tsx`; reduced-motion mode skips Lenis entirely → native scroll works when the flag is forced).
2. The harness now writes a `#dbg` line (`t… top= sy= lenis after=`) — run `--dump-dom`, read it, fix until `after=` is non-zero, then `--screenshot` desktop + mobile.
3. Fallback if still blocked: capture with `--force-prefers-reduced-motion` for layout truth and state explicitly that animation timing was verified by code review only.
4. **Cleanup:** delete `echo11/public/__qa-caps.html` and scratch captures in Design Files `browser/` (`_loc*.png`, `cap-*` after the final pair is kept or removed per user preference). Keep the user's original `browser-capture-*.png`.

---

## 7. Files

| File | Action |
|---|---|
| `echo11/src/components/sections/Capabilities.tsx` | main redesign (all sub-components) |
| `echo11/src/lib/animations.ts` | read/reuse only — add shared variants here only if needed |
| `echo11/public/__qa-caps.html` | temp QA harness — create/fix, **delete after** |
| `browser/*.png` | scratch captures — clean up |

Out of scope: all other sections/pages; `echo11` routing; echo11 "1 Issue" badge (optional investigate only).

---

## 8. Open questions (edit inline or answer in chat)

1. `TODO(user)` Heading scale — keep `4.2rem` cap or bump to `4.6rem` (match Statement)? **Default: bump.**
2. `TODO(user)` Row markers — stay pure typography (default, art direction) or add small mono indices `01–04`? **Default: pure typography.**
3. `TODO(user)` Header → list breath — current `mt-12/16` or larger `mt-14/20`? **Default: larger.**
4. `TODO(user)` Closing CTA under the list — none (default) or a single `See all services ↗`?

---

## Next step

Review this plan — tweak any spec (spacing, scale, motion timings, defaults on the four questions) directly in this file or reply with changes, then hand off to **Design mode** to implement it in `Capabilities.tsx` and run the acceptance checks.

## Provenance

Formalized by Open Design from candidate 30359b06-7e5a-468c-a071-f622abe0b377.
