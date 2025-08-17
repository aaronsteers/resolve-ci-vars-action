# 🔧 Resolve CI Variables Action

⚡️ Run your CI workflows with multiple triggers. Stop writing custom bash variable resolvers, and end your if/then expression nightmares.

This action simplifies resolving variables, evaluating expressions, and defining fallback behaviors. The action supports three types of resolution:

This action is designed to make your workflows more maintainable by:

- Coalescing the first non-empty value from a list of inputs
- Evaluating Jinja2 expressions
- Emitting both flat outputs and structured results
- Optionally logging results and writing a step summary

---

## 🔤 Variable Resolution

Learn more about each resolution type:

- [Direct Assignment](#direct-assignment)
  - E.g. `var1=my value`
- [Jinja Expression Assignment](#jinja-expression-assignment)
  - E.g. `var1=true ? 'great' : 'not great'`
- [Standard CI Vars](#standard-ci-vars)
  - Standard CI Vars. Automatic resolution of canned variables and defined inputs. We handle a number of common use cases for you, saving you time, code lines, and troubleshooting time.

### Direct Assignment

Direct variable assignment is provided with the `static_inputs` input arg.

The `static_inputs` parameter takes variable assignments in `key=value` format:

```yml
    with:
      static_inputs:
        answer=42
        question=What is the meaning of life?
```

More likely, you will use GitHub expressions to resolve these variables:

```yml
    with:
      static_inputs:
        answer=${{ github.inputs.something || github.inputs.else }}
```

The benefit of this approach is that subsequent action steps can simply pull the
resolved variable, rather than repeating the expression everywhere it is needed.

### Jinja Expression Assignment

Jinja expressions provide a powerful balance between expressiveness and simplicity.

You can leave off the normal `{{` and `}}` wrappers and provide any valid jinja2 expression.

```yml
    with:
      jinja_inputs: |
        answer=40 + 2
        question='What is the meaning' + 'of life?'
```

Just as with static inputs, you can use GitHub expressions to inject variables into the
jinja expression:

```yml
    with:
      jinja_inputs: |
        token='${{ inputs.custom_token }}' or '${{ secrets.GITHUB_TOKEN }}'
        environment='${{ inputs.environment }}' or ('main' if '${{ github.ref }}' == 'refs/heads/main' else 'dev')
        api_url='${{ vars.CUSTOM_API_URL }}' or 'https://api.example.com'
```

#### 📖 Jinja2 Syntax Reference

For help with Jinja2 expressions, check out these resources:

- [Official Jinja2 Documentation](https://jinja.palletsprojects.com/en/3.1.x/templates/) - Complete reference
- [Jinja2 Cheat Sheet](https://devhints.io/jinja) - Quick syntax reference

### Standard CI Vars

The action automatically resolves common CI variables from GitHub context, eliminating the need for complex expressions like `${{ github.event.pull_request.head.repo.full_name || github.repository }}` throughout your workflows.

#### Standard CI Variables Reference

| Variable | Description | Example Value |
|----------|-------------|---------------|
| **Resolved Variables (always the effective context)** |
| `resolved-git-ref` | Full Git ref (`refs/heads/...` or `refs/tags/...`) | `refs/heads/feature/new-connector` |
| `resolved-git-branch` | Short branch name (e.g. `main`, `feature/foo`) | `feature/new-connector` |
| `resolved-git-sha` | Commit SHA | `abc123...` |
| `resolved-git-tag` | Tag name (if applicable) | `v1.0.0` |
| `resolved-repo-name` | Repository name (e.g. `my-repo`) | `airbyte` |
| `resolved-repo-owner` | Repository owner (user or org) | `airbytehq` |
| `resolved-repo-name-full` | Owner + name (e.g. `myorg/my-repo`) | `airbytehq/airbyte` |
| `resolved-git-branch-url` | URL to the branch in GitHub | `https://github.com/airbytehq/airbyte/tree/feature/new-connector` |
| `resolved-git-commit-url` | URL to the commit in GitHub | `https://github.com/airbytehq/airbyte/commit/abc123...` |
| **PR Source Variables (for PR workflows)** |
| `pr-source-git-ref` | Git ref of the source (PR head) | `refs/heads/feature/new-connector` |
| `pr-source-git-branch` | Branch name of the source | `feature/new-connector` |
| `pr-source-git-sha` | SHA of the source commit | `abc123...` |
| `pr-source-repo-name` | Source repo name | `airbyte` |
| `pr-source-repo-owner` | Source repo owner | `contributor` |
| `pr-source-repo-name-full` | Full source repo name (owner/name) | `contributor/airbyte` |
| `pr-source-git-branch-url` | URL to the source branch | `https://github.com/contributor/airbyte/tree/feature/new-connector` |
| `pr-source-git-commit-url` | URL to the source commit | `https://github.com/contributor/airbyte/commit/abc123...` |
| `pr-source-is-fork` | Whether the source repo is a fork | `true` |
| **PR Target Variables (for PR workflows)** |
| `pr-target-git-ref` | Git ref of the target (PR base) | `refs/heads/main` |
| `pr-target-git-branch` | Branch name of the target | `main` |
| `pr-target-git-sha` | SHA of the target commit | `def456...` |
| `pr-target-git-tag` | Tag name, if PR targets a tag | `` |
| `pr-target-repo-name` | Target repo name | `airbyte` |
| `pr-target-repo-owner` | Target repo owner | `airbytehq` |
| `pr-target-repo-name-full` | Full target repo name (owner/name) | `airbytehq/airbyte` |
| `pr-target-git-branch-url` | URL to the target branch | `https://github.com/airbytehq/airbyte/tree/main` |
| `pr-target-git-commit-url` | URL to the target commit | `https://github.com/airbytehq/airbyte/commit/def456...` |
| **Additional Resolved Metadata** |
| `pr-number` | Pull request number (if applicable) | `12345` |
| `pr-url` | URL to the pull request | `https://github.com/airbytehq/airbyte/pull/12345` |
| `pr-title` | Title of the pull request | `feat: Add new connector` |
| `comment-id` | ID of the triggering comment (if applicable) | `987654321` |
| `comment-url` | URL to the triggering comment (if applicable) | `https://github.com/airbytehq/airbyte/issues/12345#issuecomment-987654321` |
| `run-id` | GitHub Actions run ID | `123456789` |
| `run-url` | URL to the GitHub Actions run | `https://github.com/airbytehq/airbyte/actions/runs/123456789` |
| `is-pr` | Boolean: whether the current context is a PR | `true` |

#### Usage Example

```yaml
- name: Resolve CI variables
  id: vars
  uses: aaronsteers/resolve-ci-vars-action@v1
  with:
    static_inputs: |
      custom_var=my_value

- name: Use resolved variables
  run: |
    echo "PR Number: ${{ steps.vars.outputs.pr-number }}"
    echo "Resolved Branch: ${{ steps.vars.outputs.resolved-git-branch }}"
    echo "Repository: ${{ steps.vars.outputs.resolved-repo-name-full }}"
    echo "All variables: ${{ steps.vars.outputs.custom }}"
```

#### Workflow Dispatch Auto-Detection

The action automatically detects and resolves common workflow_dispatch inputs:

- **PR inputs**: `pr` or `pr-number` - Automatically resolves PR context and updates resolved variables to point to PR head
- **Comment inputs**: `comment-id` - Resolves comment metadata for slash command workflows  
- **Issue inputs**: `issue-id` or `issue-number` - Resolves issue context and constructs URLs

```yaml
# Example workflow_dispatch with auto-detected inputs
on:
  workflow_dispatch:
    inputs:
      pr-number:
        description: 'PR number to operate on'
        required: false
      comment-id:
        description: 'Comment ID that triggered this'
        required: false
      issue-number:
        description: 'Issue number'
        required: false

jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - uses: aaronsteers/resolve-ci-vars-action@v1
        id: vars
      # All workflow_dispatch inputs are now available as resolved variables
      - run: echo "Operating on PR: ${{ steps.vars.outputs.pr-number }}"
```

**Key Features:**
- **Smart resolved context**: `resolved-*` variables always point to the effective working context (PR head for PRs, current branch otherwise)
- **Explicit PR source/target**: Separate `pr-source-*` and `pr-target-*` variables for fine-grained PR workflows
- **URL generation**: Automatic GitHub URLs for branches, commits, PRs, and comments
- **Fork-aware**: Properly distinguishes between source and target repositories in fork scenarios
- **Context detection**: `is-pr` boolean and other metadata for workflow logic
- **Workflow dispatch auto-detection**: Automatically detects and resolves `pr`/`pr-number`, `comment-id`, and `issue-id`/`issue-number` inputs

---

## ✨ Features

- ✅ **Auto-coalesce static inputs**
- 🧠 **Evaluate Jinja2 expressions**
- 📦 **Expose custom results in reusable outputs**: `var1`, `var2`, `var3`
- 🧾 **Structured JSON output**: `custom`
- 📊 **Optional GitHub Step Summary output**

---

## 🚀 Usage

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Resolve variables
        id: vars
        uses: your-org/resolve-ci-vars-action@v1
        with:
          static_inputs: |
            username=${{ inputs.username }}
            default_username=guest
          jinja_inputs: |
            team='${{ inputs.team }}' or 'default_team'
          var1: "priority or 'low'"
          log_outputs: true

      - name: Use resolved values
        run: |
          echo "Username: ${{ steps.vars.outputs.result }}"
          echo "Team: ${{ steps.vars.outputs.result }}"
          echo "Priority: ${{ steps.vars.outputs.var1 }}"
          echo "Custom object: ${{ steps.vars.outputs.custom }}"
```

---

## 🔠 Inputs

| Name            | Description                                                                                         | Required | Default |
|----------------|-----------------------------------------------------------------------------------------------------|----------|---------|
| `static_inputs`| Variable assignments in key=value format (multiline string)                                        | ❌       |         |
| `jinja_inputs` | Jinja2 expressions to evaluate (e.g. `user or default_user`)                                       | ❌       |         |
| `log_outputs`  | Whether to log resolved values to the console and step summary                                      | ❌       | `false` |
| `non_sensitive`| Alias for `log_outputs`                                                                            | ❌       | `false` |

---

## 📤 Outputs

| Output Name | Description                                                    |
|-------------|----------------------------------------------------------------|
| `custom`    | JSON-encoded object with all resolved values                  |
| `var1`      | Custom user-defined output variable 1                         |
| `var2`      | Custom user-defined output variable 2                         |
| `var3`      | Custom user-defined output variable 3                         |
| `{variable-name}` | Individual outputs for each resolved variable (e.g., `pr-number`, `resolved-git-branch`) |

---

## 📚 Examples

### Basic fallback with static inputs

```yaml
with:
  static_inputs: |
    username=${{ inputs.username }}
    default_username=fallback
```

### Use Jinja2

```yaml
with:
  jinja_inputs: |
    team='${{ inputs.team }}' or 'unknown'
```

---

## 🔐 Security

Sensitive values should not be logged unless you are confident they are masked.
Use `log_outputs: true` or `non_sensitive: true` only for safe-to-print variables.

---

## 📦 License

MIT License — feel free to fork, extend, and share!

---

## 🛠 Maintained by

Your Name or Org — [github.com/your-org](https://github.com/your-org)
