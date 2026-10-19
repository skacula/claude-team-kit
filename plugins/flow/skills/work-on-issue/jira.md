# Working a Jira work item

Use this file when the repo's instructions file says its issues live in Jira (a line such as "Issues are tracked in Jira project `WEB` on `example.atlassian.net`"), or when the user hands you a Jira key (`WEB-123`) or a `https://<site>.atlassian.net/browse/WEB-123` URL. The code, branches and PRs stay on GitHub; only the issue side moves to Jira. Every step and gate in [SKILL.md](SKILL.md) still applies. This file swaps the commands.

It uses the Atlassian CLI, `acli` ([install](https://developer.atlassian.com/cloud/acli/guides/install-acli/)), which works with Jira Cloud. If `acli` isn't installed but your tool has the Atlassian MCP server connected, use its equivalent tools for the same reads and writes.

## 0. Get oriented

- `acli jira auth status` must show the site the work item lives on. If it doesn't, ask the user to run `acli jira auth login --web` (or `acli jira auth switch`). Never read tokens yourself.
- A slice like `M2.3`: `acli jira workitem search --jql 'project = <KEY> AND summary ~ "\"M2.3:\""' --fields key,summary,status --csv`. JQL text search is fuzzy, so keep only the result whose summary starts with `M2.3:`.
- House style: read `gh pr list --state merged --limit 5 --json number,title,headRefName,body` as usual. With no history yet, use the defaults in section 4 below.

## 1. Review the work item

```bash
acli jira workitem view <KEY-N> --fields summary,description,status,issuetype,labels,parent,issuelinks,reporter,created,attachment --json
acli jira workitem comment list --key <KEY-N> --paginate --json
```

- **Already done** means `status.statusCategory.key` is `done`. That includes statuses like "Won't Do", so read the status name before calling it fixed.
- The **milestone** is the parent Epic. Read its description (goal and acceptance) with `acli jira workitem view <EPIC-KEY> --fields summary,description,status`.
- **Depends on** is an `issuelinks` entry of type `Blocks` where this item is the inward side ("is blocked by"). Check that each blocker's status category is `done`.

## 2. Was it already resolved?

```bash
gh search prs --repo <owner/repo> "<KEY-N>" --json number,title,state,closedAt,url
git log --oneline --grep='<KEY-N>[^0-9]' --grep='<KEY-N>$'
git log --oneline --since=<created> -- <files the item concerns>
acli jira workitem search --jql 'project = <KEY> AND statusCategory = Done AND text ~ "<key words>"' --fields key,summary,status --csv
```

Verify against the current code as SKILL.md says. A mention isn't proof.

## 3. Resolved: Product update and close

Jira doesn't turn `#123` into a link, so cite the PR by its URL. End the update with "If you still see the old behaviour, reopen this item and add a screenshot." After the user approves:

```bash
acli jira workitem comment create --key <KEY-N> --body-file <file>
acli jira workitem transition --key <KEY-N> --status "Done" --yes
```

Status names come from the project's workflow. If "Done" is rejected, ask the user which status means done here.

## 4. Not resolved: branch, build, PR

There's no `gh issue develop` for Jira. Put the key in the branch name, the commits and the PR title instead. With the GitHub for Jira app installed on the org, that is what shows them on the work item's Development panel:

```bash
git checkout <default-branch>
git pull --ff-only
git checkout -b <prefix>/<KEY-N>-<short-slug>
```

- Commit: `<Imperative summary> (<KEY-N>)`. PR title: the same.
- PR body: start with `Jira: [<KEY-N>](https://<site>/browse/<KEY-N>)` in place of `Closes #<N>`. The rest follows SKILL.md.
- Merging the PR doesn't close the work item, unless the project has an automation rule for that. Moving it on the board (to In Progress when you start, to In Review when the PR opens) is visible to the team: ask before each move, and use `acli jira workitem transition` as above.

## 5. Follow-ups

```bash
# duplicates
acli jira workitem search --jql 'project = <KEY> AND statusCategory != Done AND text ~ "<key words>"' --fields key,summary,status --csv
# file one, after the user's OK
acli jira workitem create --project <KEY> --type Bug --summary "<symptom, stated plainly>" --description-file <file> --label follow-up --json
```

- Use `--type Bug` for a bug and `Task` (or `Story`) otherwise. `acli jira project view --key <KEY> --json` lists the types the project allows.
- Jira creates a label the first time anyone uses it, and labels are shared across the whole site. Use only labels the project already uses: no new spellings.
- Add `--parent <EPIC-KEY>` only if the follow-up blocks that milestone's acceptance.
- Descriptions are plain text, so Markdown shows as typed. Keep the body to short paragraphs and `- ` lists.
- In the body and the PR's **Follow-ups filed** section, write keys (`WEB-130`), not `#` numbers.
