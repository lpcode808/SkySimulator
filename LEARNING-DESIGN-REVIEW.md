# Moonrise Learning Design Review

## Scope reviewed

- `PRD-00-master-context.md`
- `PRD-01-phase1-scientific.md`
- `PRD-02-phase2-immersive.md`
- `ASTRONOMY-REF.md`
- `index.html`
- `moonrise-p2.html`

This review looks at the project as a learning activity, not just as a technical demo. The main question is whether the current UX/UI helps a learner notice, understand, and remember the core idea:

**Why does the Moon rise in a different place on the horizon every night?**

## What has already been built well

### 1. The project already has a strong instructional shape

The two-file structure is a smart learning design move.

- `index.html` is the legible, diagram-first explanation.
- `moonrise-p2.html` is the emotional, place-based experience.

That split supports two different modes of understanding:

- analytical understanding
- felt, spatial understanding

This is a solid foundation and should be preserved.

### 2. The scrubber is doing the real teaching work

The date scrubber is the strongest part of the interaction model. It creates direct-manipulation learning:

- learners change a single variable
- the moonrise location visibly changes
- the pattern can be explored instead of merely described

That aligns well with the explorable-explanation goal in the PRD.

### 3. Phase 1 already supports pattern detection

The scientific view has several useful learning supports already:

- due East / due West reference lines
- rise and set markers
- a visible arc
- 14-day rise history dots
- live explanatory copy

Together, these make the pattern legible rather than decorative.

### 4. Phase 2 succeeds at emotional framing

The immersive page already creates a sense of place:

- dark sky treatment
- stars
- treeline silhouette
- glowing moon path
- atmospheric overlay

This matters because wonder and orientation are part of learning here, not just aesthetics.

## Main learning UX issues

### 1. The activity does not yet guide the learner's first move

Both pages explain the concept, but neither page strongly structures the learner's first action.

Right now the learner has to infer:

- what to manipulate first
- what to watch while scrubbing
- what change matters most

This makes the experience feel more like a polished demo than a guided learning activity.

### 2. The current time-of-day can distract from the main concept

The app preserves the current clock time while the learner scrubs dates. That is technically coherent, but pedagogically it can muddy the lesson.

Example: a learner may be studying "where the Moon rises" while the visible moon dot is showing where the Moon is at midnight or some other incidental time. That splits attention between:

- rise location
- current sky position
- phase information

For the core question, the most important state is usually the rise event itself, not the Moon's position at the user's current clock time.

### 3. Phase information is visually stronger than it needs to be

Moon phase is useful supporting context, but it is not the central learning target. In both pages, the phase block is prominent enough that it can compete with the main insight about horizon migration.

The current hierarchy sometimes says:

- "look at the Moon phase"

when the activity really needs to say:

- "look at where the Moon comes up"

### 4. The connection between the two views is still implicit

The horizon and orbital views are both strong on their own, but the learner still has to do the mental bridge work alone.

The current toggle says, in effect:

- here is one model
- here is another model

It does not yet clearly say:

- this orbital relationship is the reason the horizon point is shifting

That missing bridge is the biggest conceptual gap in the current experience.

### 5. Mobile composition is currently too compressed

On small screens, the experience still works, but it stops feeling comfortably learnable.

Most noticeable issues:

- the hero block consumes a lot of vertical space before the learner reaches the interaction
- controls stack into a taller sequence, which slows scanning
- the Phase 2 overlay takes up too much of the canvas on mobile
- the actual observation area becomes smaller relative to the supporting chrome

This is especially important because the activity depends on visual noticing.

### 6. Phase 2's overlay competes with the horizon in the exact place the learner needs to watch

The bottom-right phase overlay is beautiful, but it obscures part of the horizon zone and becomes more intrusive on narrow screens.

That is a problem because the horizon itself is the teaching surface.

### 7. The activity supports exploration, but not yet reflection

The experience is good at:

- showing change
- encouraging play

It is weaker at:

- helping the learner verbalize the pattern
- checking whether the learner understood the pattern
- prompting comparison across dates

That means the product is close to an explorable explanation, but not yet a full learning activity.

### 8. Accessibility support is only partial

The pages include good basics like labeled controls and live date text, but there are still clear gaps:

