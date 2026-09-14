---
name: curiotech-careercafe
scope: codebase-curiotech-careercafe
description: >-
  Official visual design system, typography, theme models, multi-surface UI protocols, and frozen landing page specifications for CurioTech and CareerCafe products (v3.2).
---

# CurioTech & CareerCafe Visual Design System Standard (v3.2)

> [!IMPORTANT]
> **CODEBASE-SPECIFIC SCOPE**: This skill is strictly specific to **CurioTech** and **CareerCafe** projects (`CurioTech-CareerCafe`). It enforces the frozen v3.2 visual design system across marketing pages, Question Banks, code playgrounds, and interview experiences.

---

## 1. Core Brand Identity & Palette

The CareerCafe visual brand is rooted in **Orange + Sage + teal-cast Charcoal**. It is designed to feel calm, editorial, technical, and evidence-led—deliberately avoiding playful, generic edtech aesthetics.

### Decision / Frozen Choices

| Dimension | Decision |
|---|---|
| **Product font** | IBM Plex Sans |
| **Code / data / timers** | IBM Plex Mono |
| **Theme model** | System / Light / Dark |
| **Light canvas** | `#F8F7F4` warm off-white |
| **Dark canvas** | `#10181C` deep teal-charcoal |
| **Primary action** | Orange (`#BC4A1E`) |
| **Practice / progress state** | Sage (`#7F9B6D` graphical, `#4F6640` text) |
| **Structure / trust** | Teal-cast Charcoal (`#17252B`) |
| **Overall aesthetic** | Calm, editorial, technical, evidence-led |

### Color Tokens & Semantic Assignments

| Semantic Role | Token Name | Light Value | Dark Value | Immutable Rule |
|---|---|---|---|---|
| **Primary Action** | `color-action` | `#BC4A1E` | `#BC4A1E` | White text on fill. Orange stays consistent across both themes. |
| **Primary Hover** | `color-action-hover`| `#A83E17` | `#A83E17` | Hover always darkens, never lightens. |
| **Practice / State** | `color-state-sage` | `#7F9B6D` (graphical)<br>`#4F6640` (text) | `#9FB58D` (graphical)<br>`#B7C9A8` (text) | Used exclusively for practice/progress states and badges. |
| **Structure / Trust**| `color-charcoal` | `#17252B` | `#F2F4F3` (ink)<br>`#17252B` (surface) | Deliberately teal-cast. NEVER substitute a neutral grey-black. |
| **Focus Ring** | `color-focus` | `#17252B` | `#FFFFFF` | 2px solid ring with 2px offset. |

### Color Semantics Directives
- **Orange = Action**: Reserved exclusively for primary interactive calls to action, submit buttons, and forward progression.
- **Sage = State / Progress**: Communicates learning status, practice progress, and positive validation. Sage is a shape/surface, NEVER normal body or link text.
- **Charcoal = Structure / Trust**: Deliberately teal-cast `#17252B`. Establishes page headers, structural dividers, dark conversion bands, and trust anchors.

---

## 2. Surface & Text Tokens

| Token | Light Value | Dark Value | Usage |
|---|---|---|---|
| **Canvas** | `#F8F7F4` | `#10181C` | Page base background (warm off-white / deep teal-charcoal). |
| **Primary Surface** | `#FFFFFF` | `#17252B` | Main cards, panels, content reading containers. |
| **Elevated Surface**| `#FFFFFF` | `#1D2D33` | Modals, flyouts, popovers, elevated cards. |
| **Sunken Surface** | `#EFEDE7` | `#0D1518` | Inset wells, code blocks, secondary sections. |
| **Practice / Sage** | `#EEF3E8` | `#1B2A1E` | Practice mode accents, completed card surfaces. |
| **Static Code** | `#F2F4F0` | `#1A282D` | Read-only syntax and code snippet containers. |
| **Input Background** | `#FFFFFF` | `#132025` | Form inputs, textareas, code editors. |
| **Strong Ink** | `#17252B` | `#F2F4F3` | Headings, bold emphasis, titles. |
| **UI Body** | `#2C3B42` | `#D9E0DD` | Form labels, control text, dashboard copy. |
| **Long-Form Reading**| `#33444B` | `#C9D2CF` | Question Bank content, case studies, explanations. |
| **Secondary Text** | `#4A5D64` | `#AEB6B9` | Subtitles, helper text, inactive options. |
| **Metadata** | `#5B6E75` | `#95A3A7` | Timestamps, counters, tags, caption copy. |

---

## 3. Typography System & Scales

Product and UI typography strictly uses **IBM Plex Sans**. Code, data tables, metrics, and timers use **IBM Plex Mono**. **No third typeface is permitted.**

