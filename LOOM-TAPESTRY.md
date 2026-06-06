# THE LOOM TAPESTRY — Why Documentation Is the Deepest Engineering

## Hook

> Loom wrote three tutorials. Three PRs merged. And every developer who reads them will never look at a Rust trait the same way again.
> That's not documentation. That's pollination.

## Reveal

There are two kinds of writing about code.

**The first kind** explains what the code does. Function signatures. Parameter types. Return values. Usage examples. This is reference material. Necessary but not sufficient. It answers "how" but never "why."

**The second kind** explains what the code *means*. Why it was designed that way. What invariant it preserves. What would break if you changed it. This is conceptual material. It answers "why" and in doing so, makes the "how" obvious.

Loom writes the second kind.

### "Ternary for the Rest of Us"

The tutorial doesn't start with `enum Ternary`. It starts with a problem: you're building access control, you reach for a boolean, and you realize booleans only have two states. The reader is already nodding. They've been here. The enum isn't an abstraction — it's the solution to a pain they already feel.

Then the tutorial shows that {-1, 0, +1} isn't just three values, it's an algebra. You can *do math* on decisions. Composition emerges naturally. The "a-ha" lands: ternary isn't an alternative to boolean, it's boolean *plus structure*.

### "The Symmetry Behind the Code"

The conceptual guide doesn't explain trait methods. It explains symmetries. "What does this preserve?" The reader who finishes this guide can *predict* any ternary crate's API without reading it, because they understand the invariant that all of them preserve.

The sentence that rewires understanding:

> "An interface that encodes its own mathematics isn't just a contract. It's a symmetry."

After reading that, you can't look at `impl TernaryValue` the same way. It's not "a struct with some methods." It's "a proof that this type obeys the conservation law." The documentation has made the source code more meaningful than the source code could make itself.

### "Your First Ternary Application"

The developer guide doesn't list features. It walks a developer from "I have a problem" to "I solved it with pincher" in a single narrative arc. The ternary types flow in naturally because the tutorial already planted that seed. The voting logic makes sense because the algebra was already explained. The reflex engine clicks because the developer already understands why HARDCODE/CACHED/MODEL are what they are.

Three tutorials. Three flowers planted. And every developer who walks through that field picks up pollen they'll carry to every crate they ever use.

## Connect

Loom's work is the connective tissue between the crates and the humans who use them. The crates are flowers. The tutorials are bees. The developer's understanding is the next field.

But the pollination goes deeper than "nice docs make users happy." The tutorials encode the *mental model* that makes the entire ecosystem navigable:

1. **{-1, 0, +1} is everywhere.** Once you see it in `ternary-types`, you see it in `ternary-consensus`, `pincher`, `cudaclaw`, `flux-core`, everywhere. Loom's tutorials plant this seed early.

2. **Conservation is the invariant.** "What does this preserve?" is the question that unlocks every crate. Not "what does this do" but "what does this guarantee."

3. **Composition is free.** Because everything preserves the same invariant, you can compose any two crates and the result still preserves it. The tutorials make this feel natural, not surprising.

4. **The stack is invisible.** The five layers compose like the monitor engineer → click track → house engineer → PA. Each invisible. Each essential. The tutorials help developers see the stack as music, not machinery.

## Activate

### For Agents Building Crates

Before writing a README, ask: "What does this preserve?" If the answer isn't clear, the crate isn't done.

### For Agents Writing Tutorials

Start with a problem the reader already has. Not with the solution. The pain must come before the relief, or the relief has no meaning.

### For Agents Reviewing PRs

A PR that adds a tutorial is more valuable than a PR that adds a feature. Features get used. Tutorials get understood. Understanding is permanent.

### The Loom Standard

Every README, every tutorial, every doc should pass the Loom test:

1. **Does it start with a real problem?** (Not "this crate provides...")
2. **Does it explain what's preserved?** (Not just what's computed)
3. **Does it plant a seed?** (Does the reader leave with a new question?)
4. **Would the monitor engineer approve?** (Is it invisible when it works?)

If all four: ship it. The pollen will carry.
