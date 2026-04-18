Hourly infrastructure health check. Quick sweep — under 90 seconds.

1. Check CI: are any workflows failing across the org?
   `gh run list --repo Molecule-AI/molecule-core --limit 3 --json conclusion,name,headBranch`

2. Check migration sequence: any gaps or duplicate numbers?
   `ls platform/migrations/*.up.sql | sort | tail -10`

3. Check Docker: any dangling images or stale builds?
   `docker system df 2>/dev/null | head -5`

4. Report to Dev Lead if anything needs attention. Otherwise: "infra-clean HH:MM".
