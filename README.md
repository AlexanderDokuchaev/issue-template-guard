# issue-template-guard

A GitHub Action that checks whether newly opened or edited issues follow your
[issue form templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository#creating-issue-forms).

When an issue is missing required sections, the action:

- applies a configurable **label** (default `template-missing`)
- posts a configurable **comment**.

When the issue is edited to include every required section, the label is
removed and the comment is marked resolved. The action is idempotent — it edits
its own comment instead of posting duplicates.

## How it works

The required sections are derived automatically from your issue form YAML files
in `.github/ISSUE_TEMPLATE/`. Every field with `validations.required: true`
becomes a required section heading. There is no separate rule list to keep in
sync — update your templates and the guard follows.

## Usage

Create a workflow in the repository you want to protect, e.g.
`.github/workflows/issue-template-guard.yml`:

```yaml
name: Issue Template Guard

on:
  issues:
    types: [opened, edited, reopened]

permissions:
  contents: read
  issues: write

jobs:
  guard:
    runs-on: ubuntu-latest
    steps:
      - uses: AlexanderDokuchaev/issue-template-guard@main
        with:
          label_name: template-missing
          message: |
            Thanks for opening this issue :wave:

            It looks like this issue does not follow one of issue templates.
            Issues that don't follow the template may not be triaged and closed after some time.
```

## Inputs

| Input                | Required | Default                  | Description                                             |
| -------------------- | -------- | ------------------------ | ------------------------------------------------------- |
| `label_name`         | no       | `template-missing`       | Label applied to issues that do not follow a template   |
| `message`            | no       | *(see `action.yml`)*     | Comment posted on issues that do not follow a template  |
| `token`              | no       | `${{ github.token }}`    | Token used to read templates and manage labels/comments |

# Permissions

The calling workflow must grant `issues: write` so the action can add labels and
comments.

## Notes

- A non-compliant issue does **not** fail the check run — it is only labeled and
  commented. The action fails when templates cannot be read or when no issue form
   with required fields is found in `.github/ISSUE_TEMPLATE`
- The `label_name` label is created automatically on first use if it does not
  exist. Pre-create it to control its color and description.
- Only fields of type `input`, `textarea`, `dropdown`, and `checkboxes` marked
  `required: true` count as required sections. `markdown` blocks are ignored.

## Limitations

Templates are read with a small built-in line parser (no full YAML library), so
keep `type`, `label`, and `validations.required` on their own lines - the layout
the GitHub form editor produces. Block scalars (`label: |`), anchors, and aliases
are not supported for labels.

```yaml
body:
  - type: textarea
    attributes:
      label: Steps to reproduce
    validations:
      required: true
```

## License

See [LICENSE](LICENSE).