### Marketing Typography Scale

| Role | Desktop | Mobile | Weight / Tracking |
|---|---|---|---|
| **Hero H1** | 60 / 63px | 32 / 38px | 600, tracking `-0.03em` |
| **H2** | 40 / 48px | 26 / 32px | 600, tracking `-0.024em` |
| **H3** | 26 / 34px | 21 / 28px | 600, tracking `-0.012em` |
| **Lead Paragraph** | 20 / 32px | 18 / 28px | 400 |
| **Body** | 17 / 28px | 16 / 26px | 400 |
| **Button** | 16 / 20px | 16 / 20px | 600 |

### Product UI Typography Scale

| Role | Desktop | Mobile | Weight / Tracking |
|---|---|---|---|
| **Screen Title** | 24 / 32px | 20 / 28px | 600 |
| **Section Head** | 18 / 24px | 16 / 22px | 600 |
| **UI Label** | 14 / 18px | 14 / 18px | 500 |
| **Body / Explanation** | 15 / 24px | 15 / 24px | 400 |
| **Data / Code / Monospace** | 13 / 20px | 13 / 20px | 400 (IBM Plex Mono) |
| **Caption / Meta** | 12 / 16px | 12 / 16px | 400 |

### Readability Guardrails
- **Reading Measure**: In Question Bank explanations and long-form editorial views, line lengths must never exceed **680px**.
- **Line Height**: Body line height must remain `1.55`–`1.65` (`24px` on `15px` font; `28px` on `17px` font).

---

## 4. Theme System (System / Light / Dark)

- **3-State Model**: `system` (follows OS preference), `light`, and `dark`.
- **Zero-Flash Hydration**: The active theme class (`light` or `dark`) must be resolved inline before DOM render (e.g. inline script in `<head>`).
- **Surface Elevation**: In Dark Mode, elevation is communicated by lightening the teal-charcoal surface, not by drop shadows.

---

## 5. Multi-Surface UI Architecture

CareerCafe comprises 4 distinct surfaces sharing one unified token system:

1. **Marketing Landing Surface**: High contrast, generous `96px` section rhythm, prominent Orange CTAs, Charcoal conversion footer.
2. **Question Bank Surface**: Optimized for sustained reading. 17/30 problem types, clean tabs, max `680px` content measure, subtle borders.
3. **Interactive Playground Surface (SQL / Python)**: Split-pane layout, sunken `#0D1518` editor background, IBM Plex Mono syntax, clean execution console.
4. **Interview Mode Surface**: Distraction-free interface. Sage feedback and validation are suppressed during active interview sessions to prevent cognitive bias; only structural Charcoal and action Orange are visible.

---

## 6. Frozen Landing Page Specification

The landing page follows a **frozen 12-section architecture**:
1. **Sticky Global Navigation**: Logo, primary nav, theme toggle, and Orange "Start Practicing" CTA.
2. **Hero Section**: H1 formula ("The Technical Interview Practice Engine"), proof badge, primary/secondary CTAs, 16:10 real screenshot.
3. **Trust & Social Proof Bar**: Logos and real candidate placement metrics.
4. **Interactive Problem Preview**: Real 17/30 Question Bank preview card.
5. **Multi-Track Curriculum**: Data Engineering, System Design, Full-Stack, and Machine Learning tracks.
6. **Execution Playground Showcase**: Real SQL/Python code editor and execution sandbox preview.
7. **Interview Simulation Mode**: Mock interview flow with timing and rubric assessment.
8. **Candidate Analytics & Progress**: Mastery scorecards and progress tracking.
9. **Company Benchmark Comparison**: Real interview rubric calibration against top tech firms.
10. **Testimonials & Case Studies**: Real evidence-led reviews with candidate profiles.
11. **Pricing & Plans**: Transparent tiers with feature comparison.
12. **Conversion Footer**: Teal-cast Charcoal band with final primary CTA.

---

## 7. Visual Invariants & Anti-Patterns

1. **NO Neon or Generic Edtech Palettes**: Never introduce purple, bright blue, or gradient fills.
2. **NO Card-in-Card Nesting**: Cards must not be nested more than 1 level deep.
3. **NO Decorative Levitation**: Ban hover-elevation transforms (e.g. `translateY(-4px)`).
4. **NO Bubbly Corner Radii**: Standard card radius is `8px`; interactive buttons are `6px`. Never use pill shapes for primary cards.
5. **NO Font Leakage**: Only IBM Plex Sans and IBM Plex Mono.

---

## 8. Reference Document

The full raw specification is preserved in [`references/CAREERCAFE_DESIGN_SYSTEM_v3.2.md`](references/CAREERCAFE_DESIGN_SYSTEM_v3.2.md).
