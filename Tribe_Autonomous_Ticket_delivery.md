# Tribe Autonomous Ticket Delivery - AI-Assisted Development Workflow

This document outlines the standard workflow for autonomous ticket delivery using Claude Code to create Jira tickets, branches, commits, and pull requests for the TribeCRM platform.

## Prerequisites

- Access to Jira (efficy.atlassian.net)
- GitHub access to efficy-sa repositories
- Local development environment setup

## Complete Workflow

### 1. Create Jira Ticket

**Before creating any code changes, create a Jira ticket first.**

#### Required Fields:
- **Project**: TRI (Growth BU)
- **Issue Type**: Task (or Bug/Story as appropriate)
- **Summary**: Clear, concise description of the change
- **Description**: Include the following sections:
  - Brief explanation of the issue/change (include the source if available - e.g. the SonarQube or Sentry issue that is fixed)
  - Changes made (bullet points)
  - Technical details (file paths, line numbers)
  - **⚠️ IMPORTANT**: Add note: "This is AI-generated code using Claude Code"
  - Test coverage checklist
  - Related PR link (to be added later)

#### Required Metadata:
- **Epic Link**: TRI-2901 ("AI-generated tickets" for all Claude Code work)
- **Team**: Assign to reviewer's team (e.g., "Tribe - Sales" for Anthony Tilleul)
- **Assignee**: The reviewer who will review the PR
- **Developed by**: Set to the developer (Johann Norsa)
- **Product**: Tribe CRM

#### Jira Commands:
```bash
# After ticket creation, note the ticket number (e.g., TRI-2902)
# Then update the ticket fields using Atlassian MCP tools
```

### 2. Create Branch

**Branch Naming Convention**: `TRI-####-description-of-change`

```bash
# Pull latest master-rsbuild (or main/master depending on repo)
git checkout master-rsbuild
git pull origin master-rsbuild

# Create new branch
git checkout -b TRI-2902-fix-css-property-file-viewer
```

### 3. Implement Changes

- Make the necessary code changes
- Review the code for correctness
- Ensure all imports and dependencies are correct

### 4. Test Changes with Playwright

**⚠️ CRITICAL: Testing must be completed BEFORE committing and creating PR**

#### Testing Steps:
1. **Start Development Server** (if not already running):
   ```bash
   cd c:/git/tribecrm-app
   npm start
   ```

2. **Validate with Playwright MCP**:
   - Use Playwright MCP tools to open the application
   - Navigate to relevant pages that use the modified code
   - Check browser console for new errors
   - Take screenshots if needed for documentation
   - Verify the specific component/feature works correctly

3. **Console Error Check**:
   ```bash
   # Use Playwright to check console logs
   # Look for errors introduced by the changes
   # Pre-existing errors are OK, new errors are blockers
   ```

4. **Functional Testing**:
   - Test the specific feature/component modified
   - Verify no regressions in related functionality
   - Confirm hot-reload works properly

**Example Playwright Testing**:
```bash
# Navigate to application
playwright_navigate --url http://localhost:3000

# Take initial screenshot
playwright_screenshot --name "before-test"

# Check console for errors
playwright_console_logs --type error

# Navigate to relevant component
# (specific steps depend on the change)

# Take final screenshot
playwright_screenshot --name "after-test"
```

**Testing Checklist**:
- [ ] Dev server running without errors
- [ ] Application loads successfully
- [ ] No new console errors introduced
- [ ] Modified component/feature works correctly
- [ ] Hot-reload functioning properly
- [ ] Screenshots captured (if applicable)

### 5. Commit Changes

