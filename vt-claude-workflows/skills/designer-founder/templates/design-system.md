# Design System: {project_name}

**Stitch Project ID:** {stitch_project_id}
**Generated:** {date}
**Last Updated:** {date}

---

## 1. Visual Theme & Atmosphere

### Overall Feel
{describe the overall visual feel - e.g., "Modern and minimalist with soft edges and subtle depth"}

### Design Principles
- {principle_1}
- {principle_2}
- {principle_3}

---

## 2. Color Palette & Roles

### Primary Colors
| Name | Value | Usage |
|------|-------|-------|
| Primary | `{color}` | Main CTAs, active states |
| Primary Hover | `{color}` | Hover state for primary |
| Primary Foreground | `{color}` | Text on primary backgrounds |

### Secondary Colors
| Name | Value | Usage |
|------|-------|-------|
| Secondary | `{color}` | Secondary actions |
| Secondary Hover | `{color}` | Hover state for secondary |
| Secondary Foreground | `{color}` | Text on secondary backgrounds |

### Neutral Colors
| Name | Value | Usage |
|------|-------|-------|
| Background | `{color}` | Page background |
| Foreground | `{color}` | Primary text |
| Muted | `{color}` | Subtle backgrounds |
| Muted Foreground | `{color}` | Secondary text |
| Border | `{color}` | Borders and dividers |

### Semantic Colors
| Name | Value | Usage |
|------|-------|-------|
| Destructive | `{color}` | Errors, delete actions |
| Success | `{color}` | Success states |
| Warning | `{color}` | Warning states |

### Dark Mode Variants
{if dark_mode_supported}
| Light Mode | Dark Mode | Usage |
|------------|-----------|-------|
| `{light}` | `{dark}` | Background |
| `{light}` | `{dark}` | Foreground |
{/if}

---

## 3. Typography Rules

### Font Family
- **Sans:** `{font_family}` (e.g., Inter, system-ui)
- **Mono:** `{mono_family}` (e.g., JetBrains Mono, monospace)

### Scale
| Name | Size | Weight | Line Height | Usage |
|------|------|--------|-------------|-------|
| H1 | `{size}` | `{weight}` | `{lh}` | Page titles |
| H2 | `{size}` | `{weight}` | `{lh}` | Section headers |
| H3 | `{size}` | `{weight}` | `{lh}` | Subsections |
| Body | `{size}` | `{weight}` | `{lh}` | Default text |
| Small | `{size}` | `{weight}` | `{lh}` | Captions, labels |

### Text Colors
- Primary: `foreground`
- Secondary: `muted-foreground`
- Links: `primary`
- Disabled: `muted-foreground` with 50% opacity

---

## 4. Component Stylings

### Buttons
| Variant | Background | Text | Border | Hover |
|---------|------------|------|--------|-------|
| Default | `primary` | `primary-foreground` | none | `primary/90` |
| Secondary | `secondary` | `secondary-foreground` | none | `secondary/80` |
| Outline | transparent | `foreground` | `border` | `muted` |
| Ghost | transparent | `foreground` | none | `muted` |
| Destructive | `destructive` | `destructive-foreground` | none | `destructive/90` |

### Border Radius
| Element | Radius |
|---------|--------|
| Buttons | `{radius}` |
| Cards | `{radius}` |
| Inputs | `{radius}` |
| Modals | `{radius}` |

### Shadows
| Name | Value | Usage |
|------|-------|-------|
| sm | `{shadow}` | Subtle elevation |
| md | `{shadow}` | Cards, dropdowns |
| lg | `{shadow}` | Modals, popovers |

### Spacing Scale
Base unit: `{base}` (e.g., 4px)

| Name | Value | Usage |
|------|-------|-------|
| xs | `{value}` | Tight spacing |
| sm | `{value}` | Compact elements |
| md | `{value}` | Default spacing |
| lg | `{value}` | Section spacing |
| xl | `{value}` | Large gaps |

---

## 5. Layout Principles

### Grid System
- **Container max-width:** `{width}` (e.g., 1280px)
- **Columns:** {columns} (e.g., 12)
- **Gutter:** `{gutter}` (e.g., 24px)

### Breakpoints
| Name | Width | Notes |
|------|-------|-------|
| sm | `{width}` | Mobile landscape |
| md | `{width}` | Tablet |
| lg | `{width}` | Desktop |
| xl | `{width}` | Large desktop |

### Content Widths
- **Prose:** max-width `{width}` for readable text
- **Forms:** max-width `{width}` for form containers
- **Full:** 100% width containers

---

## 6. Animation & Motion

### Transitions
| Property | Duration | Easing |
|----------|----------|--------|
| Colors | `{duration}` | `{easing}` |
| Transform | `{duration}` | `{easing}` |
| Opacity | `{duration}` | `{easing}` |

### Micro-interactions
- **Hover:** {describe hover behavior}
- **Focus:** {describe focus behavior}
- **Active:** {describe active state}

---

## Notes for Stitch

When generating screens with Stitch, reference this document to maintain consistency:

1. **Include design system context** in prompts
2. **Reference color names** (e.g., "primary", "muted") not hex values
3. **Use typography scale names** for consistent sizing
4. **Apply spacing scale** for consistent rhythm

### Example Stitch Prompt Addition

```
Use the design system:
- Primary color for CTAs
- Border radius: {radius}
- Font: {font_family}
- Spacing: 4px base unit
```
