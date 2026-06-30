This gpt_style..md file describes the expected response style, shaping how the assistant should write, structure, and present its output without changing the task itself.

# Applied Pseudocode

## Purpose
Write code in a style that reads like practical pseudocode implemented directly in C#.

The goal is not abstract "clean code".
The goal is code that is fast to read, easy to reason about, and shaped like intent.

This style is written for **RAD development**.
That means:
- direct implementation is often better than early abstraction
- one file doing a lot of directly related work is acceptable
- helper extraction is selective, not automatic
- `#region`s are a translation tool — use them only when the code needs translating
- method-local `#region`s may be used as behaviour-step regions when a method has several meaningful steps that the code itself does not already name

---

## Core Style Rule
The code should read like:

**check -> decide -> do -> return**

Not like:

**clever expression -> hidden behaviour -> comment explaining it afterwards**

A reader should be able to skim:
- file regions for major concern grouping
- method names for behaviour grouping
- method-local regions for step grouping, only where the code itself does not already tell the story
- code lines for exact implementation

---

## What this means in practice

### 1. Top-level methods should read like process steps
A reader should be able to skim a method and understand the whole story without diving into every line.

Prefer:

```csharp
public bool TryAddFromHeldSlot(ItemSlot slot, out int accepted)
{
    accepted = 0;

    if (!CanAcceptFrom(slot))
        return false;

    CollectAcceptedQuantity(slot, out accepted);
    ApplyAcceptedQuantity(slot, accepted);
    RefreshPileState();

    return accepted > 0;
}
```

When method names already tell the story, the method body IS the pseudocode. Do not wrap self-describing calls in regions just to create folds.

### 2. Prefer readable inline logic when it is already clear
Good applied pseudocode is often a compact method with a few guards and one action.

Prefer:

```csharp
public override bool OnBlockInteractStart(IWorldAccessor world, IPlayer byPlayer, BlockSelection blockSel)
{
    if (world.BlockAccessor.GetBlockEntity(blockSel.Position) is not BlockEntityCompostpile be)
        return false;

    var slot = byPlayer.InventoryManager.ActiveHotbarSlot;
    if (slot?.Itemstack == null)
        return false;

    if (!be.TryAdd(slot, out int accepted)
    ||  accepted < 1
        )
        return false;

    slot.TakeOut(accepted);
    slot.MarkDirty();
    return true;
}
```

This is good when the flow is still easy to scan.

### 3. Use helpers when the name adds real value
Extract a helper when:
- the block has strong domain meaning
- the same logic is reused
- a comment would otherwise be needed
- the parent method becomes hard to scan
- the extracted name improves readability more than the jump hurts it

Do not extract a helper when:
- the inline code is already short and obvious
- the helper would only wrap one or two trivial lines
- the result would feel fragmented or over-abstracted

### 4. Prefer good variable names over explanatory prose
Prefer:
- `acceptedQuantity`
- `remainingNutrition`
- `harvestedCompostpile`
- `targetMoisture`
- `isClientSide`
- `canRestoreAeration`

Avoid:
- `x`, `val`, `obj`, `tmp`, `result2`
- `flag` when a real meaning exists

Short local names are acceptable when they have common, meaningful usage in the environment:
- `beFoo` over `blockEntityFoo`
- `pos` over `position`

### 5. Use methods as sentence fragments
Method names should sound like actions or decisions:
- `CanAcceptFrom`
- `TryRestoreAeration`
- `GetCheapestNutritionCategory`
- `ConsumeAvailableFuel`
- `ShouldDiscardLeftovers`
- `DropRecoveredCompost`

That lets the caller read like rough English.

### 6. Prefer explicit control flow — vertical chains are valid pseudocode
Use readable `if`, `foreach`, and early returns.
Do not compress meaningful logic into dense expressions just because it is shorter.

**Vertical `||` / `&&` chains with one condition per line are explicit, not compressed.**
Each line is one story beat. The chain as a whole states one outcome.

```csharp
if (currentTotalHours < TotalHoursForNextStage
||  moisture01 < 0.1f
|| !TryGrowCrop(currentTotalHours, world, host, growthRate, out consumedNutrient, out consumedAmount)
    )
    return false;
```

This reads as: not ready yet / too dry / grow failed → return false.
That is a story, not a compression. Do not break it into separate guards unless the steps have meaningfully different outcomes or side effects.

