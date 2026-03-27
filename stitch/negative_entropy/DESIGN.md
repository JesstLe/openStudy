```markdown
# Design System Specification: The Kinetic Archive

## 1. Overview & Creative North Star

### Creative North Star: "The Terminal Authority"
This design system rejects the "softness" of modern consumer web design in favor of a high-density, hardcore engineering aesthetic. It is designed for the high-performance scholar—someone who views knowledge acquisition as a systematic reduction of chaos. We move beyond "templates" by embracing **Brutalist Precision**: a layout logic driven by data density, intentional asymmetry, and a rigorous adherence to structural hierarchy.

To break the "standard" feel, the UI utilizes a **Substrate-First** approach. Instead of floating cards on a background, the interface feels like a single, machined slab of carbon where information is etched, not printed. We replace rounded friendliness with sharp 0px corners and "active" connectivity lines that simulate a live neural network.

---

## 2. Colors: The Chromatic Engine

The palette is rooted in a deep-space charcoal, designed for 12-hour "deep work" sessions. It uses high-energy accents to represent "live" data flowing through a cold system.

### Core Palette (Material Design Mapping)
- **Primary (`#00FF41` / `primary_container`):** The "Matrix Green." Used for high-priority actions, successful executions, and "Active" state nodes.
- **Secondary (`#00A2FD` / `secondary_container`):** The "Cyber Blue." Used for connectivity lines, secondary links, and structural relationships.
- **Tertiary (`#FFBA38` / `tertiary_fixed_dim`):** The "Alert Amber." Reserved exclusively for Spaced Repetition cues and memory decay warnings.
- **Surface (`#131313` / `surface`):** The base obsidian layer.

### The "No-Line" Rule
**Borders are a failure of hierarchy.** Designers are prohibited from using 1px solid borders to define sections. Boundaries must be established through:
1.  **Background Shifts:** Transitioning from `surface` to `surface_container_low`.
2.  **Negative Space:** Using the `Spacing Scale` (e.g., a `6` (1.3rem) gap) to create structural "rifts."
3.  **The Grid:** Utilizing a subtle, 20px dot-grid background (`outline_variant` at 5% opacity) to provide a subconscious sense of alignment.

### Surface Hierarchy & Nesting
Treat the UI as a series of recessed or extruded plates:
- **Level 0 (Base):** `surface` (#131313)
- **Level 1 (Navigation/Sidebar):** `surface_container_low` (#1C1B1B)
- **Level 2 (Main Workspace):** `surface_container` (#201F1F)
- **Level 3 (Overlays/Active Modals):** `surface_container_high` (#2A2A2A)

### The "Glass & Gradient" Rule
For "hardcore" polish, use **Technical Gradients**. Primary CTAs should not be flat; they should use a linear gradient from `primary_fixed_dim` to `primary_container` at a 135° angle. Floating panels should utilize `backdrop-blur` (12px) with a 60% opaque `surface_container_lowest` to simulate a "Heads-Up Display" (HUD).

---

## 3. Typography: The Intellectual Layer

The system uses a dual-font strategy to separate **Content** from **System Data**.

- **Display & Headlines (Space Grotesk):** A low-contrast, geometric sans-serif that feels engineered. Used for high-level concepts and system headers.
- **Body & Titles (Inter):** High-legibility, neutral sans-serif for long-form reading and documentation.
- **Monospace (JetBrains Mono / Fira Code):** Used for all `label-sm`, code blocks, and "Hardcore" metadata. **Note:** All numerical data must be Monospace to ensure tabular alignment.

**Typographic Hierarchy:**
- **Display-LG (3.5rem):** Reserved for system-level statistics or major module entries.
- **Label-SM (0.6875rem):** Monospace. Used for timestamps, UUIDs, and technical metadata. Always uppercase with `0.05em` letter spacing.

---

## 4. Elevation & Depth: Tonal Layering

Traditional drop shadows are forbidden. We simulate depth through **Luminance and Glow.**

### The Layering Principle
Depth is achieved by "stacking" container tiers. To emphasize a focused node in the knowledge graph, do not lift it with a shadow; instead, shift its background to `surface_bright` and apply a `primary` outer glow.

### Ambient Shadows & Neon Glows
- **Focused State:** Use a "Neon Underglow" instead of a shadow. `box-shadow: 0 0 15px 2px rgba(0, 255, 65, 0.15);`
- **Floating HUDs:** If a floating menu is required, use a `surface_container_highest` background with a `0.1rem` "Ghost Border" using `outline_variant` at 15% opacity.

### The "Ghost Border" Fallback
If contrast is required for accessibility in data-dense lists, use a `px` width line of `outline_variant` at 10% opacity. It should be felt, not seen.

---

## 5. Components: Machined Interface

### Buttons: The "Command" Units
- **Primary:** `primary_container` background, `on_primary_container` text. Sharp `0px` corners. On hover, apply a `primary` outer glow.
- **Tertiary:** No background. `primary` text. `0.1rem` underline that expands from center on hover.

### Input Fields: The "Terminal" Style
- **Default:** `surface_container_lowest` background. No border. Bottom-only stroke of `outline_variant` (20% opacity).
- **Active:** Bottom stroke transitions to `primary`. Monospace font for input text.

### Knowledge Cards & Lists
- **Prohibition:** No divider lines between list items.
- **Alternative:** Use a vertical `spacing-2` gap and a subtle `surface_container_low` background on hover to define the "hit area."
- **Data Density:** Use `body-sm` for descriptions but pair it with `label-sm` (Monospace) for technical tags.

### Specialty Component: The "Entropy Node"
A visualization component representing a knowledge unit. 
- **Active Node:** `primary` color fill with a `4px` pulse animation.
- **Decaying Node:** `tertiary_fixed_dim` (Amber) border with a flickering opacity to signal Spaced Repetition urgency.

---

## 6. Do's and Don'ts

### Do
- **Use Sharp Corners:** `0px` everywhere. No exceptions.
- **Embrace Density:** Information is power. Don't fear "busy" layouts if they are structured on the 20px grid.
- **Monospace for Data:** All dates, IDs, and coordinates must be in Monospace.
- **Asymmetric Layouts:** Use a 12-column grid but feel free to leave the leftmost 2 columns for "System Metadata" and the right 10 for content.

### Don't
- **No Rounded Corners:** Never use `border-radius`. It breaks the "hardcore" engineering aesthetic.
- **No Standard Shadows:** Do not use `0 2px 4px rgba(0,0,0,0.5)`. Use background color shifts or neon glows.
- **No Illustrative Icons:** Use only functional, geometric, stroke-based icons. Avoid "filled" or "playful" icon sets.
- **No Generic Transitions:** Use "Instant" or "Step" easing for UI state changes (0.1s duration). Avoid "bouncy" or "organic" animations.

---
**Director's Final Note:** This design system is a tool for the mind. It should feel like a high-end IDE or a spacecraft's navigation console. If it feels "friendly," you've gone too far. Make it efficient. Make it sharp. Make it undeniable.**