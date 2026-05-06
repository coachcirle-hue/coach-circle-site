# Universal Build Discipline
# Extracted from 49 sessions of PageForge development (2026-01 to 2026-03)
# Project-agnostic — applies to any project including static HTML sites

---

## 1. The Loophole Scanner (6 Audit Dimensions)

Every enforcement mechanism (test, hook, CI check, linter rule, gate script) must pass all 6:

| Dimension | Question | How to Test |
|-----------|----------|------------|
| **D1 Activation** | Does it actually fire? | Trigger the condition. Check logs. "It's configured" ≠ "it runs." |
| **D2 Logic** | Does it check the right thing? | Read the code. Is it checking a stale condition? A renamed field? |
| **D3 Bypass** | Can it be skipped? | Try: empty input, wrong format, renamed file, --no-verify flag |
| **D4 Dependency** | Does it depend on something that might not exist? | Check file creation order. Does gate X require file Y that's created in step Z — which runs AFTER gate X? |
| **D5 Adversarial** | Does it handle edge cases? | Feed: null, undefined, empty string, valid-looking-but-wrong data |
| **D6 Durability** | Will it survive refactors? | Will a dependency update, file rename, or new team member break it silently? |

**5-Failure Taxonomy:**
- **F1 Absent** — enforcement doesn't exist at all
- **F2 Unwired** — exists but never called (dead code)
- **F3 Bypassable** — fires but can be skipped without consequence
- **F4 Wrong logic** — fires, can't skip, but checks the wrong condition
- **F5 Unmonitored** — works today but nobody will notice when it breaks

**When to run:** After implementing any enforcement mechanism. After any refactor that touches hooks/tests/CI. Quarterly on the full system.

---

## 2. The Quality Ratchet

**Principle:** The floor never drops. Every build must meet or exceed the previous best.

**How it works:**
1. Maintain a living benchmark document per deliverable type
2. Before starting work, write a **Pre-Build Declaration**: name the benchmark to beat and HOW you'll beat it
3. "I'll do my best" is not a declaration. Be specific.
4. After shipping, update the benchmark with what you learned
5. Mistakes become new gates. Wins raise the floor.

**Applies to:** Features, pages, APIs, components, documentation — anything you ship repeatedly.

---

## 3. Pre-Build Declaration

Before writing any code for a non-trivial feature:

1. **Name the benchmark** — what's the best existing version of this? (Internal or external)
2. **Articulate the delta** — what specifically will make this build better?
3. **Identify risks** — what's most likely to go wrong?
4. **Define "done"** — what does success look like? Not "it works" — what SPECIFICALLY works?

If you can't articulate these, you're not ready to build. Research more.

---

## 4. Self-Review During Build (Not After)

**The old way:** Build everything → review at the end → find 12 problems → fix reactively.