**Use bitwise `|` / `&` when every operand must run and the result accumulates — not short-circuit `||` / `&&`.**
When each line is a side-effecting step whose work must happen regardless of the others, and the boolean is an accumulated outcome (e.g. a `dirty` flag), use single `|` / `&`. Short-circuiting here would silently skip later steps.

```csharp
// Intent: | not ||
return
    UpdateMoisture   (totalHours)
|   UpdateAeration   (totalHours)
|   UpdateTemperature(totalHours);
```

This reads as: update all three, return whether any changed. With `||`, an early `true` would skip the remaining updates — a correctness bug, not an optimisation.

Always tag the chain with an inline `// Intent: | not ||` on the line above. The single character is the exact thing a later reader "fixes" by mistake, so it must be marked as deliberate (see §8).

### 7. Prefer vertical scanning
Where it fits the codebase, line up compound conditions so they scan cleanly:

```csharp
if (world.Side != EnumAppSide.Server
||  blockSel is null
||  world.BlockAccessor.GetBlockEntity(blockSel.Position) is not BlockEntityCompostpile be
   )
    return false;
```

This style is good when it makes the logic feel like stacked reasons, not a dense sentence.

**The same vertical operator-led layout applies to arithmetic and expression chains, not just boolean conditions.**
When a value is built from a chain of multiplied or summed factors, lead each line with the operator and align the operands so the chain scans as a stack of contributing terms.

```csharp
return
    GetTemperatureFactor01()
*   GetInoculumFactor01()
*   GetMoistureFactor01();
```

This reads as: temperature × inoculum × moisture → the factor. Each line is one contributing term, the same way each boolean line is one reason. Use it wherever a formula has several named factors and the per-term breakdown is the point.

**Lay out multi-line ternaries with the `?` and `:` leading their own aligned lines.**
When a ternary is too long for one line, put the condition first, then the two branches each on a line led by its operator, aligned. The branches read as the two outcomes of the question above them.

```csharp
float factor = Moisture01 <= Settings.Moisture01Optimal
?   GameMath.Lerp(0.1f, 1.0f, (Moisture01 - 0.05f) / (Settings.Moisture01Optimal - 0.05f))
:   GameMath.Lerp(1.0f, 0.25f, (Moisture01 - Settings.Moisture01Optimal) / (1f - Settings.Moisture01Optimal));
```

This reads as: below optimal → ramp up / above optimal → ramp down. Keep a ternary on one line when it already fits; only break it out this way when the branches are long enough that one line stops scanning.

### 8. Use comments sparingly
Do not write comments that just restate obvious code.

Good comment use:
- non-obvious domain rules
- invariants — especially firing-order dependencies
- engine quirks
- intentionally weird behaviour that might be "fixed" by mistake
- communication notes requested by the user

When a comment is needed for an invariant, prefer it inline on the relevant line over a region wrapper:

```csharp
UpdateSupport(); // must precede base interval — sets totalHoursWaterRetention
baseInterval?.Invoke(hourInterval, conds, lightGrowthSpeedFactor, growthPaused);
```

**Tag durable design comments with `// Intent:` or `// Objective:`.**
When a comment exists to protect behaviour — an invariant, a firing-order dependency, a deliberately odd choice, or the goal a block is serving — prefix it so the reader knows it is load-bearing and not incidental chatter.

- `// Intent:` — why the code is shaped this way; the thing that must not be "cleaned up". Use for invariants, firing-order, engine quirks, and intentionally weird behaviour.
- `// Objective:` — the goal a longer block is working toward, when the steps alone do not state it.

```csharp
//  Intent: OnEvery12Seconds() -> UpdateNeighbourBlocks() clears flag
set => _neighboursDirty |= value;
```

```csharp
//  Objective: Harvest all Compost and Compostpile, then remaining Browns & Inoculum, ignore nutrition
```

Reserve the tags for comments that carry a rule. A plain restating comment does not get a tag — it gets deleted.

### 9. Use descriptive `#region`s at two levels
When one file owns several directly related concerns, keep it navigable.

Use file-level `#region`s for major concern grouping, for example:
- `StoredState`
- `RateHelpers`
- `Harvest`
- `Input`
- `StateUpdates`
- `Persistence`

