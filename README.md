# RHPDS Reusable GitHub Workflows

This repository contains reusable GitHub Actions workflows for the RHPDS organization.

## Available Workflows

### 🤖 Jira AI Analysis

Automatically analyzes pull request changes using Claude AI and posts summaries to linked Jira tickets.

**Features:**
- Validates Jira ticket number in PR description
- Analyzes PR diffs with Claude AI
- Posts AI-generated summaries to Jira tickets
- Enforces Jira ticket requirement for all PRs

---

## Setup Instructions

### 1. Copy PR template to your repository

The PR template is automatically detected by GitHub when placed in:
`.github/pull_request_template.md`

**Option A: Copy from this repo**
```bash
curl -o .github/pull_request_template.md \
  https://raw.githubusercontent.com/rhpds/rhpds-workflows/main/.github/pull_request_template.md
```

**Option B: Create manually**
See the template in this repo's `.github/pull_request_template.md`

### 2. Add workflow files to your repository

Create `.github/workflows/jira-update.yml`:

```yaml
name: Update Jira with AI Analysis

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate-ticket:
    uses: rhpds/rhpds-workflows/.github/workflows/validate-jira-ticket.yml@main

  jira-analysis:
    needs: validate-ticket
    uses: rhpds/rhpds-workflows/.github/workflows/jira-ai-update.yml@main
    secrets:
      MAAS_API_KEY: ${{ secrets.PH_MAAS_API_TOKEN }}
      JIRA_API_TOKEN: ${{ secrets.PH_JIRA_API_TOKEN }}
      JIRA_EMAIL: ${{ secrets.PH_JIRA_API_USER }}
```

### 3. Create a PR

When you create a PR, the template will automatically appear. Fill in the Jira ticket:

```markdown
## Jira Ticket
**Jira:** RHCLOUD-1234
```

The workflow will:
1. **Validate** that a Jira ticket is present (fails if missing)
2. **Analyze** the PR changes with Claude AI
3. **Post** a summary to the Jira ticket

---

## Organization Secrets

The following secrets are configured at the organization level and available to all repos:

- `PH_MAAS_API_TOKEN` - API key for MAAS Claude endpoint
- `PH_JIRA_API_TOKEN` - Jira API token
- `PH_JIRA_API_USER` - Jira user email

---

## Example Jira Comment

When a PR is created/updated, Jira receives a comment like:

> 🤖 **AI Analysis of commit `abc123`**
> 
> **Repository:** rhpds/my-project  
> **Commit:** https://github.com/rhpds/my-project/commit/abc123
> 
> ---
> 
> **Summary:** Added input validation to login form to prevent empty username/password submissions.
> 
> **Impact:** Improves security by rejecting invalid login attempts before they reach the backend.
> 
> **Testing:** Verify empty fields show validation errors, test with various invalid inputs.

---

## How It Works

### PR Template
- Placed in `.github/pull_request_template.md` in each repo
- GitHub automatically applies it to all PRs in that repo
- Each repo can have its own template or use the standard one

### Validation Workflow
- Runs on PR open/edit/sync
- Checks for `**Jira:** TICKET-123` format in PR description
- Fails the check if missing (can be made required for merge)

### AI Analysis Workflow
- Runs after validation passes
- Extracts the Jira ticket from PR description
- Gets PR diff and analyzes with Claude
- Posts summary to the Jira ticket

---

## Applying to Specific Repos

The PR template only applies to repos where you add the `.github/pull_request_template.md` file.

**To enable for a repo:**
1. Add `.github/pull_request_template.md`
2. Add `.github/workflows/jira-update.yml`

**To skip a repo:**
- Simply don't add these files

**To customize for a repo:**
- Modify the template file in that repo
- Each repo can have different templates

---

## Backward Compatibility

The AI analysis workflow still supports `.jira` files for direct push events. Priority order:
1. Explicitly provided ticket via workflow input
2. PR description `**Jira:** TICKET-123`
3. `.jira` file in repo (legacy)

---

## Troubleshooting

**PR template not showing?**
- Ensure file is named exactly `.github/pull_request_template.md`
- File must be committed and pushed to the default branch

**Validation failing?**
- Check PR description has `**Jira:** TICKET-123` format
- Ticket must match pattern: `[A-Z]+-[0-9]+`

**Jira not updating?**
- Verify Jira ticket exists and is accessible
- Check the Actions tab for error logs
- Ensure organization secrets are configured

**AI analysis failing?**
- Check that MAAS endpoint is accessible from GitHub runners
- Verify `PH_MAAS_API_TOKEN` secret is valid

---

## Contributing

To modify the reusable workflows:
1. Make changes to `.github/workflows/*.yml`
2. Test with a demo repository
3. Push to `main` branch - all repos using it will automatically get the update
