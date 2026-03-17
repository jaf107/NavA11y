# Dynamic Accessibility Testing Tool

A purely dynamic accessibility testing tool that performs automated WCAG compliance checks (Focus Behavior) on web pages using browser automation (Playwright), unlike static analysis tools which cannot detect interaction-based violations.

**Current Focus**: WCAG 2.4 Navigable — Focus Visible, Focus Appearance, Focus Order, and Focus Not Obscured.

## Features

- ✅ **Dynamic Analysis**: Testing via Playwright interaction
- ✅ **Focus Visible (2.4.7 AA)**: Detects missing focus indicators
- ✅ **Focus Appearance (2.4.13 AAA)**: Validates stricter AAA requirements (contrast & size)
- ✅ **Focus Order (2.4.3 A)**: Detects logical/visual order mismatches and focus traps
- ✅ **Focus Not Obscured (2.4.11 AA / 2.4.12 AAA)**: Detects elements hidden by overlapping content
- ✅ **Visual Reports**: HTML reports with annotated screenshots

## Installation

### Prerequisites
- Node.js 16+
- pnpm (recommended) or npm

### Setup

```bash
# Install dependencies
pnpm install

# Install browsers
npx playwright install chromium
```

## Usage

### Test a URL

```bash
node run-check.js --url https://example.com
```

### Test a Local File

```bash
node run-check.js --file ./fixtures/test-page.html
```

## Reports

Generated in `reports/`:
- `index.html`: Human-readable report with screenshots
- `results.json`: Machine-readable results

## Supported WCAG Checks

| Check | Level | Description |
|-------|-------|-------------|
| **2.4.3 Focus Order** | A | Tab order matches visual/DOM order |
| **2.4.7 Focus Visible** | AA | Visible focus indicator present |
| **2.4.11 Focus Not Obscured** | AA | Element not fully hidden |
| **2.4.12 Focus Not Obscured** | AAA | Element not partially hidden |
| **2.4.13 Focus Appearance** | AAA | 3:1 contrast, 2px minimum |

## Datasets

### 1. Semrush Validation Dataset (Real-World)
A dataset of 27 high-traffic websites derived from the Semrush top 30 list.
- **List:** `evaluation/dataset.json`
- **Run Benchmark:** `node evaluation/run-all-benchmark.mjs`

### 2. Focus Behavior Dataset (Synthetic Test Cases)
A comprehensive set of 142 accessibility test cases from the [GDS accessibility-tool-audit](https://github.com/alphagov/accessibility-tool-audit).
- **Files:** `dataset/focus-behavior-dataset/tests/`
- **Metadata:** `dataset/focus-behavior-dataset/tests.json`
- **Run Audit:** `node evaluation/run-gds-evaluation.mjs`
