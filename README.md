# Cilective/Select-Tests

Run only the tests needed for your change.

```yaml
- uses: cilective/select-tests@v1
  with:
    endpoint-url: https://api.cilective.com/select-tests
    cilective-api-key: ${{ secrets.CILECTIVE_API_KEY }}
```
> Automatically selects tests to run for your PR.

---

## 🚀 Usage

```yaml
- uses: cilective/select-tests@v1
  with:
    endpoint-url: https://api.cilective.com/select-tests
    cilective-api-key: ${{ secrets.CILECTIVE_API_KEY }}
    # github-token: ${{ secrets.GITHUB_TOKEN }} # Optional
```

---

## 🔑 Inputs

| Name                | Required | Description                                                                    |
|---------------------|----------|--------------------------------------------------------------------------------|
| `endpoint-url`      | Yes      | The URL of your Cilective select-tests endpoint.                               |
| `cilective-api-key` | Yes      | API key for authorizing the request. Use a GitHub secret.                      |
| `github-token`      | No       | GitHub token for API access. Defaults to the Actions-provided token if omitted. |

---

## 🎯 Outputs

| Name            | Description                                   |
|-----------------|-----------------------------------------------|
| `selected-tests`| A JSON array of test names to run for the PR. |

---

## 🛠️ How it Works

This action:
1. Collects all source files and changed files in the PR.
2. Streams them directly to your Cilective endpoint as a compressed tar.gz archive.
3. Receives a list of relevant test names based on your project’s dependency graph.

The stream is low-memory, fast, and supports large repositories.

---

## 🧩 Example: Use in a Workflow

```yaml
jobs:
  select-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: cilective/select-tests@v1
        id: select
        with:
          endpoint-url: https://api.cilective.com/select-tests
          cilective-api-key: ${{ secrets.CILECTIVE_API_KEY }}

      - name: Run selected tests
        run: |
          echo '${{ steps.select.outputs.selected-tests }}' > tests.json
          # For .NET, for example (joins test names with |):
          dotnet test --filter "FullyQualifiedName~$(jq -r '.[]' tests.json | paste -sd '|' -)"
```
> This example uses `jq` to join test names with `|` for use in `dotnet test`.  
> Adjust as needed for your test runner.

---

## 📝 Requirements

- The workflow must be triggered by a pull request event (`pull_request` or `pull_request_target`).

- Use `actions/checkout@v4` with `fetch-depth: 0` to ensure full commit history is available:
  ```yaml
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0
  ```

> Without this, the action will fail to compute diffs.

---

## 🔒 Security

- Pass `cilective-api-key` as a repository secret.
- The action does **not** log sensitive data.

Ongoing work toward SOC 2 compliance and audit-friendly practices.

---

## 📄 License

[Apache 2.0](./LICENSE)