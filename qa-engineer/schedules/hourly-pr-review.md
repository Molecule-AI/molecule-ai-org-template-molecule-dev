Independent QA work cycle. Find PRs to review and issues to test. Do NOT wait for delegation.

STEP 1 — FIND PRS NEEDING REVIEW:
  gh pr list --repo Molecule-AI/molecule-core --state open --json number,title,author,reviews --jq '.[] | select(.author.login == "app/molecule-ai") | "#\(.number) \(.title | .[0:60])"'
  For each PR without a QA review comment, review it: read the diff, check for bugs, missing tests, security issues.

STEP 2 — POST REVIEW:
  For each PR you review, post a comment with [qa-agent] tag.
  If blocking issues found, clearly state "Verdict: Block" with the specific fix needed.
  If clean, state "Verdict: Approve" with what you verified.

STEP 3 — CHECK OPEN ISSUES FOR TEST GAPS:
  gh issue list --repo Molecule-AI/molecule-core --state open --json number,title --jq '.[] | select(.title | test("test|coverage|QA|bug|regression"; "i"))'
  If test coverage issues exist, write tests and open a PR.

STEP 4 — FOLLOW UP ON YOUR OWN REVIEWS:
  Check if any of your previous blocking reviews have been addressed but you haven't re-reviewed.
  If the dev fixed your findings, post an updated review.

STEP 5 — REPORT what you reviewed, approved, or blocked.

RULES: All PRs target staging. Check molecule-controlplane PRs too.
