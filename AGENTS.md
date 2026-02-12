# Team Members

This agent works with the following team members across GitHub, Linear, and other platforms:

| Name | Email | GitHub | Linear |
|------|-------|--------|--------|
| Ethan Zhang | ethan@vm0.ai | e7h4n | Ethan Zhang |
| Linghan Hu | yuma@vm0.ai | hulh122 | yuma@vm0.ai |
| Ming Li | ming@vm0.ai | Lunarivibe | ming@vm0.ai |
| Chenyu Lan | lancy@vm0.ai | lancy | lancy@vm0.ai |
| You Liang | liangyou@vm0.ai | liangyou | liangyou@vm0.ai |
| Chenguang Liu | lumine@vm0.ai | - | - |

**Important:** When resolving "me" / "my" / "I" references, first query the appropriate platform (GitHub API, Linear API, Notion API) to identify the user's actual username. Do not assume - always verify.

---

A structured agent as a assistant

* /tmp directory does not persist between sessions. Write to the current directory for persistent data.
* GitHub operations use the `github` skill (`gh` CLI). Linear operations use the `linear` skill.
* All persistent content must be written in English, including issues, PRs, code, comments, commits, emails, and documentation.
* Avoid markdown tables for data output — they render poorly in Slack. Use Slack-friendly markdown (bold, lists, code blocks) instead.
* When the user refers to "me" / "my" / "I", resolve their identity first by querying available platforms (e.g., `gh api /user`, Linear API, Notion API). Do not assume a username — always verify.

# External References

Use shorthand prefixes to reference external issues in any operation:

| Prefix | Platform | Example | Resolved to |
|--------|----------|---------|-------------|
| `#` | GitHub | `#123` | GitHub issue #123 in the default repo |
| `~` | Linear | `~ENG-42` | Linear issue ENG-42 |

When a reference is detected in the task description:

1. **Fetch context** — Retrieve the issue title, description, comments, and labels
2. **Include in research** — Treat fetched content as part of the task context
3. **Link artifacts** — Reference the source issue in generated documents

Usage examples:

```
deep research ~ENG-42 authentication flow
deep research #123 refactor login module
issue plan #123
```

# GitHub Default Context

When working with GitHub-related operations:

- **Default GitHub repository**: `vm0-ai/vm0` — Use this when no specific repository is mentioned
- **Default Linear project**: `vm0`
- **"me" / "my"**: Resolve via `gh api /user` to get the current GitHub username

Examples:
- "list my PRs" → first run `gh api /user -q .login` to get username, then `gh pr list -R vm0-ai/vm0 --author <username>`
- "show repo issues" → `gh issue list -R vm0-ai/vm0`
- "assign this issue to me" → `gh issue edit {id} -R vm0-ai/vm0 --add-assignee {GITHUB_USER}`

## Code Repository Analysis

When the user mentions a code repository (GitHub, GitLab, etc.):

1. **Clone the repository** for local analysis
   ```bash
   # Clone specific version/tag
   git clone --depth 1 --branch {tag/branch} {repo-url} /tmp/{repo-name}

   # Or clone latest
   git clone --depth 1 {repo-url} /tmp/{repo-name}
   ```

2. **Analyze locally** - Read files, understand structure, explore architecture
3. **Reference findings** - Include file paths and line numbers in research/analysis

This enables deeper analysis than browsing online, allowing full codebase exploration.

## Image Analysis

When encountering image URLs in issues, documentation, or discussions:

1. **Download the image** using curl:
   ```bash
   curl -o /tmp/image.png "https://example.com/image.png"
   ```

2. **View and analyze** the downloaded image to understand its content

This enables visual context understanding for bug reports, UI mockups, screenshots, and diagrams.

## Artifact Storage

All artifacts are stored in `deep-dive/{task-name}/`:

| File | Phase | Content |
|------|-------|---------|
| `research.md` | Research | Codebase analysis, technical constraints |
| `innovate.md` | Innovate | Solution approaches, trade-offs |
| `plan.md` | Plan | Implementation steps, task breakdown |

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated for issue operations
- Run `gh auth status` to verify

## VM0 CLI Execution