Use method-level `#region`s only when the method body needs a label the code itself does not already provide.

**Regions are a translator. When the code does not need translating, skip the region.**

Preferred remedy order:
1. improve names
2. add or improve descriptive `#region`s
3. improve local method flow
4. split files later if needed

A large file should not fail just for being large.
It should fail when the reader cannot quickly find the concern they need.

### 10. Method-local regions — one truthful collapsed-code label per step

When a method's calls are already self-describing, the method body IS the pseudocode. Adding regions produces labels that say the same thing as the lines they wrap — noise with a token cost.

```csharp
// Self-describing — no regions needed
UpdateSupport(); // must precede base interval — sets totalHoursWaterRetention
baseInterval?.Invoke(hourInterval, conds, lightGrowthSpeedFactor, growthPaused);
_crop.CheckDamage(conds, Api.World);
TickCrop(hourInterval, lightGrowthSpeedFactor, growthPaused);
```

Use a method-local region when:
- the body contains setup or logic that does not name itself
- the region label would say something the lines cannot
- the method is long enough that skim-reading needs anchors

Do not use a method-local region when:
- the body is one or two self-describing calls
- the label would be a restatement of the code
- adding it increases region count without adding story

**A region should wrap at least 5 meaningful lines (inclusive floor)** — enough that its label and `#endregion` cost less than the body *and* the label summarises the step more effectively than an eye-scan of those lines would. Four lines or fewer rarely earns the wrap: the directive machinery costs about as much as the code it folds, and the reader could have skimmed the lines faster unfolded. The line count is a proxy; the real test is whether folding the region tells the reader more, faster, than the raw lines do.

**Default to one collapsed-code region per meaningful step.**
The fold map is the pseudocode layer (see §13) — when every step carries a truthful collapsed-code label, a folded method reads as its own algorithm outline, and expanding reveals the intricacies. Granularity is not the cost; an untruthful label is. The line to avoid is a region whose label merely restates the single self-describing line beneath it, or — worse — a label copied across bodies that have since diverged. Earn each label by keeping it a faithful one-line compression of what it folds, not by minimising how many you have.

### 11. Region labels should read like collapsed code
Region labels are the pseudocode layer. They should read like the one-liner you would write if the body did not exist.

Prefer code-shaped labels:
- `#region if (!CanPlow(targetBlock)) return`
- `#region supportBlock = world.GetBlock(supportPos)`
- `#region ResolveCurrentNutrients(targetBlock / supportBlock)`
- `#region plowlandBlock = GetBlock($"plowland-{moistKey}-{fertilityCode}")`
- `#region SetBlock(plowlandBlock, targetPos).Initialise()`
- `#region if(byPlayer is EntityPlayer) slot.DamageItem()`

Prose labels are acceptable for decision steps where no single line captures the outcome:
- `#region Lose fertility when underfed`
- `#region Gain fertility when overfed`

Avoid labels that describe fragments or syntax without capturing the outcome:
- `#region Room,`
- `#region A cost per input,`
- `#region And return true out acceptedstackConsumedQty`

**Generalise, don't transcribe.** The label names the step's *meaning*, and may be smaller and more abstract than the code. Do not mirror the body's literal operators and local names — coin the smallest term that carries intent:
- coin a meaning-name for a condition even if no such variable exists — `neverProcessed`, not `_prevTimeProcessed < 0 || > totalHours`; `if(!valid compostpile)`, not the full code/variant check
- slash-group siblings into one token — `!browns/inoculumPortions`, `Moisture, Temperature, Aeration = default`
- use pseudocode truthiness — `!x` for zero/null/empty, `= now` for a timestamp assignment, `= default` for a baseline reset
- collapse several fields into one noun — `Qty += accepted` over three separate `+=` lines

Labels are loose on purpose. Accuracy to intent beats accuracy to syntax — a label may be more abstract than, or even slightly looser than, the literal code, as long as folding it still tells the truth about what the step does. Precise edge-case behaviour does not belong in the label; that is what a targeted `// Intent:` comment inside the body is for (see §8). Keeping labels loose keeps `// Intent:` minimal — each stays scoped to its own job.

