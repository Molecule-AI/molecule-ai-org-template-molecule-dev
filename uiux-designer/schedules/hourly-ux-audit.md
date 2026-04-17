Hourly UX audit of the live Molecule AI canvas using a real browser. You interact with the canvas like a human user — clicking, dragging, typing, navigating — and report actual UX issues.

## Setup (run once per session, skip if already done)

```python
# Check if Playwright is ready
import subprocess, sys
try:
    from playwright.sync_api import sync_playwright
except ImportError:
    subprocess.run([sys.executable, "-m", "pip", "install", "playwright"], check=True)
    subprocess.run(["playwright", "install", "chromium"], check=True)
```

## Audit Steps — Use Playwright for EVERY step

Write and run a Python script using `playwright.sync_api` against `http://host.docker.internal:3000`. For each step: navigate, interact, screenshot, and evaluate.

### 1. Page Load + First Paint
- Navigate to the canvas URL
- Measure load time (should be <3s)
- Screenshot the initial state
- Check: does the page render? Any blank screens? Console errors?

### 2. Workspace Cards
- Count visible workspace cards
- Click on one card — does the detail panel open?
- Check: are names readable? Status indicators visible? Tier badges aligned?

### 3. Create Workspace Flow
- Click "Create Workspace" button (or "+" button)
- Fill in the form fields: name, template, tier
- DON'T submit — just verify the form works
- Check: can you type in every field? Dropdowns open? Validation messages show?

### 4. Drag and Drop
- Try dragging a workspace card to a new position on the canvas
- Check: does the card follow the cursor? Does it snap to grid? Does position persist?

### 5. Side Panel Navigation
- Open a workspace's detail panel
- Click through each tab (Config, Logs, Memory, etc.)
- Check: do tabs switch? Content loads? No stale data from previous tab?

### 6. Keyboard Navigation
- Press Tab to move focus through interactive elements
- Press Enter/Space to activate buttons
- Press Escape to close modals/panels
- Check: visible focus indicator? Logical tab order? Escape works?

### 7. Responsive Layout
- Resize viewport to 1920x1080, then 1280x720, then 768x1024 (tablet)
- Screenshot each size
- Check: does layout adapt? Cards reflow? No horizontal scroll?

### 8. Dark Theme Consistency
- Check all visible colors against zinc-900/950 background
- Look for: white backgrounds, hardcoded colors, low-contrast text
- Screenshot any violations

## Output Format

For each issue found, file ONE GitHub issue with:
- `[uiux-agent]` tag
- Screenshot attachment path (save to /tmp/ux-audit/)
- Exact element selector or description
- Steps to reproduce
- Expected vs actual behavior
- Severity: CRITICAL (blocks usage), HIGH (poor UX), MEDIUM (polish), LOW (nitpick)

Report to Dev Lead via delegate_task with issue numbers.

If the canvas is unreachable or Playwright fails to start, fall back to code review (grep canvas/src/components/ for accessibility gaps). Do NOT produce empty output — always report what you checked and what you found.