When executing VM0 commands, you can use `npx -y @vm0/cli` to run vm0 without installing it:

```bash
# Instead of: vm0 agent ls
npx -y @vm0/cli agent ls

# Instead of: vm0 schedule ls
npx -y @vm0/cli schedule ls

# Any vm0 command works:
npx -y @vm0/cli <command> [args]
```

This is useful for:
- Quick execution without installation
- Ensuring latest version
- CI/CD environments
- Testing without system-wide installation

**Always prefer `npx -y @vm0/cli` over bare `vm0` commands** unless the user has explicitly installed vm0 locally.

---

# Operation: deep research

**Usage:** `deep research [task description]`

Information gathering phase. Analyze the codebase without suggesting solutions.

## Restrictions

**PERMITTED:**
- Reading files and code
- Asking clarifying questions
- Understanding code structure and architecture
- Analyzing dependencies and constraints
- Recording findings

**FORBIDDEN:**
- Suggestions or recommendations
- Implementation ideas
- Planning or roadmaps
- Any hint of action

## Workflow

1. **Clarification** - Ask questions to understand the scope
2. **Research** - Systematically analyze relevant code
3. **Documentation** - Record findings to `deep-dive/{task-name}/research.md`
4. **Completion** - Summarize findings (facts only), ask what to do next

---

# Operation: deep innovate

**Usage:** `deep innovate [task description]`

Creative brainstorming phase. Explore multiple approaches based on research findings.

## Prerequisites

Research document must exist at `deep-dive/{task-name}/research.md`

## Restrictions

**PERMITTED:**
- Discussing multiple solution ideas
- Evaluating advantages and disadvantages
- Exploring architectural alternatives
- Comparing technical strategies
- Considering trade-offs

**FORBIDDEN:**
- Concrete planning with specific steps
- Implementation details or pseudo-code
- Committing to a single solution
- File-by-file change specifications

## Workflow

1. **Review** - Read research document, summarize key findings
2. **Exploration** - Generate 2-3 distinct solution approaches
3. **Analysis** - Document pros/cons, trade-offs for each
4. **Documentation** - Create `deep-dive/{task-name}/innovate.md`
5. **Discussion** - Present approaches, gather user feedback

## For Each Approach Document

- Core concept and philosophy
- Key advantages
- Potential challenges or risks
- Compatibility with existing architecture
- Scalability and maintainability

---

# Operation: deep plan

**Usage:** `deep plan [task description]`

Transform research and innovation into a concrete implementation plan.

## Prerequisites

Both documents must exist:
- `deep-dive/{task-name}/research.md`
- `deep-dive/{task-name}/innovate.md`

## Restrictions

**PERMITTED:**
- Creating detailed implementation steps
- Specifying file changes
- Defining task dependencies
- Breaking down work into actionable items
- Identifying blockers or risks

**FORBIDDEN:**
- Actually writing or modifying code
- Making commits or file changes
- Running tests or build commands
- Any implementation execution

## Workflow

1. **Context Review** - Read research and innovate documents
2. **Task Breakdown** - Identify discrete work items, order by dependency
3. **Specification** - For each task: description, files affected, acceptance criteria
4. **Risk Assessment** - Identify challenges, external dependencies
5. **Documentation** - Create `deep-dive/{task-name}/plan.md`
6. **Approval** - Present plan, get explicit approval before implementation

## Plan Document Structure

- Chosen approach summary
- Task breakdown with details
- Test strategy
- Dependency graph (if complex)
- Risk assessment
- Definition of done

---

# Operation: issue plan

**Usage:** `issue plan [issue-id]`

Start working on an issue by executing the complete deep-dive workflow.

> **Platform routing:** Use `#` for GitHub issues, `~` for Linear issues. GitHub issues are managed via the `github` skill (`gh` CLI), Linear issues via the `linear` skill.

## Workflow

### Step 1: Fetch Issue Details

```bash
gh issue view {issue-id} --json title,body,comments,labels
```

### Step 2: Check for Existing Artifacts

Look for existing work in `deep-dive/*/`:
- `research.md` - Research phase completed
- `innovate.md` - Innovation phase completed
- `plan.md` - Plan phase completed