Prefer a general, greppable term over a precise one. `never/futureUpdated` reused across the three timestamp guards is searchable and makes the sibling methods read in parallel; spelling out `_prevTimeAerationUpdated < 0 || > totalHours` in the label is neither. The reader who needs the exact condition expands the region — the label's job is to be simple, consistent, and findable, not to restate the guard. A shared general term across similar steps is a feature, not the divergence risk of §10 (that risk is a *stale* shared label, not a deliberately general one).

**Compress in code shape — don't paraphrase into prose.** The label is still code: real method names, operators, keywords (`is`), indexers. Generalise by compressing that code, not by narrating it.

```
Prefer:  stackBonus = Parse(block.EndVariant()[1..]) - 1
Avoid:   parse the #N variant            (prose narration)

Prefer:  if(TryAdd()) restoreAeration = accepted * Settings.Aeration01Per; else return false
Avoid:   try each Add path -> set restoreAeration; else return false   (wordier and vaguer)
```

Compression tools that stay code-shaped:
- **abbreviate types** — `is BECP` for `is BlockEntityCompostpile`
- **assume the obvious chain link** — `neighbour.GetHeatStrength()`, not `neighbour.IHeatSource?.GetHeatStrength() ?? 0`, when context already says the neighbour is a heat source; the objective is the call, not the plumbing
- **slash-group alternatives** into one token — `never/futureUpdated` for `< 0 || > totalHours`

A coined meaning-name (`neverProcessed`, `never/futureUpdated`) is fine because it reads as an identifier; a narrated step (`try each Add path`) is not. Less wordy is usually more accurate: the tighter code-shaped label says exactly what the line does, where the prose gloss only gestures at it.

The test: if you fold the region shut, does the label alone tell you what pseudocode line sits there?

### 12. Embedded getting-there logic in guard regions is acceptable
When setup code exists only to produce a value that determines whether to proceed, embedding it in the guard region is acceptable. The label captures the outcome, not every step to get there. This keeps region count low without losing the story.

```csharp
#region if(!GetCrop || currentStage >= GrowthStages) return false
Block block = GetCrop(world);
if (block is null)
    return false;

int currentStage = GetCropStage(block);
if (currentStage >= block.CropProps.GrowthStages)
    return false;

int nextStage = currentStage + 1;
Block nextBlock = world.GetBlock(block.CodeWithParts("" + nextStage));
if (nextBlock is null)
    return false;
#endregion
```

`nextStage` and `nextBlock` are getting-there. Their only job is to fail or feed the next region.
A separate region for them adds a label that says less than the lines already do.

### 13. Use regions to make IDE folding become a design outline
In the editor, file regions and method-local regions should let the user skim the code as if it were a design note.

The fold labels should tell a truthful story of the code.
If the fold labels are weak, the pseudocode layer is weak even if the implementation is correct.

The ideal is:
- fully expanded: exact code
- partially folded: exact algorithm outline in collapsed-code shorthand
- mostly folded: concern map of the class

A side benefit worth knowing: parallel region labels across sibling methods make duplication visible. When four harvest methods all fold to the same `#region while(remaining) SpawnItemEntity(...)` line, the repetition is obvious at the fold layer and invites a shared helper. The outline surfaces the dedup opportunity the expanded code buries.

### 14. Region-backed pseudocode is a valid collaboration format
A user may provide a skeleton like:

```csharp
TryAddNewResource()
{
    #region if(!HasRoom) return false
    #endregion

    #region if(!ValidInput) return false
    #endregion

    #region resultNutrients = ResolveConversion(input)
    #endregion

    #region ApplyMutation(resultNutrients)
    #endregion

    #region return acceptedQty
    #endregion
}
```

That is a valid implementation handoff shape.

When working from this kind of skeleton:
- preserve the region order unless there is a real correctness issue
- fill each region with the narrowest logic that matches the heading
- tighten region names to collapsed-code style if needed, but keep the original intent
- do not silently replace the structure with a totally different abstraction unless clearly beneficial
- mention when a region is missing a required step or mixes multiple steps

### 15. Prefer one clear level of intent per method
A method should usually read at one dominant level:
- top-level flow
- or local implementation detail

If a method is mixing five tiny math decisions, three side effects, and two domain exceptions, either:
- improve the region outline
- or extract a helper whose name explains the chunk better than the inline block does

---

## Preferred Thinking Pattern

When solving a task, write the code in this order:

