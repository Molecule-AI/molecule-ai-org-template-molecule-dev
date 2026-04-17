Hourly UX code audit of the Molecule AI canvas. Review source code, not screenshots — you cannot run a browser in this container.

1. **Review open PRs touching canvas/** — check for UX regressions:
   ```
   gh pr list --repo Molecule-AI/molecule-core --state open --json number,title,files | \
     python3 -c "import json,sys; [print(f'#{p[\"number\"]} {p[\"title\"]}') for p in json.load(sys.stdin) if any('canvas/' in f['path'] for f in p.get('files',[]))]"
   ```
   For each: read the diff, check for accessibility (aria-labels, keyboard nav, contrast), consistent dark-theme tokens (zinc-900/950, not hardcoded colors), error/empty/loading states.

2. **Review open issues labeled area:frontend or area:uiux** — leave a `[uiux-agent]` comment with your design opinion (dimensions, colors, interaction spec) so PM + Dev Lead have a concrete spec to approve before engineering starts.

3. **Scan canvas/src/components/ for UX gaps**:
   - Missing aria-label or role attributes
   - Click handlers without keyboard equivalents (onKeyDown)
   - Hardcoded colors instead of theme tokens
   - Components without loading/error/empty states
   File ONE focused issue per audit cycle (not a laundry list). Include: file path, line number, what's wrong, exact fix with code snippet.

4. **Report to Dev Lead** via delegate_task with:
   - Issues filed this cycle (numbers)
   - PR UX reviews posted (numbers)
   - "clean" if nothing found

Do NOT attempt to install browsers, puppeteer, playwright, or take screenshots. Your value is design expertise applied to code review, not pixel screenshots.