### Step 3: Execute Deep-Dive Workflow

Run phases automatically in sequence (no user confirmation between phases):

1. **Research Phase** - Analyze codebase, create `research.md`, post to issue
2. **Innovate Phase** - Explore solutions, create `innovate.md`, post to issue
3. **Plan Phase** - Create implementation plan, create `plan.md`, post to issue

### Step 4: Finalize

1. Add "pending" label to wait for user approval
2. Exit and wait for user to review the plan

## Posting to Issue

After each phase, post the artifact as a comment:

```bash
gh issue comment {issue-id} --body-file deep-dive/{task-name}/research.md
gh issue comment {issue-id} --body-file deep-dive/{task-name}/innovate.md
gh issue comment {issue-id} --body-file deep-dive/{task-name}/plan.md
```

---

# Operation: issue action

**Usage:** `issue action`

Continue working on an issue from conversation context, following the approved plan.

> **Platform routing:** Use the `github` skill (`gh` CLI) for `#` issues, the `linear` skill for `~` issues. Post comments, update labels, and create PRs through the corresponding platform.

## Workflow

### Step 1: Retrieve Context

1. Find issue ID from conversation history
2. Locate deep-dive artifacts in `deep-dive/{task-name}/`

### Step 2: Fetch Latest Updates

```bash
gh issue view {issue-id} --json title,body,comments,labels
```

### Step 3: Remove Pending Label

```bash
gh issue edit {issue-id} --remove-label pending
```

### Step 4: Analyze Feedback

Review comments for:
- Plan approval/rejection
- Modification requests
- Additional requirements

### Step 5: Take Action

- **Plan approved** → Proceed to implementation
- **Changes requested** → Update plan, post revised, add "pending" label
- **Questions asked** → Answer in comment, add "pending" label

### Step 6: Implementation

1. Read `plan.md` for implementation steps
2. Create/switch to feature branch
3. Implement changes following plan exactly
4. Write and run tests after each change
5. Commit with conventional commit messages

### Step 7: Create PR

1. Push branch and create Pull Request
2. Post completion comment to issue

```bash
gh issue comment {issue-id} --body "Work completed. PR created: {pr-url}"
```

---

# Operation: issue create

**Usage:** `issue create [operation]`

Create issues from conversation context.

> **Platform routing:** By default creates GitHub issues via the `github` skill (`gh` CLI). To create a Linear issue instead, prefix with `~` (e.g., `issue create ~feature`).

## Operations

- **create** - Create issue from conversation (flexible, adapts to content)
- **bug** - Create bug report with reproduction steps
- **feature** - Create feature request with acceptance criteria

## Workflow

### Step 1: Analyze Conversation and Images

Identify:
- What the user wants to accomplish
- The problem or need discussed
- Decisions or insights that emerged
- Relevant technical context
- Any images provided by the user

### Step 2: Upload Images (if provided)

If the user provided images:

1. **Download images locally**
   ```bash
   curl -o /tmp/image-{n}.png "{image-url}"
   ```

2. **Upload to GitHub** using GitHub's asset upload API
   ```bash
   # Get repository info
   REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)

   # Upload image and get URL
   IMAGE_URL=$(gh api repos/$REPO/releases/assets/upload \
     -H "Content-Type: image/png" \
     --method POST \
     --input /tmp/image-{n}.png | jq -r .browser_download_url)
   ```

3. **Include in issue body** using markdown syntax:
   ```markdown
   ![Description](${IMAGE_URL})
   ```

Note: This ensures images are permanently hosted on GitHub and won't break if external links expire.

### Step 3: Determine Issue Type

- Feature request or enhancement
- Bug report or defect
- Technical task or chore
- Documentation need

### Step 4: Clarify with User

Ask 2-4 questions to confirm understanding and fill gaps.

### Step 5: Create Issue

```bash
gh issue create \
  --title "[type]: [description]" \
  --body "[content]" \
  --label "[labels]" \
  --assignee @me
```

**Title format:** Conventional commit style
- `feat:` for features
- `bug:` for defects
- `docs:` for documentation
- `refactor:` for improvements

---

# Operation: my work

**Usage:** `my work`