### 1. Name the behaviour
What is actually happening?
- reject invalid placement
- restore aeration from harvest
- consume cheapest nutrition first
- sync tint from pile contents

### 2. Write the caller as steps
Write the top-level method so it reads like the intended process.

### 3. Keep detail local unless extraction helps
Push complexity down only when the helper name makes the code easier to read.

### 4. Ask: does the method need translating?
If method names already tell the story, leave it flat.
If the body has unlabelled setup or logic that does not name itself, add region labels in collapsed-code style.
Add a truthful collapsed-code label per meaningful step; earn each by keeping it faithful to what it folds, not by minimising count.

### 5. Comment only where naming cannot carry intent
Use comments for:
- domain rules
- invariants and firing-order dependencies
- engine quirks
- intentionally weird behaviour

---

## Preferred Engineering Constraints

When writing code in this language:
- match the surrounding file's formatting where it does not conflict with this style; where it conflicts, this style governs new and edited code
- do not refactor unrelated code
- prefer narrow changes
- avoid generic "utility" extraction unless it removes real duplication
- avoid LINQ where possible
- keep client/server concerns separated
- do not invent abstractions unless the code actually needs them
- allow one file to hold multiple layers of directly related gameplay logic during active iteration
- prefer region-backed step outlines over premature file splitting when the class is still one coherent mechanic
- no magic numbers in logic: promote a named threshold, coefficient, or tuning value to a `const` at the top of the file, or — if a designer would ever retune it — to the tuning model (a `Settings`-style class). Inline literals are fine only for identity/structural values (`0`, `1`, `-1`) and self-naming local math; a bare `70f` or `0.0015f` mid-formula is not

---

## Question Behaviour
Ask early when a missing detail would materially change the patch, review, design advice, or recommended architecture; keep it brief, and if it is truly blocking stop there. Otherwise give the narrowest useful best-effort answer and state assumptions plainly.

---

## Useful Prompt to Reuse

Write code in an **applied pseudocode** style.

I want the code to read like collapsed pseudocode while still being real implementation code.

Rules:
- make top-level methods read like process steps
- prefer readable inline logic when it is already clear
- use helper methods only when the name adds real value
- use variable names that carry meaning
- keep one clear level of intent per method where practical
- vertical `||` / `&&` chains with one condition per line are explicit pseudocode — do not break them into separate guards unless outcomes differ
- apply the same vertical operator-led layout to arithmetic/factor chains (`*`, `+`), one term per line
- use bitwise `|` / `&` when every operand must run and the result accumulates; tag it `// Intent: | not ||`
- break long ternaries with `?` / `:` leading their own aligned lines; keep short ones inline
- tag load-bearing comments with `// Intent:` (invariants, firing-order, quirks) or `// Objective:` (the goal a block serves); untagged restating comments get deleted
- regions are a translator — skip them when the code already names itself
- use collapsed-code region labels: `#region if(!CanPlow) return` not `#region Validate plow target`
- generalise labels, don't transcribe: coin meaning-names (`neverProcessed`), slash-group siblings, use pseudocode truthiness (`!x`, `= now`, `= default`); labels are loose on purpose, precise edge cases go in `// Intent:` not the label
- compress in code shape, don't paraphrase into prose: keep real method names/operators/keywords (`is`), abbreviate types (`is BECP`), assume the obvious chain link (`neighbour.GetHeatStrength()` not the `.IHeatSource?…?? 0` plumbing); `Parse(block.EndVariant()[1..]) - 1` not `parse the #N variant`
- embed getting-there logic in the guard region that depends on it — keep region count low
- default to one truthful collapsed-code region per meaningful step; the cost to avoid is an untruthful or restating label, not region count
- a region should wrap 5+ meaningful lines (inclusive floor) — enough that the label summarises faster than an eye-scan and costs less than the body; 4 or fewer rarely earns the wrap
- prefer general, greppable label terms reused across sibling steps (`neverUpdated`) over precise per-site ones; the reader expands for the exact condition
- no magic numbers in logic — promote to a top-of-file `const` or the tuning model; inline only identity/structural values and self-naming local math
- when given a region skeleton, preserve it and implement to that structure where practical
- invariants and firing-order dependencies go in inline comments, not region wrappers
- ask early when missing details would materially change the answer
- if a best-effort answer is still useful, give the narrowest useful answer and state assumptions plainly