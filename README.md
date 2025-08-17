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
| `static_inputs`| Comma-separated list of input variable names to coalesce                                           | ❌       |         |
| `jinja_inputs` | Jinja2 expression to evaluate (e.g. `user or default_user`)                                         | ❌       |         |
| `var1`-`var3`  | Optional named expressions. Will be exposed as `var1`-`var3` outputs.                  | ❌       |         |
| `log_outputs`  | Whether to log resolved values to the console and step summary                                      | ❌       | `false` |
| `non_sensitive`| Alias for `log_outputs`                                                                            | ❌       | `false` |

---

## 📤 Outputs

| Output Name | Description                                                    | Input Aliases              |
|-------------|----------------------------------------------------------------|----------------------------|
| `result`    | The resolved value from `static_inputs` or `jinja_inputs`     | `static_inputs`, `jinja_inputs` |
| `var1`      | Custom user-defined output variable 1                         | `var1`                 |
| `var2`      | Custom user-defined output variable 2                         | `var2`                 |
| `var3`      | Custom user-defined output variable 3                         | `var3`                 |
| `custom`    | JSON-encoded object with all resolved custom values           | N/A                        |

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
