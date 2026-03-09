# GitLab to Jira Migrator

A Python tool that migrates GitLab issues (both open and closed) to Jira. Handles descriptions, comments, attachments, linked issues, and sub-tasks. Converts GitLab Markdown (including emoji shortcodes) to Jira Wiki markup and supports resuming interrupted migrations via persistent mapping.

## Features

- **Full Issue Migration:** Migrates both open and closed GitLab issues.
- **Comments & Attachments:** Migrates comments and attachments. Attachments exceeding Jira's size limit (20 MB) are skipped and replaced with fallback links.
- **Markdown Conversion:** Converts GitLab Markdown to Jira Wiki markup using the [mistletoe](https://github.com/miyuchina/mistletoe) library with a custom renderer.
- **Emoji Support:** Converts GitLab emoji shortcodes to Unicode emoji.
- **Linked Issues & Sub-tasks:** Migrates linked issues and sub-tasks via GitLab's links API.
- **Persistent Mapping:** Stores issue mappings in a JSON file so interrupted migrations can resume without duplicating work.
- **Jira Epic Assignment:** Optionally assigns migrated parent issues to a specified Jira Epic.
- **Issue Type Mapping:** Creates issues as "Bug" if the GitLab `issue_type` is `incident`, otherwise "Task".
- **Text Truncation:** Truncates descriptions and comments exceeding Jira's 32,767-character limit and appends a link to the original GitLab content.
- **Label Filtering:** Selectively migrate issues using `--skip-labels` or `--include-labels-only`.
- **GitLab Issue Marking:** Optionally adds an `imported-to-jira` label and a Jira link comment to each migrated GitLab issue.
- **GitLab Issue Closing:** Optionally closes GitLab issues after successful migration.

## Requirements

- Python 3.11+
- [uv](https://github.com/astral-sh/uv)

## Installation

```bash
git clone https://github.com/your-fork/GitLab2Jira.git
cd GitLab2Jira
uv sync
```

## Usage

```bash
uv run gitlab2jira \
    --gitlab-url https://gitlab.example.com \
    --gitlab-token YOUR_GITLAB_TOKEN \
    --gitlab-project-id 334 \
    --jira-url https://yourcompany.atlassian.net \
    --jira-user your.email@example.com \
    --jira-api-token YOUR_JIRA_API_TOKEN \
    --jira-project-key PROJ \
    --jira-epic-key EPIC-123 \
    --mark-migrated \
    --close-issues
```

You can also pass a namespace path instead of a numeric project ID:

```bash
--gitlab-project-id mygroup/myproject
```

### Command-Line Arguments

| Argument | Required | Description |
|---|---|---|
| `--gitlab-url` | Yes | URL of your GitLab instance |
| `--gitlab-token` | Yes | GitLab access token (requires `api` scope if using `--mark-migrated` or `--close-issues`, otherwise `read_api`) |
| `--gitlab-project-id` | * | GitLab project ID or path (e.g. `R8/connectivity`) |
| `--gitlab-group-id` | * | GitLab group ID — migrates all projects in the group |
| `--jira-url` | Yes | URL of your Jira instance |
| `--jira-user` | Yes | Your Jira email address |
| `--jira-api-token` | Yes | Your Jira API token |
| `--jira-project-key` | Yes | Jira project key where issues will be created |
| `--jira-epic-key` | No | Jira Epic key to assign all migrated parent issues to |
| `--mapping-file` | No | JSON file to store migration mapping (default: `issue_mapping.json`) |
| `--skip-labels` | No | Skip issues that have any of these labels (mutually exclusive with `--include-labels-only`) |
| `--include-labels-only` | No | Only migrate issues that have at least one of these labels (mutually exclusive with `--skip-labels`) |
| `--mark-migrated` | No | Add `imported-to-jira` label and a Jira link comment to each migrated GitLab issue |
| `--close-issues` | No | Close each GitLab issue after successful migration |

*Either `--gitlab-project-id` or `--gitlab-group-id` must be provided.

### How It Works

1. **Fetching Issues:** Retrieves all issues from the specified GitLab project or group, sorted by IID.
2. **Filtering:** Skips already-migrated issues (via mapping file) and applies any label filters.
3. **Markdown & Emoji Conversion:** Converts descriptions and comments from Markdown to Jira Wiki markup, including emoji shortcode → Unicode.
4. **Attachments:** Downloads attachments from GitLab and uploads them to Jira. Files over 20 MB are replaced with fallback links.
5. **Issue Type & Status:** Issues are created as "Bug" (incident), "Sub-task" (child issue), or "Task". Status is derived from GitLab labels and state.
6. **User Mapping:** Maps GitLab users to Jira accounts by public email. Falls back gracefully if no match is found.
7. **Persistent Mapping:** Saves GitLab→Jira key mappings after each issue so migrations can be safely resumed.
8. **Post-migration:** Optionally marks and/or closes the original GitLab issue.

### Label Filtering Examples

```bash
# Only migrate issues tagged for migration
uv run gitlab2jira ... --include-labels-only gitlab-to-jira
# Skip trash
uv run gitlab2jira ... --skip-labels trash wontfix duplicate
```

### Resuming an Interrupted Migration

The mapping file (`issue_mapping.json` by default) tracks which issues have already been migrated. Simply re-run the same command and already-migrated issues will be skipped automatically.

## Contributing

Contributions, issues, and feature requests are welcome! Please open an issue or submit a pull request on GitHub.