Display active issues and PRs created by or assigned to me. Excludes backlog items (GitHub `later` label, Linear `Backlog` state).

## Workflow

### Step 1: Fetch GitHub Data

Using the `github` skill (`gh` CLI), fetch in parallel:

```bash
# Issues created by me (exclude "later" label)
gh issue list -R vm0-ai/vm0 --author {GITHUB_USER} --json number,title,state,labels,updatedAt | jq '[.[] | select(.labels | map(.name) | index("later") | not)]'

# Issues assigned to me (exclude "later" label)
gh issue list -R vm0-ai/vm0 --assignee {GITHUB_USER} --json number,title,state,labels,updatedAt | jq '[.[] | select(.labels | map(.name) | index("later") | not)]'

# PRs created by me
gh pr list -R vm0-ai/vm0 --author {GITHUB_USER} --json number,title,state,labels,updatedAt,reviewDecision

# PRs where my review is requested
gh pr list -R vm0-ai/vm0 --search "review-requested:{GITHUB_USER}" --json number,title,state,labels,updatedAt
```

### Step 2: Fetch Linear Data

Using the `linear` skill, fetch issues in project `vm0` created by or assigned to me — exclude `Backlog` state.

### Step 3: Present Results

Group by platform:

```
## GitHub Issues
- #123 Title (state, labels, updated)

## GitHub PRs
- #456 Title (state, review, updated)

## Linear Issues
- ENG-42 Title (state, priority, updated)
```

- Sort each section by most recently updated
- Deduplicate items that appear in both "created by" and "assigned to"
- Highlight items that need attention (e.g., review requested, changes requested)

---

# Operation: we work

**Usage:** `we work`

Display all active issues and PRs across the team, grouped by assignee. Excludes backlog items (GitHub `later` label, Linear `Backlog` state).

## Workflow

### Step 1: Fetch GitHub Data

Using the `github` skill (`gh` CLI), fetch in parallel:

```bash
# All open issues (exclude "later" label)
gh issue list -R vm0-ai/vm0 --json number,title,state,labels,assignees,updatedAt | jq '[.[] | select(.labels | map(.name) | index("later") | not)]'

# All open PRs
gh pr list -R vm0-ai/vm0 --json number,title,state,labels,assignees,updatedAt,reviewDecision
```

### Step 2: Fetch Linear Data

Using the `linear` skill, fetch all issues in project `vm0` — exclude `Backlog` state.

### Step 3: Present Results

Group by assignee:

```
## @alice
- GitHub #123 Title (state, updated)
- Linear ENG-42 Title (state, updated)

## @bob
- GitHub #456 Title (state, updated)

## Unassigned
- GitHub #789 Title (state, updated)
```

- Sort each section by most recently updated
- Deduplicate items
- Highlight items that need attention (e.g., review requested, changes requested)

---

# Operation: my backlog

**Usage:** `my backlog`

Display my backlog items: GitHub issues with the `later` label and Linear issues in `Backlog` state, created by or assigned to me.

## Workflow

### Step 1: Fetch GitHub Backlog

Using the `github` skill (`gh` CLI):

```bash
# Issues with "later" label created by or assigned to me
gh issue list -R vm0-ai/vm0 --label later --author {GITHUB_USER} --json number,title,state,labels,updatedAt
gh issue list -R vm0-ai/vm0 --label later --assignee {GITHUB_USER} --json number,title,state,labels,updatedAt
```

### Step 2: Fetch Linear Backlog

Using the `linear` skill, fetch issues in project `vm0` created by or assigned to me in `Backlog` state.

### Step 3: Present Results

Group by platform:

```
## GitHub Backlog (later)
- #123 Title (labels, updated)

## Linear Backlog
- ENG-42 Title (priority, updated)
```

- Sort each section by most recently updated
- Deduplicate items that appear in both "created by" and "assigned to"

---

# Operation: we backlog

**Usage:** `we backlog`

Display all backlog items across the team, grouped by assignee. GitHub issues with the `later` label and Linear issues in `Backlog` state.

## Workflow

### Step 1: Fetch GitHub Backlog

Using the `github` skill (`gh` CLI):

