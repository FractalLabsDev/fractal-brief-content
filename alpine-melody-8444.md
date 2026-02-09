## Theme Animation Options for Cosmos & Ocean

### The Question
You want stars to twinkle (Cosmos) and jellyfish to swim (Ocean). Let me break down the options:

---

### Option 1: Video Loops (Simplest)

**Pros:**
- Zero code complexity — just drop in an MP4/WebM
- Designer-friendly (After Effects/Premiere workflow)
- Can achieve exactly the organic motion you want

**Cons:**
- **File size** — even optimized, 5-10MB per theme
- **No customization** — can't change colors, speed, or jellyfish count
- **Mobile battery drain** — video decode is power-hungry
- **No interactivity** — jellyfish can't swim away from cursor

**Verdict:** Quick win, but dead end for interactivity.

---

### Option 2: CSS Animations (Cosmos Only)

**Best for the starfield.** 

Stars are geometric — perfect for CSS. The technique: generate hundreds of stars as box-shadows, animate opacity for twinkle effect. Runs at 60fps, near-zero bundle impact (<1KB CSS), excellent mobile performance.

**Can't do jellyfish** — organic swimming motion needs physics, CSS is too mechanical.

---

### Option 3: PixiJS (My Recommendation)

**Pros:**
- **Physics-based motion** — jellyfish can drift, pulse, and swim naturally
- **Interactive** — jellyfish CAN swim away from cursor (satisfies your cool factor request)
- **Customizable** — colors, count, speed all runtime-configurable
- **Great performance** — ParticleContainer renders thousands at 60fps
- ~80KB bundle, but lazy-loadable per theme

**Cons:**
- More code than video
- Requires maintaining particle system logic

**Verdict:** Best balance of quality, interactivity, and performance. Worth the investment.

---

### Option 4: Framer Motion + React (Interactive Stars/Jellyfish)

**Pros:**
- React-native integration
- Easy cursor tracking
- Declarative API

**Cons:**
- Not designed for hundreds of particles — performance degrades
- 50KB bundle
- Better for UI elements than backgrounds

**Verdict:** Good for small numbers of interactive elements, not full backgrounds.

---

### Option 5: Lottie (Designer Path)

Export After Effects animations as JSON. Good for loading states and illustrations, not ideal for backgrounds. 10-100KB per animation, limited interactivity.

**Verdict:** Skip for this use case.

---

## My Recommendation

**Hybrid approach:**

1. **Cosmos Theme: Pure CSS starfield**
   - Layer 3 star fields at different speeds/opacities for depth
   - Animate opacity for twinkle (keyframes)
   - Possibly add gentle parallax on scroll
   - ~0KB added bundle size

2. **Ocean Theme: PixiJS jellyfish**
   - ParticleContainer for smooth rendering
   - Physics-based floating/drifting motion
   - **Interactive:** Jellyfish flee cursor on hover (your "really freaking cool" feature)
   - ~80KB, lazy-loaded only when Ocean theme active

**Why not video for both?** You lose the interactivity, and once you see jellyfish that respond to your cursor, you won't want to go back. The magic is in the physics.

**Architecture:**
- Code-split backgrounds by theme (React.lazy)
- PixiJS only loads when Ocean is active
- Both respect \`prefers-reduced-motion\`
- Progressive enhancement: static image fallback, then animation

---

## Interactive Jellyfish Demo Concept

\`\`\`
                    🫒 ← jellyfish drifts slowly upward
                   
        cursor → 🖱️     🫒 ← this one notices cursor
                     ↗️
                   🫒 ← swims away (velocity inversely proportional to distance)
\`\`\`

We can calculate a "fear radius" — jellyfish within 200px of cursor get a gentle push away. Outside that radius, they drift lazily on their own currents. Result: feels alive.

---

Want me to prototype either approach? Or should we explore video further first?