**Commit Message Format**: Use [Conventional Commits](https://www.conventionalcommits.org/)

```
<type>(<scope>): [TRI-####] <description>

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

#### Example:
```bash
git add src/@Component/Generic/FileViewer/FileViewer.module.scss

git commit -m "$(cat <<'EOF'
fix(file-viewer): [TRI-2902] fix css property marginTop to margin-top

Changed marginTop to margin-top to fix SonarQube BLOCKER issue.
CSS properties must use kebab-case, not camelCase.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

**IMPORTANT**:
- Always pass commit messages via HEREDOC for proper formatting
- Include the Jira ticket number in square brackets
- Add Co-Authored-By line to credit AI assistance
- Use `--no-verify` flag only if absolutely necessary to bypass hooks

### 6. Push to Remote

```bash
git push -u origin TRI-2902-fix-css-property-file-viewer
```

### 7. Create Pull Request

**⚠️ CRITICAL: Base Branch Must Be master-rsbuild**

Always verify the PR targets **master-rsbuild**, NOT master.

**PR Title Format**: `TRI-#### - Description`

**PR Body Format**:
```markdown
## Summary
- Bullet point 1
- Bullet point 2
- Bullet point 3

## Test plan
- [ ] Tested locally with hot-reload
- [ ] Verified changes in browser
- [ ] Ran automated tests

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

#### GitHub CLI Command:
```bash
gh pr create --base master-rsbuild --title "TRI-2902 - Fix CSS property in FileViewer" --body "$(cat <<'EOF'
## Summary
- Fixed SonarQube BLOCKER issue css:S4654
- Changed `marginTop: 10px;` to `margin-top: 10px;` in FileViewer.module.scss
- CSS properties must use kebab-case, not camelCase

## Test plan
- [x] Source code verified with correct CSS syntax
- [x] Local deployment tested with hot-reload
- [x] Automated testing with Playwright

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" --reviewer anthony.tilleul@efficy.com
```

### 8. Update Jira Ticket

After PR creation, update the Jira ticket:

1. **Add PR Link**: Add comment to Jira ticket with GitHub PR URL
2. **Transition Status**: Move ticket to "Review" state (It should be assigned first to "In Progress" and then to "Review")
3. **Verify Fields**:
   - Epic: TRI-2901
   - Team: Reviewer's team
   - Assignee: Reviewer
   - Developed by: Developer (Johann Norsa)

## Workflow Summary Checklist

- [ ] Create Jira ticket with all required fields and AI-generated code notice
- [ ] Note the ticket number (TRI-####)
- [ ] Pull latest main branch (master-rsbuild/main/master depending on repo)
- [ ] Create branch: `TRI-####-description`
- [ ] Implement changes
- [ ] **Test changes with Playwright MCP** (BEFORE committing)
  - [ ] Dev server running without errors
  - [ ] No new console errors introduced
  - [ ] Modified component/feature verified working
- [ ] Commit with Conventional Commits format including [TRI-####]
- [ ] Push branch to remote
- [ ] **Create PR targeting master-rsbuild** (use `--base master-rsbuild`)
  - [ ] Verify base branch is master-rsbuild, NOT master
  - [ ] Add correct title format and reviewer
- [ ] Add PR link to Jira ticket
- [ ] Transition Jira ticket to Review status
- [ ] Verify all Jira fields are correct (Epic, Team, Assignee, Developed by)

## Important Notes

### Git Safety
- NEVER run destructive git commands without explicit user request
- NEVER skip hooks unless explicitly requested
- NEVER force push to main/master
- Always create NEW commits rather than amending (unless explicitly requested)
- Prefer adding specific files by name rather than `git add -A` or `git add .`

### Jira Requirements
- All Claude Code work must be linked to Epic TRI-2901
- Always mark tickets as AI-generated in the description
- Ticket must be in Review state before reviewer can access it
- Team assignment must match the reviewer's team

### GitHub Requirements
- **ALWAYS target master-rsbuild as the base branch for PRs** (NOT master)
- Follow branch naming convention strictly: `TRI-####-description`
- Use Conventional Commits format
- Include Co-Authored-By line for AI assistance
- Add reviewer to PR immediately upon creation
- Verify base branch before creating PR: `gh pr create --base master-rsbuild`

### Testing Requirements
- **ALWAYS test changes with Playwright MCP BEFORE committing**
- Testing must validate no new errors are introduced
- Pre-existing errors/warnings are acceptable
- Focus testing on the specific component/feature modified
- Capture screenshots for documentation when applicable
- Document test results in PR description

## Example End-to-End Workflow

```bash
# 1. Create Jira ticket TRI-2902 (via Atlassian MCP)
# 2. Pull and create branch
git checkout master-rsbuild
git pull origin master-rsbuild
git checkout -b TRI-2902-fix-css-property-file-viewer

# 3. Make changes
# (edit files)

# 4. Test with Playwright (BEFORE committing)
# - Start dev server if not running: npm start
# - Use Playwright MCP to navigate to http://localhost:3000
# - Check console logs for errors
# - Test the specific component/feature
# - Take screenshots if needed
# - Verify no new errors introduced

# 5. Commit changes (ONLY after testing is complete)
git add src/@Component/Generic/FileViewer/FileViewer.module.scss
git commit -m "$(cat <<'EOF'
fix(file-viewer): [TRI-2902] fix css property marginTop to margin-top

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"

# 6. Push to remote
git push -u origin TRI-2902-fix-css-property-file-viewer

# 7. Create PR (IMPORTANT: Use --base master-rsbuild)
gh pr create --base master-rsbuild --title "TRI-2902 - Fix CSS property in FileViewer" \
  --body "$(cat <<'EOF'
## Summary
- Fixed SonarQube BLOCKER issue css:S4654
- Changed marginTop to margin-top in FileViewer.module.scss

## Test plan
- [x] Tested locally
- [x] Verified changes

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)" \
  --reviewer anthony.tilleul@efficy.com

# 8. Update Jira ticket (via Atlassian MCP)
# - Add PR link as comment
# - Transition to "In Progress" (transition ID 7)
# - Transition to "Review" (transition ID 11 - "Development finished")
```

## Resources

- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [Jira API Documentation](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)

## Reviewers

Common reviewers and their teams:
- **Anthony Tilleul** (anthony.tilleul@efficy.com) - Team: Tribe - Sales

---

Last updated: 2026-02-03
**Latest changes**:
- Added mandatory Playwright testing step (Step 4) before committing
- Emphasized that PRs must target **master-rsbuild**, NOT master (added `--base master-rsbuild` to all PR creation commands)
