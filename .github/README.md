# Issue Exporter Workflow

Use this as a reusable workflow from another repository by following the
[main README](../README.md#use-in-your-repository). There is no need to copy
this directory into the calling repository.

## Run and download

1. Configure the caller workflow and `ISSUE_READ_TOKEN` secret as described in
   the main README. The caller workflow must be on its default branch to appear
   in the manual-run UI. To run directly in this exporter repository instead,
   configure the same secret here; the export will contain this repository's issues.
2. Open **Actions > Export issues > Run workflow**.
3. Enter a required GitHub issue search query, for example:

   ```text
   label:Severity-1,Severity-2 label:service:npm -label:project:npm_cli
   ```

4. Open the successful run and select its **issue-export-...** artifact, or use
   the download link in the run summary.
5. Extract the downloaded ZIP and open `index.html`.

The artifact contains an index, individual issue pages with descriptions and all
issue comments, a stylesheet, and `issues.json`. Issue text and comments work
offline. Media remains linked to its original location: images, videos, and other
external resources are not downloaded and can require internet access and GitHub
authorization. The exporter itself calls only GitHub's API.

Artifacts are stored by GitHub Actions, not committed or published to Pages.
They expire after **7 days**, can be deleted earlier, and count toward Actions
storage allowances. Downloads require GitHub sign-in and repository read access.
Exports can contain sensitive issue data; anyone with access to the downloaded
ZIP can read it.

## Search behavior

- The repository and owner come from `GITHUB_REPOSITORY`, not user inputs.
  In a reusable workflow this identifies the calling repository.
- `is:issue` is always applied. Pull-request queries are rejected.
- If no `state:` qualifier is supplied, `state:open` is added. Use `state:closed`
  for closed issues. In particular, `is:closed` alone does not suppress this
  default; use `state:closed` instead.
- Use GitHub's search syntax for other filters. Multiple `label:` qualifiers
  require all labels; comma-separated alternatives match any listed label.
- Multi-word labels must be quoted: `label:"help wanted"`.
  Alternatives can also be quoted separately: `label:"help wanted","good first issue"`.
  A comma inside a quoted name is literal: `label:"priority, high"`.
- All positive and excluded label names are checked, case-insensitively, against
  the complete repository label list before searching. Unknown labels fail the
  run with an error listing each invalid option. Empty label values also fail.
- `repo:`, `org:`, and `user:` qualifiers are rejected to keep the export scoped
  to the repository running the workflow. Parentheses and OR expressions are
  grouped inside that repository scope, using GitHub's advanced search mode.
- GitHub search indexing can lag behind issue updates. Incomplete searches and
  queries matching more than GitHub's 1,000-result search limit fail rather than
  silently export a partial result. Narrow large queries, for example by date.
- A valid query with no matches still produces an empty index and JSON export.

GitHub reports invalid search syntax and access errors in the workflow logs.
No artifact is uploaded when validation or API requests fail.

## Authentication and permissions

The workflow requires the `ISSUE_READ_TOKEN` secret, passed explicitly by the
caller. Use a fine-grained personal access token with access to the calling
repository and **Issues: Read-only** permission (including label access).
The exporter reads `ISSUE_READ_TOKEN` from its environment and fails if it is
missing. The job declares `contents: read` and `issues: read`, but these job
permissions do not grant additional access to the supplied token.
Running it manually requires write access to the repository and Actions enabled.

The workflow executes the exporter and its tests from this exporter repository's
`main` branch, even when called from another repository or when the workflow
reference is pinned to a commit. Only call workflow code you trust with your
token. It uses GitHub-hosted Ubuntu runners and Node.js 22, with no npm dependencies.

## Tests

```sh
node --test .github/scripts/export-issues.test.mjs
```

Tests mock GitHub responses; they do not use credentials or make network requests.
