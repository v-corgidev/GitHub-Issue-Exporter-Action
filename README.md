# GitHub-Issue-Exporter-Action

Export GitHub issues and their comments to a downloadable ZIP containing
offline HTML pages and JSON. Call the reusable workflow from your repository;
you do not need to copy the exporter scripts.

## Use in your repository

1. Create a fine-grained personal access token with access to your repository
   and **Issues: Read-only** permission. Add it as an Actions repository secret
   named `ISSUE_READ_TOKEN` under **Settings > Secrets and variables > Actions**.
   Complete any organization approval or authorization required for the token.
2. Add the following workflow as `.github/workflows/export-issues.yml` in your
   repository and commit it to the default branch:

   ```yaml
   name: Export issues

   on:
     workflow_dispatch:
       inputs:
         query:
           description: 'GitHub issue search query (for example, label:bug)'
           required: true
           type: string

   permissions:
     contents: read
     issues: read

   jobs:
     export:
       uses: CorgiDev/GitHub-Issue-Exporter-Action/.github/workflows/export-issues.yml@main
       with:
         query: ${{ inputs.query }}
       secrets:
         ISSUE_READ_TOKEN: ${{ secrets.ISSUE_READ_TOKEN }}
   ```

3. Open **Actions > Export issues > Run workflow** in your repository and enter
   a query such as `label:bug` or `state:closed label:"help wanted"`.
4. Open the successful run, download the **issue-export-...** artifact, extract
   the ZIP, and open `index.html`.

The reusable workflow searches **the calling repository**, not this exporter
repository. Do not include `repo:`, `org:`, or `user:` in the query. Only issues
are exported; the default is open issues unless a `state:` qualifier is supplied.
Queries matching more than 1,000 issues fail rather than produce partial exports.

The required `query` input is a string, and `ISSUE_READ_TOKEN` is a required
secret. The exporter reads that secret through the `ISSUE_READ_TOKEN` environment
variable; it does not use the automatic workflow token to read issues.
The workflow uses GitHub-hosted Ubuntu runners and Node.js 22. It checks out
exporter scripts from this repository's `main` branch, not the caller's code.
The `@main` reference receives updates automatically; pinning the workflow to a
commit does not pin the separately checked-out scripts, which still use `main`.

Artifacts belong to the caller's workflow run and expire after **7 days**.
Downloads require GitHub sign-in and repository read access. Exports may contain
sensitive issue data; anyone with the ZIP can read it. Issue text and comments
work offline, but linked media may require internet access and GitHub authorization.

For detailed search rules, running this workflow directly, and local tests, see
the [workflow documentation](.github/README.md).