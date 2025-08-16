# Modal Overflow issue

Here’s why that happens:

1. **Flex establishes a new formatting context**
   As soon as you add `flex`, the direct children become **flex items** instead of block elements.
   That means normal block flow (where children just stack and push content downward) is replaced by flex layout rules.

2. **Flex items don’t stretch vertically by default**
   With block flow, a child can naturally grow taller than the parent and push scrolling.
   With flex, the parent (`h-svh`) locks to the viewport height, and the child is laid out **within that height** — if it’s taller, overflow happens in a way that can push content out of the top of the viewport.

3. **Scroll + flex interaction**
   When you combine `flex` with `overflow-y-auto`, the scrollable area is still constrained to the parent height (`100svh`). But the flex child’s baseline alignment (default `align-items: stretch`) changes how overflow is distributed.
   Instead of starting at the top like in block flow, the extra height gets clipped from the top because the flexbox algorithm keeps the flex item inside the container’s height box.

---

🔑 **Key difference:**

* With `display: block`, children flow naturally from the top → overflow scrolls as expected.
* With `display: flex`, children are flex items → the parent treats them differently, and extra height can get hidden above the viewport start.

That’s why simply adding `flex` (even without centering) causes your modal content to look chopped.

```vue
<template>
  <Teleport to="body">
    <div v-if="props.modelValue"
      class="fixed inset-0 h-svh bg-[#020D173D] backdrop-blur-sm z-[30] overflow-y-auto"  <!-- Do not apply flex here. Flex will cut overflowing modal content from top and you can not see it. -->
      @click="close">
      <div class="flex items-center justify-center border-8 py-10">
        <div class="bg-white rounded-[1.25rem] w-md modal-content" :class="{ 'open': props.modelValue }" @click.stop>
          <slot />
        </div>
      </div>
    </div>
  </Teleport>
</template>
```

Imagine your modal wrapper is the **screen height box** (`h-svh`).
Inside it, you put your modal content, which is taller than the screen.

---

### 🟦 Case 1: `display: block` (default)

```
┌─────────────────────────────┐   <- parent (100svh, scrollable)
│ [Modal content starts here] │
│ content line 1              │
│ content line 2              │
│ ...                         │
│ content line N              │  <- taller than screen
│                             │
│ (scrollbar lets you reach ↑)│
└─────────────────────────────┘
```

✅ Everything flows **from the top**.
✅ If modal is taller than the screen, you just scroll.

---

### 🟥 Case 2: `display: flex`

```
┌─────────────────────────────┐   <- parent (100svh, scrollable flex container)
│   [Modal content]           │
│   (laid out as a flex item) │
│   ────────────────────────  │
│   ... but because it's a    │
│   flex item, overflow above │
│   the top is not visible!   │
│   (you can scroll, but the  │
│   start is already shifted) │
└─────────────────────────────┘
```

⚠️ Here, the modal doesn’t behave like normal block flow.
Instead, flex **forces the child to fit inside the flex context height**.
If the child is taller → extra content is pushed **outside the top** (and effectively clipped).

---

👉 That’s why your modal seems “cut from the top” just by typing `flex`.

---

⚡ If you want both **flex centering** and **scroll when overflowing**, the fix is:

* Keep the **outer wrapper** as `block` (so scrolling works normally).
* Add a **flex inner wrapper** for centering:

```vue
<div class="fixed inset-0 h-svh overflow-y-auto bg-[#020D173D] backdrop-blur-sm">
  <div class="min-h-svh flex items-center justify-center py-10">
    <div class="bg-white rounded-[1.25rem] w-md modal-content">
      <slot />
    </div>
  </div>
</div>
```

This way:

* If modal is short → it’s centered.
* If modal is tall → it scrolls from the top without cutting.
