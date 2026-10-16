# Planning in Jira

Use this file when the repo's instructions file says its issues live in Jira (a line such as "Issues are tracked in Jira project `WEB` on `example.atlassian.net`"), or the user asks for the plan in Jira. Sections 2 and 3 of [SKILL.md](SKILL.md) don't change. This file replaces the GitHub commands in sections 1, 4 and 5, and the approval gate still comes before anything is created.

It uses the Atlassian CLI, `acli` ([install](https://developer.atlassian.com/cloud/acli/guides/install-acli/)), which works with Jira Cloud. If `acli` isn't installed but your tool has the Atlassian MCP server connected, use its equivalent tools.

| GitHub | Jira |
|---|---|
| Milestone `M1: <name>`, goal and acceptance in the description | Epic with the same summary and description |
| Issue `M1.2: <title>` in the milestone | Story (or Task) with the same summary, parent = the Epic |
| `Depends on #N` | A `Blocks` link from the blocker to the blocked item, plus `Depends on <KEY-N>` in the description |
| Labels `area:*`, `os:*`, `follow-up`, `phase-gate` | Jira labels with the same names |
| Closed issue | Status category `done` |

## 1. Read what Jira already has

```bash
acli jira auth status
acli jira project view --key <KEY> --json
acli jira workitem search --jql 'project = <KEY> AND issuetype = Epic' --fields key,summary,status --csv --paginate
# then once per Epic (search can't print the parent field, so query by it)
acli jira workitem search --jql 'parent = <EPIC-KEY>' --fields key,summary,status,labels --csv --paginate
```

`project view` lists the work item types the project allows (`issueTypes`). If it has no Epic or Story, ask the user what to use instead. If `auth status` shows another site, ask the user to run `acli jira auth switch`. Never read tokens yourself.

## 4. Create in Jira

As on GitHub, nothing is created before the user approves, and a re-run must never make duplicates. Before you create each item, search for its summary and skip it if one starts with the same `M<m>.<n>:` prefix:

```bash
acli jira workitem search --jql 'project = <KEY> AND summary ~ "\"M1.2:\""' --fields key,summary,status --csv
```

1. **Labels**: there's nothing to create. Jira makes a label the first time it's used, and labels are shared across the whole site, so check the spelling against the labels the project already uses.
2. **Epics**, one per milestone. Write the goal and acceptance to a file first. Descriptions are plain text, so Markdown shows as typed: keep to short paragraphs and `- [ ]` lines.
   ```bash
   acli jira workitem create --project <KEY> --type Epic --summary "M1: <name>" --description-file <file> --json
   ```
3. **Stories**, in milestone order, so that dependency links can name real keys:
   ```bash
   acli jira workitem create --project <KEY> --type Story --parent <EPIC-KEY> --summary "M1.2: <title>" --description-file <file> --label "area:<x>,os:<y>" --json
   acli jira workitem link create --out <BLOCKER-KEY> --in <BLOCKED-KEY> --type Blocks --yes
   ```
4. Write the keys back into the plan file next to each slice (`WEB-42` where the GitHub version has `#42`), and set its **Tracking** line to say the slices are Jira work items in project `<KEY>`.
5. Ask about committing the plan file, as in SKILL.md.

Report the Epic URLs (`https://<site>/browse/<EPIC-KEY>`) and the number of stories created per Epic.

## 5. Status mode

- Read the Epics and their children as in section 1, plus recently merged PRs: `gh pr list --state merged --limit 30 --json number,title,headRefName,mergedAt,url`. A PR belongs to the work item whose key is in its title or branch name.
- A slice is done when its status category is `done`. A merged PR whose work item isn't done yet is drift: list those items, and **ask** before moving them (`acli jira workitem transition --key <KEY-N> --status "Done" --yes`, with the project's own status name).
- The rest of the drift checks are the same as on GitHub. A Story whose parent changed is the Jira version of "an issue whose milestone changed".
- An Epic with no open children: check its acceptance was demonstrated, then **ask** before you move the Epic to done.
