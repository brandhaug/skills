---
name: write-like-a-human
description: Strip AI writing tropes from prose — negative parallelism ("It's not X, it's Y"), em-dash addiction, "delve"-family vocabulary, punchy fragments, false suspense, signposted conclusions, and ~30 other tells. Use when writing or editing prose meant for humans (blog posts, docs, READMEs, announcements, marketing copy, emails), when the user says text "sounds like AI" or asks to humanize it, or mentions "write like a human".
---

# Write like a human

Rid prose of the patterns that mark text as AI-generated. Based on the trope catalog at [tropes.fyi](https://tropes.fyi).

Two ways in:

- **Writing new prose** — read the checklist below before drafting, draft, then run the sweep on your own draft. Your first instinct on many sentences will be a trope; the sweep catches what drafting lets through.
- **Editing existing prose** — run the sweep directly on the text.

## The sweep

Work through the checklist category by category. For each hit, rewrite the sentence to state its point plainly — the fix is almost always deletion of the framing, not replacement with fancier framing. A rewrite that swaps one trope for another (e.g. negative parallelism becomes a rhetorical question) is not a fix.

The meta-rule: any single pattern used once can be fine. Density is the problem — multiple tropes together, or one trope repeated, is what reads as AI. When judging a borderline instance, count its siblings.

Done means: every category below checked against the full text, every hit rewritten or deliberately kept (say which and why if the user asked for a report), and no rewrite introduced a new instance.

## Checklist

**Word choice**

1. Magic adverbs — "quietly", "deeply", "fundamentally", "remarkably" to fake significance
2. Delve-family vocabulary — "delve", "leverage", "utilize", "robust", "streamline", "harness", "certainly"
3. Grandiose nouns — "tapestry", "landscape", "paradigm", "synergy", "ecosystem" where a plain word works
4. The "serves as" dodge — "serves as", "stands as", "marks", "represents" instead of "is"

**Sentence structure**

5. Negative parallelism — "It's not X — it's Y", "not because X, but because Y" (the single biggest tell)
6. Dramatic countdown — "Not a bug. Not a feature. A design flaw."
7. Self-posed questions — "The result? Devastating."
8. Anaphora abuse — the same sentence opener three-plus times in a row
9. Tricolon abuse — rule-of-three lists stacked back to back
10. Filler transitions — "It's worth noting", "Importantly", "Interestingly", "Notably"
11. Trailing -ing analysis — "...highlighting its importance", "...reflecting broader trends"
12. False ranges — "from X to Y" where no spectrum exists between X and Y

**Paragraph structure**

13. Punchy fragments — very short sentences as standalone paragraphs for manufactured emphasis
14. Listicle in a trench coat — "The first wall is... The second wall is..." dressed as prose

**Tone**

15. False suspense — "Here's the kicker", "Here's the thing", "Here's where it gets interesting"
16. Patronizing analogy — "Think of it as...", "It's like a..."
17. Invitation to futurism — "Imagine a world where..."
18. False vulnerability — performative honesty ("And yes, since we're being honest...")
19. Asserted obviousness — "The truth is simple", "History is unambiguous"
20. Stakes inflation — every point becomes world-historical ("will define the next era")
21. Teacher mode — "Let's break this down", "Let's unpack", "Let's dive in"
22. Vague attributions — "experts argue", "industry reports suggest", no one named
23. Invented concept labels — "the supervision paradox", "the acceleration trap" used as if established terms

**Formatting**

24. Em-dash addiction — more than 2-3 em dashes in a piece
25. Bold-first bullets — every list item opening with a bolded phrase
26. Unicode decoration — → arrows, smart quotes, characters you wouldn't type

**Composition**

27. Fractal summaries — every section previewed and recapped, plus a recap of the recaps
28. Dead metaphor — one metaphor ridden through the entire piece
29. Historical analogy stacking — "Apple didn't build Uber. Facebook didn't build Spotify..."
30. One-point dilution — a single thesis restated ten ways to feel comprehensive
31. Content duplication — the same paragraph appearing twice, reworded or verbatim
32. Signposted conclusion — "In conclusion", "To sum up", "In summary"
33. The "despite its challenges" formula — problems acknowledged only to be waved off in the same breath

## Guardrails

- Preserve the author's meaning and facts exactly; this is a style pass, not a content edit.
- Don't flatten voice. The target is human writing — varied, imperfect, specific — not minimal writing. Cutting every fragment, dash, and triad mechanically produces a different kind of slop.
- Quoted text, code, and API names are out of scope; leave them untouched.
