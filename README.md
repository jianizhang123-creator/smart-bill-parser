# Smart Bill Parser

> An interactive prototype for intelligent bill category matching -- built to explore how imported financial transactions can be automatically mapped to user-defined category systems.

## Problem

When users import bills from payment platforms (Alipay, WeChat Pay, etc.), category naming conventions differ significantly from the user's personal accounting categories. Manual re-categorization is tedious and error-prone, especially for large transaction histories.

## Solution

A two-step intelligent matching system:

1. **Exact Match** -- Direct name + direction (expense/income) matching against the user's existing category hierarchy (supports both L1 and L2 categories)
2. **Synonym Match** -- Fuzzy matching via a built-in synonym library covering major payment platforms (e.g., Alipay's "餐饮美食" maps to user's "餐饮")

For unmatched categories, users are guided through a decision flow:

- **Use existing categories** -- AI auto-maps unrecognized transactions to the closest existing category
- **Add new categories** -- Append unmatched categories to the user's current scheme
- **Replace entire scheme** -- Swap the user's categories with the imported bill's category system (available for new users only)

## Architecture

The matching pipeline works as follows:

```
Bill Import → File Parsing → Category Detection
                                    ↓
                          ┌─────────────────────┐
                          │  For each category:  │
                          │  1. Exact match      │
                          │  2. Synonym match    │
                          └─────────────────────┘
                                    ↓
                    ┌───────────────┴───────────────┐
                    │                               │
              All matched?                   Has unmatched
                    │                               │
            Silent categorize          Show strategy modal
                    │                    (AI / Add / Replace)
                    ↓                               ↓
                         Preview Screen
                              ↓
                     Confirm & Import
```

The preview screen also supports **live re-matching**: when a user switches a transaction's target account book, the system re-runs the matching pipeline against the new book's category scheme in real time.

## Flow Chart

Open `flow-chart.html` to view detailed Mermaid-rendered flow diagrams covering:

1. **Overall import flow** -- End-to-end from file selection to import completion
2. **Matching logic** -- Step-by-step exact + synonym matching for single and dual-column categories
3. **New user modal** -- Three-option decision flow (AI / Add / Replace) with secondary confirmation
4. **Existing user modal** -- Two-option decision flow (AI / Add)
5. **Account book assignment** -- How bills are routed to the correct book and category scheme
6. **Preview re-matching** -- What happens when a user reassigns a bill to a different account book

## Tech Stack

- Vanilla HTML / CSS / JavaScript
- No frameworks -- lightweight, zero-dependency prototype
- Mobile-first design (iPhone viewport with realistic device chrome)
- Mermaid.js for flow chart rendering (CDN-loaded)

## Quick Start

1. Open `index.html` in any browser
2. Use the left-side control panel to select a scenario:
   - **User type**: New user vs. existing user (affects available options)
   - **Bill data**: No categories, all matched, or partially matched
3. Click "开始演示" (Start Demo) to walk through the flow
4. On the preview screen, try switching account books (existing user mode) to see real-time re-matching

## UI Overview

The prototype renders inside a realistic iPhone frame with:

- **Control panel** (left) -- Scenario selector, current category scheme display, and a color-coded legend
- **Phone simulator** (right) -- Screens for welcome, file import, matching progress, preview, and various modals
- **Live category panel** -- The left panel updates in real time as categories are added or replaced

Match results are color-coded throughout:
- Green = exact match
- Blue = synonym match
- Red = unmatched
- Orange = AI-assigned or newly added

## Key Design Decisions

- **Two-step matching** reduces false positives while maintaining high recall
- **Direction-aware matching** prevents expense/income category confusion (e.g., "工资" as income vs. expense)
- **Progressive disclosure** in the user choice flow -- simplest option (AI) is pre-selected, destructive option (replace) requires secondary confirmation
- **Silent success path** -- when all categories match, the system skips the modal entirely and goes straight to preview
- **Live re-matching on book change** demonstrates that the matching engine runs per-scheme, not globally

---

Built as a product exploration prototype to validate the matching logic and interaction design before engineering implementation.
