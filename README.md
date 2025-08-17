# 🔧 Resolve Variables Action

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

Enable automatic resolution of common CI variables with `standard_ci_vars: true`. This eliminates the need for complex expressions like `${{ github.event.pull_request.head.repo.full_name || github.repository }}` throughout your workflows.

#### Standard CI Variables Reference

| Variable | Description | Resolves From | Example Value |
|----------|-------------|---------------|---------------|
| **Repository & Git Context** |
| `repo` | Target repository (base repo in owner/name format) | `github.repository` or `github.event.pull_request.base.repo.full_name` | `airbytehq/airbyte` |
| `repo-owner` | Target repository owner/organization | `github.repository_owner` | `airbytehq` |
| `repo-name` | Target repository name only | Repository name from base repo | `airbyte` |
| `head-ref` | Source branch/ref being merged (null for issues) | `github.head_ref` or `github.ref_name` | `feature/new-connector` |
| `base-ref` | Target branch/ref for merge (null for issues) | `github.base_ref` or default branch | `main` |
| `head-repo` | Source repository (important for forks, null for issues) | `github.event.pull_request.head.repo.full_name` | `contributor/airbyte` |
| `base-repo` | Target repository (same as `repo`, null for issues) | `github.event.pull_request.base.repo.full_name` | `airbytehq/airbyte` |
| `head-sha` | Head commit SHA (null for issues) | `github.event.pull_request.head.sha` | `abc123...` |
| `base-sha` | Base commit SHA (null for issues) | `github.event.pull_request.base.sha` | `def456...` |
| `gitref` | Smart ref resolution (null for issues) | PR head ref when `pr` input exists, otherwise `github.ref_name` | `feature/branch` |
| **Pull Request Variables** |
| `pr-number` | Pull request number | `github.event.pull_request.number` or `github.event.issue.number` | `12345` |
| `pr-title` | Pull request title | `github.event.pull_request.title` | `feat: Add new connector` |
| `pr-author` | Pull request author username | `github.event.pull_request.user.login` | `contributor` |
| `pr-draft` | Whether PR is draft | `github.event.pull_request.draft` | `true` |
| `is-pr-from-fork` | Whether PR is from a fork | `github.event.pull_request.head.repo.fork` | `true` |
| **Issue Variables** |
| `issue-number` | Issue number (equals pr-number for PRs) | `github.event.issue.number` or `github.event.pull_request.number` | `12345` |
| `issue-title` | Issue title | `github.event.issue.title` or `github.event.pull_request.title` | `Bug: Fix connector issue` |
| `issue-author` | Issue author username | `github.event.issue.user.login` or `github.event.pull_request.user.login` | `reporter` |
| `issue-type` | Type of issue context | Detected from event type | `pull_request` or `issue` |
| **Comment Variables** |
| `comment-id` | Comment ID that triggered workflow | `github.event.comment.id` | `987654321` |
| `comment-author` | Comment author username | `github.event.comment.user.login` | `maintainer` |
| `comment-body` | Comment body text | `github.event.comment.body` | `/test connector` |
| **Workflow Context** |
| `trigger-type` | Event that triggered workflow | `github.event_name` | `pull_request` |
| `actor` | User who triggered the workflow | `github.actor` | `contributor` |
| `run-id` | Workflow run ID | `github.run_id` | `123456789` |
| `run-number` | Workflow run number | `github.run_number` | `42` |

#### Usage Example

```yaml
- name: Resolve CI variables
  id: vars
  uses: aaronsteers/resolve-vars-action@v1
  with:
    standard_ci_vars: true
    static_inputs: |
      custom_var=my_value

- name: Use resolved variables
  run: |
    echo "PR Number: ${{ steps.vars.outputs.pr-number }}"
    echo "Head Ref: ${{ steps.vars.outputs.head-ref }}"
    echo "Repository: ${{ steps.vars.outputs.repo }}"
    echo "All variables: ${{ steps.vars.outputs.all }}"
```

**Key Features:**
- **Smart gitref resolution**: Handles "run from main, operate on PR branch" pattern for slash commands
- **Issue-aware**: Git refs are null for issue events since issues don't have branches
- **Fork-friendly**: Properly distinguishes between head and base repositories
- **Trigger-agnostic**: Works with pull_request, workflow_dispatch, issue_comment, and more

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
        uses: your-org/resolve-vars-action@v1
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
| `standard_ci_vars` | Whether to automatically resolve standard CI variables from GitHub context                     | ❌       | `false` |
| `log_outputs`  | Whether to log resolved values to the console and step summary                                      | ❌       | `false` |
| `non_sensitive`| Alias for `log_outputs`                                                                            | ❌       | `false` |

---

## 📤 Outputs

| Output Name | Description                                                    |
|-------------|----------------------------------------------------------------|
| `all`       | JSON-encoded object with all resolved values                  |
| `var1`      | Custom user-defined output variable 1                         |
| `var2`      | Custom user-defined output variable 2                         |
| `var3`      | Custom user-defined output variable 3                         |
| `{variable-name}` | Individual outputs for each resolved variable (e.g., `pr-number`, `head-ref`) |

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