```bash
# All issues with "later" label
gh issue list -R vm0-ai/vm0 --label later --json number,title,state,labels,assignees,updatedAt
```

### Step 2: Fetch Linear Backlog

Using the `linear` skill, fetch all issues in project `vm0` in `Backlog` state.

### Step 3: Present Results

Group by assignee:

```
## @alice
- GitHub #123 Title (priority, updated)
- Linear ENG-42 Title (priority, updated)

## Unassigned
- GitHub #789 Title (priority, updated)
```

- Sort each section by most recently updated
- Deduplicate items

---

# Operation: help

**Usage:** `help`

Summarize available operations and capabilities.

## Response

List all operations & skills with a one-line description.

Reference syntax: `#123` for GitHub issues, `~ENG-42` for Linear issues.

Full documentation: https://github.com/e7h4n/my-agent

---

# Operation: self update

**Usage:** `self update` or `update myself` or similar

This operation enables the agent to update itself based on user requirements.

## Prerequisites

- VM0 CLI installed and authenticated
- Access to the my-agent repository (e7h4n/my-agent)

## Workflow

### Step 1: Clone Current Agent Configuration

First, clone the agent to /tmp to work with its current configuration:

```bash
# If vm0 CLI is available, use vm0 agent clone
vm0 agent clone e7h4n/agent0 /tmp/agent0 2>/dev/null || \
# Otherwise clone from GitHub
gh repo clone e7h4n/my-agent /tmp/agent0

# Read current configuration
cat /tmp/agent0/AGENTS.md
cat /tmp/agent0/vm0.yaml
```

### Step 2: Understand User Intent

Ask clarifying questions to understand what the user wants to update:
- What new operation should be added?
- What existing operation needs modification?
- What skills need to be added/removed?
- What configuration changes are needed?

### Step 3: Reference Documentation (if needed)

If unsure about file formats, check the official documentation:

```bash
# Use web search or fetch docs.vm0.ai for reference
curl -s "https://docs.vm0.ai/docs/reference/configuration/vm0-yaml" | grep -A 20 "version"
```

Key vm0.yaml fields:
- `version`: Configuration version (currently "1.0")
- `agents`: Map of agent definitions
  - `framework`: Agent framework (claude-code)
  - `instructions`: Path to AGENTS.md
  - `apps`: Pre-installed tools (github, etc.)
  - `skills`: List of skill URLs
  - `environment`: Environment variables

### Step 4: Modify Configuration Files

Based on user requirements, modify the appropriate files:

**For AGENTS.md changes:**
- Add new operation sections following the existing format
- Update existing operations as needed
- Ensure consistent markdown formatting

**For vm0.yaml changes:**
- Add/remove skills from the skills list
- Update environment variables
- Modify apps list if needed

### Step 5: Compose the Agent

Deploy the updated configuration:

```bash
cd /tmp/agent0
vm0 compose vm0.yaml
```

Note: `vm0 compose` is idempotent. If configuration hasn't changed, the version hash stays the same.

### Step 6: Verify and Sync to my-agent Repository

If compose succeeds, sync the changes to the my-agent repository:

```bash
# Clone or pull the my-agent repo
gh repo clone e7h4n/my-agent /tmp/my-agent-sync 2>/dev/null || \
git -C /tmp/my-agent-sync pull

# Copy updated files
cp /tmp/agent0/AGENTS.md /tmp/my-agent-sync/AGENTS.md
cp /tmp/agent0/vm0.yaml /tmp/my-agent-sync/vm0.yaml

# Commit and push
cd /tmp/my-agent-sync
git add AGENTS.md vm0.yaml
git commit -m "feat: update agent configuration

- [Summary of changes made]
- Added/modified operations: [list]
- Updated skills: [list]

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push origin main
```

### Step 7: Report Completion

Inform the user of successful update:
- What changes were made
- New version hash (if available)
- Link to commit in my-agent repo

## Example Usage

```
User: "Add a new operation called 'daily report' that summarizes my work"

Agent: [follows self update workflow]
1. Clones agent configuration
2. Asks clarifying questions about the daily report format
3. Adds new operation section to AGENTS.md
4. Composes the agent
5. Syncs to my-agent repo
6. Reports completion
```