- no strong visible focus treatment
- no keyboard shortcut model
- most meaning lives inside canvas rendering
- no equivalent structured text summary of the currently visible visual relationships

For a learning tool, this matters because accessibility is part of instructional clarity.

## What each phase is currently best at

### Phase 1 strengths

- best for noticing the migration pattern
- best for reading exact reference positions
- best for making the concept screenshot-friendly

### Phase 1 weaknesses

- the top half of the canvas feels a little underused
- the view explains the pattern, but not yet the learner's task
- the orbital view is accurate enough, but still feels somewhat separate from the horizon story

### Phase 2 strengths

- best for emotional engagement
- best for giving the phenomenon a real-world feel
- best for conference-demo or child-facing wonder

### Phase 2 weaknesses

- some atmospheric UI competes with the teaching target
- the overlay is too dominant on small screens
- the horizon view is strong, but the orbital view is less instructionally explicit than the scientific version

## Recommended refinement order

### Priority 1: Clarify the learner task

Add a small, explicit first-step prompt near the scrubber or canvas:

- "Drag the date and watch where the Moon rises on the horizon."

Then add one follow-up prompt:

- "Notice when the rise point is farthest north and farthest south."

This is the highest-leverage improvement because it turns exploration into guided noticing.

### Priority 2: Re-center the activity around rise time

Shift the default interpretation of the scene toward the rise event.

Best options:

- default the visual emphasis to moonrise, not the current clock time
- add a simple mode such as `Now` / `Rise`
- or snap the visible dot to the rise moment by default while keeping current-time data secondary

This would align the interaction with the actual question being taught.

### Priority 3: Make the cross-view bridge explicit

When the learner switches views, add a short sentence or annotation that ties the models together.

Examples:

- "This orbit angle is why tonight's rise point is south of due East."
- "As the Moon's sky position swings north and south, its rise point slides along your horizon."

This will reduce the conceptual jump between the two views.

### Priority 4: Reduce competition from secondary information

Rebalance the visual hierarchy so that:

- rise point and migration pattern are primary
- phase is secondary
- exact rise/set metadata is tertiary

In practice, that likely means:

- smaller or quieter phase treatment in Phase 1
- moving or shrinking the Phase 2 overlay on mobile
- avoiding any panel that covers critical horizon area

### Priority 5: Add comparison support

The next learning leap is helping learners compare states rather than just scrub continuously.

Highest-value additions:

- pin two dates and compare rise positions
- show monthly northmost and southmost markers
- add a simple "compare to due East" callout

This would help learners form a stable mental model instead of relying on fleeting motion memory.

### Priority 6: Improve mobile teaching composition

Mobile should not just be a smaller desktop.

Recommended mobile changes:

- shorten or collapse the hero copy once the learner reaches the app
- reduce non-essential canvas overlays
- move detail panels below the canvas instead of over it
- give the observation area more vertical priority

### Priority 7: Add lightweight reflection prompts

Without turning the project into a quiz app, add one or two simple prompts below the main experience:

- "What do you notice?"
- "Is tonight's moonrise north or south of due East?"
- "Scrub two weeks forward. What changed?"

These prompts would make the experience more teachable in classrooms, demos, or self-study.

### Priority 8: Strengthen accessibility

Recommended additions:

- visible focus states
- keyboard shortcuts for previous day, next day, play, and today
- a compact text summary of the currently shown visual state
- reduced-motion handling for autoplay

## Suggested next build sequence

If refining in short iterations, this is the best order:

1. Add guided instructional prompts and reframe the primary task around moonrise location.
2. Rework information hierarchy so rise-point learning is visually primary.
3. Improve the bridge between horizon view and orbital view.
4. Fix mobile composition, especially the Phase 2 overlay and hero-to-canvas ratio.
5. Add comparison and reflection features.
6. Add accessibility and keyboard improvements.

## Bottom line

The project is already strong as an explorable astronomy artifact. It has real conceptual clarity, a thoughtful two-phase structure, and a high-quality visual foundation.

What it needs next is not a redesign from scratch. It needs a refinement pass that turns the current demo into a more intentional learning activity:

- clearer first-step guidance
- tighter instructional hierarchy
- better connection between the two mental models
- stronger mobile and accessibility support
- small reflection and comparison features that help learners consolidate the pattern
