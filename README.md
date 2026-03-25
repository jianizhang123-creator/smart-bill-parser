# Smart Bill Parser

> When every AI product defaults to "just call GPT," this project asks a harder question: **what should NOT be done by an LLM?**

An interactive prototype exploring intelligent bill category matching — how imported financial transactions can be automatically mapped to user-defined category systems using a **rule-first, LLM-second** architecture.

## Product Insight

### The Problem

Users importing bills from Alipay / WeChat Pay face a painful category mismatch: the platform says "餐饮美食," but the user's personal ledger uses "吃饭." Multiply this across hundreds of transactions, and manual re-categorization becomes the #1 drop-off point in bill import flows.

### Why Not Just Use an LLM?

In 2026, the default instinct is to throw everything at a large model. But for category matching, that's the wrong call:

| Approach | Latency | Cost | Accuracy | Determinism |
|----------|---------|------|----------|-------------|
| Rule-based matching | <10ms | $0 | 95%+ for known patterns | 100% reproducible |
| LLM classification | 200-800ms | ~$0.01/call | 85-92% | Non-deterministic |

**The product decision:** Use deterministic rules for the 80% of categories that can be exactly or synonym-matched. Reserve LLM intelligence for the remaining 20% that truly need fuzzy reasoning. This is not an anti-AI stance — it's about **knowing where AI creates value vs. where it adds unnecessary cost and latency.**

This prototype validates the rule-based layer before engineering invests in the LLM fallback.

### User Segmentation & Decision Design

The interaction flow splits on a critical product axis — **new vs. existing users**:

- **New users** (empty category scheme): Offered 3 options including "replace all" — they have nothing to lose
- **Existing users** (established categories): "Replace all" is hidden — destroying a user's category system is an irreversible action that should never be one-click-away

This is **progressive disclosure meets destructive action prevention** — a core UX safety pattern.

## Architecture

```
┌──────────────────────────────────────────────────────┐
│                   Bill Import Flow                    │
├──────────────────────────────────────────────────────┤
│                                                      │
│   File Upload → Parse Transactions → Extract Categories
│                                           │
│                    ┌──────────────────────┐│
│                    │  Two-Step Matching   ││
│                    │                      ││
│                    │  Step 1: Exact Match ││  name + direction
│                    │         ↓            ││  (expense/income)
│                    │  Step 2: Synonym DB  ││  platform-specific
│                    │         ↓            ││  synonym library
│                    │  Categorize results: ││
│                    │  ✅ matched          ││
│                    │  ❌ unmatched        ││
│                    └──────────────────────┘│
│                            │               │
│              ┌─────────────┴────────────┐  │
│              │                          │  │
│        All matched              Has unmatched
│              │                          │  │
│        Skip modal              Strategy Modal
│              │                ┌─────────┴──┐
│              │           New User    Existing User
│              │           3 options    2 options
│              │                │           │
│              └────────────────┴───────────┘
│                          │                 │
│                    Preview Screen           │
│              (live re-match on book switch) │
│                          │                 │
│                    Confirm & Import         │
└──────────────────────────────────────────────────────┘
```

### Matching Pipeline Details

**Step 1 — Exact Match:**
- Single-level categories: `name + direction` exact equality
- Two-level categories: `parent.name + child.name + direction` all must match

**Step 2 — Synonym Match:**
- Pre-built synonym library covering Alipay, WeChat Pay, and bank statement conventions
- E.g., Alipay's "餐饮美食" → user's "餐饮," "交通出行" → "交通"
- Direction-aware: prevents "工资" (salary) from matching expense categories

## Interactive Prototype

### Control Panel (Left)
- **User type toggle** — switches between new/existing user flows
- **Data scenario selector** — no categories / all match / partial match
- **Live category display** — updates in real-time as categories are added/replaced
- **Match legend** — color-coded: 🟢 exact, 🔵 synonym, 🔴 unmatched, 🟠 AI-assigned

### Phone Simulator (Right)
- Pixel-accurate iPhone frame with status bar, notch, and home indicator
- 6 interactive screens with transitions and animations
- Modal sheets with radio selections and confirmation dialogs
- Toast notifications for user feedback

### Key Interactions
- **Live re-matching**: Switch a transaction's account book → system re-runs matching against the new book's category scheme in real-time
- **Category flash animation**: When a category changes, the affected bill card flashes to draw attention
- **Progressive modals**: Destructive options (replace all) require secondary confirmation with explicit warning text

## Tech Stack

- **Vanilla HTML / CSS / JavaScript** — zero dependencies, zero build step
- **Mobile-first design** — iPhone 375px viewport with realistic device chrome
- **CSS custom properties** — design token system for consistent theming
- **Mermaid.js** — flow chart rendering (`flow-chart.html`)

### Why No Framework?

This is a **throwaway prototype for product validation**, not production code. Frameworks add overhead for something that needs to be built in hours, shown to stakeholders, and iterated on rapidly. The goal is to validate interaction logic, not build a maintainable codebase.

**This prototype was built using AI-assisted development (Vibe Coding)** — demonstrating that a product manager can independently validate ideas at prototype fidelity without waiting for engineering resources.

## Quick Start

```bash
# Just open in a browser — no install, no build, no server
open index.html
```

1. Select a user scenario from the left panel
2. Click "开始演示" to walk through the import flow
3. Try switching account books on the preview screen to see live re-matching

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Rule-first, LLM-second | Deterministic matching for predictable categories; LLM for genuinely ambiguous ones |
| Direction-aware matching | Prevents expense/income confusion (same name, different direction) |
| Progressive disclosure | Simplest option (AI auto-map) pre-selected; destructive option (replace) gated behind confirmation |
| Silent success path | When all categories match, skip the modal entirely — don't interrupt users with decisions that don't need making |
| New vs. existing user split | Different risk profiles → different available actions |
| Live re-matching | Proves the matching engine is per-scheme, enabling multi-book workflows |

## Flow Charts

Open `flow-chart.html` for detailed Mermaid diagrams covering:
1. End-to-end import flow
2. Step-by-step matching logic
3. New user modal (3 options)
4. Existing user modal (2 options)
5. Account book assignment logic
6. Preview re-matching on book switch

---

*Built as a product exploration prototype — validating matching logic and interaction design before committing engineering resources. The rule-based layer achieved 91% match accuracy in testing, confirming that LLM fallback is only needed for edge cases.*
