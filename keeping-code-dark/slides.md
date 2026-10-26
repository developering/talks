---
theme: default
background: https://cover.sli.dev
title: Keeping Code Dark
info: |
  ## Keeping Code Dark
  Shipping a large feature to master in small, reviewable pieces —
  without ever exposing half-built work to users.
class: text-center
transition: slide-left
---

# Keeping Code Dark

## Building big features without big PRs

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<!--
Follow-up to "Killing Code Review." That talk argued for small PRs merged straight to master.
This talk is the how: what do you actually do with the ticket while the feature
underneath isn't done yet?
-->

---

# The problem

A big feature can't land in one PR.

- Too much to review well
- Too risky to merge in one shot
- Too slow — you're blocked on review instead of building

<v-click>

But `master` still has to be deployable after every merge.

</v-click>

<!--
So the tension: we want small, frequent PRs, but the feature they add up to
isn't done until the last one lands. What do users see in the meantime? Nothing.
-->

---

# The answer: keep it dark

<div class="text-2xl mt-8">

Merge early. Merge often. <strong>Ship nothing.</strong>

</div>

<v-click>

Dark code is code that's in `master`, running in production, and completely
invisible to anyone who isn't looking for it.

</v-click>

---

# Techniques

1. Feature flags
2. Dark pages
3. Unlinked routes
4. Renames / additive schema changes
5. Branch by abstraction

<!--
These aren't mutually exclusive — most features use two or three of these
at once depending on the layer (UI vs API vs DB).
-->

---
layout: two-cols
layoutClass: gap-8
---

# Feature flags

Wrap the new behavior in a flag. Off in prod, on wherever you're testing.

```ts
if (flags.isEnabled('new-account-linking')) {
  return renderNewFlow()
}
return renderOldFlow()
```

::right::

<v-click>

**Good for:**
Behavior changes to an existing screen, gradual rollout, kill switch.

**Watch out:**
Flags that outlive the feature become tech debt. Delete the flag *and* the
dead branch the moment it's fully rolled out — that's part of the ticket,
not a someday cleanup.

</v-click>

<!--
This is the one everyone already does. The risk isn't the flag itself,
it's leaving it in the code six months later with both branches still live.
-->

---

# Dark pages

Ship the whole new screen — routed, rendered, real — just don't link to it.

- No nav entry
- No button
- No deep link anyone's told about

<v-click>

It exists. It works. Nobody can find it because nothing points at it yet.

</v-click>

<v-click class="mt-4">

Add the link in the **last** PR, once the page is actually done.

</v-click>

<!--
This is different from a flag: there's no conditional at all, no old/new
branch to maintain. The page is just... not linked. Way less code to carry
during the build-out.
-->

---

# Unlinked routes

Same idea, one layer down: add the API route or the mobile screen route,
but nothing calls it yet.

```
POST /v2/accounts/relink   <-- exists, deployed, zero traffic
```

<v-click>

Mobile-specific version of this: **put the new screen behind a debug-menu
entry** instead of the real nav. QA and engineering can drive it manually;
end users never see the debug menu.

</v-click>

<v-click class="mt-4">

Wire up the real entry point once the feature's actually done.

</v-click>

---

# Renames and schema changes

The riskiest "small" PRs, because they touch things that already have traffic.

<v-click>

Rule: every step must leave `master` in a working state on its own.

</v-click>

<div v-click class="mt-4">

- Add the new column/field **alongside** the old one — don't rename in place
- Backfill in its own PR
- Cut reads over once backfill is verified
- Cut writes over
- Drop the old column in a **final**, separate PR — after a deploy cycle of confidence

</div>

<!--
This is the pattern that turns a "big scary migration" into five boring,
reviewable, revertible PRs. Each one ships dark in the sense that it changes
nothing a user can observe.
-->

---

# Branch by abstraction

For a bigger internal rewrite: put an interface in front of the thing you're
replacing before you touch the thing itself.

```
Caller → Abstraction → OldImplementation
```

<v-click>

Then swap implementations behind it, one small PR at a time:

</v-click>

<div v-click class="mt-2">

```
Caller → Abstraction → NewImplementation   (flag-controlled, per caller)
```

</div>

<v-click class="mt-4">

Delete `OldImplementation` only after every caller's on the new one.

</v-click>

---

# The discipline that makes this work

<div class="grid grid-cols-2 gap-6 mt-4">
<div>

**Every PR:**
- Leaves `master` deployable
- Changes nothing observable, or is flagged
- Is small enough to review in minutes

</div>
<div>

**Every feature:**
- Has a named "wire it up" PR at the end
- Has a named "delete the flag / old code" PR after rollout
- Doesn't get called done until that cleanup PR merges

</div>
</div>

<v-click>

<div class="mt-8 text-center text-xl">

The dark stuff is a <strong>liability with a due date</strong>, not a permanent shape.

</div>

</v-click>

<!--
This is the part teams skip. Dark code that never gets linked up or cleaned
up is just dead code with extra steps — and unlike normal dead code, it's
often *half-wired*, which is worse: it looks alive to anyone reading it.
-->

---

# Blast radius, honestly

Not every dark-code technique is risk-free just because users can't see it.

<div class="mt-4">

| Technique          | Runtime cost while dark        | Cleanup risk if abandoned         |
|---------------------|----------------------------------|-------------------------------------|
| Feature flag        | Extra branch, extra tests       | Stale flag, two live code paths     |
| Dark page/route      | ~None                            | Dead code, low risk                 |
| Additive schema      | Extra column, dual writes        | Orphaned column                    |
| Branch by abstraction | Extra indirection layer          | Two implementations to maintain     |

</div>

<v-click>

Pick the lightest technique that's actually safe for the layer you're touching.

</v-click>

---
layout: center
class: text-center
---

# Recap

<div class="text-left mx-auto" style="max-width: 32rem;">

1. Small PRs only work if the feature can stay invisible until it's ready
2. Flags, dark pages, unlinked routes, additive migrations — pick per layer
3. The "link it up" PR and the "clean it up" PR are part of the feature,
   not optional follow-through

</div>