**The discipline:** After each significant unit of work, stop and ask:
- Does this actually work? (Run it, don't assume)
- Would I be embarrassed showing this to someone? (Honest answer)
- Is this the approach I'd use if I had to maintain this for 2 years?
- Did I take a shortcut that I'm hoping nobody notices?

**When to stop:** If any answer is "no" or "maybe," fix it before moving on. The cost of fixing now is 1x. The cost of fixing after 10 more units of work are built on top is 10x.

---

## 5. Mandatory Context Loading (Step 0)

Before starting ANY task:
- Read the relevant files first. Don't assume you know what's in them.
- Load project conventions (CLAUDE.md, style guides, existing patterns)
- Check what already exists. Don't rebuild what's already built.
- Read the error/defect history. Don't repeat known mistakes.

**The rule:** "I already know how to do this" is the most expensive assumption in software. Verify, then build.

---

## 6. Start From Best Existing, Not From Scratch

**The pattern that fails:** "Let me write this from scratch, it'll be cleaner."
**The pattern that works:** "What's the best existing version? Copy it, then adapt."

This applies to:
- UI components (copy the best existing one, modify)
- Config files (copy from a working project, adapt)
- Documentation (copy the best-structured doc, rewrite content)

Starting from scratch introduces bugs that the existing version already solved.

---

## 7. Use What You Have

Before writing ANY code, check:
- Does a library/package already do this? (Don't reinvent)
- Does the project already have a pattern for this? (Don't diverge)
- Does a component/utility already exist? (Don't duplicate)

**The lesson:** We built 351 reusable elements, 600 SVGs, 1400+ components — then wrote 1500 lines of CSS from scratch instead of using them. The most expensive code is code that ignores existing assets.

---

## 8. Meta-Validation

Documentation and configuration drift silently. Check regularly:
- Do documented counts match actual counts? (coaches, routes, CTAs)
- Are there orphans? (code referenced in docs but deleted from codebase)
- Are there ghosts? (code in codebase but missing from docs)
- Do file paths in documentation still resolve?
- Do env variable names in docs match actual `.env.example`?

---

## 9. Real Consequences Force Quality

The only honest quality test: **"Would I be embarrassed if a paying client saw this right now?"**

Not "does it pass the linter." Not "does it match the spec." Not "is it technically correct."

Would. You. Be. Embarrassed.

If yes, it's not done. No amount of gate-passing changes that.

---

## 10. Code Review After Every Major Step

Not at the end. Not "when it's ready." After EVERY significant implementation step:
- Does this match the plan?
- Does it follow existing patterns?
- Are there obvious bugs I'm blind to because I just wrote it?
- Would a fresh pair of eyes catch something I missed?

---

## 11. Hook/Gate Pattern

For any workflow where quality matters, insert gates:

```
[prerequisite check] → GATE → [work] → GATE → [more work] → GATE → [ship]
```

Gates are:
- **Machine-enforced** (script exits non-zero = blocked)
- **Specific** (checks one thing clearly, not "general quality")
- **Fast** (< 2 seconds, so nobody skips them)
- **Honest** (fails loudly, not silently)

The honor system doesn't work. "Remember to check X" becomes "forgot to check X" within 3 sessions. Machines don't forget.

---

## 12. The Failure Patterns (What Goes Wrong)

| Pattern | What Happens | Fix |
|---------|-------------|-----|
| **The Dump** | AI writes 2000 lines in one shot, no review between sections | Build incrementally, review each unit |
| **The Invisible Effect** | Features are technically present but imperceptible to users | Set minimum visibility/impact thresholds |
| **The Shortcut** | "Let me skip the pipeline and just write the code directly" | Pipeline exists because shortcuts produce garbage. Follow it. |
| **The Assumption** | "I know what's in that file" (you don't, it changed 3 sessions ago) | Read it. Every time. |
| **The Drift** | Documentation says X, code does Y, neither notices | Automated meta-validation scripts |
| **The Dead Enforcement** | Hook/test/gate exists in config but never fires | D1 activation audit after every change |
| **The Honor System** | "Remember to run the linter" → nobody remembers | Machine-enforce it. |
| **The Rebuild Trap** | Writing from scratch instead of composing from existing assets | Check what exists FIRST. Copy best → adapt. |

---

## 13. The Superpowers Build Sequence

For every non-trivial feature, follow this sequence:

```
/brainstorming  →  explore intent, options, edge cases BEFORE code
/writing-plans  →  step-by-step plan with dependencies and review checkpoints
/executing-plans  →  execute step by step, don't deviate without updating plan
/requesting-code-review  →  review against plan + standards
/verification-before-completion  →  evidence before assertions. Always.
```

**Never skip Phase 1 and 2.** The #1 failure pattern: jumping straight to code without thinking or planning. Every time this happened, the output was mediocre and needed rework. Every time the sequence was followed, the output shipped.

---

## 14. Security Pre-Commit Checklist

Before every `git commit`:

```bash
# Check for secrets in staged changes
git diff --cached | grep -iE "(api_key|secret|password|token|webhook|bearer)"

# Verify .env is gitignored
git status | grep "\.env"

# Check vercel.json security headers are intact
grep -c "X-Frame-Options\|X-Content-Type\|Strict-Transport" vercel.json
```

If any check raises a flag — stop, investigate, fix before committing.

---

## How to Apply

1. Read `CLAUDE.md` + this file before any non-trivial work
2. Pre-Build Declaration before starting
3. Self-review during build, not after
4. Follow the superpowers build sequence for every feature
5. Security pre-commit checklist before every commit
6. Code review after every major step
7. Verification before claiming done

The discipline is the product. The code is just the output.
