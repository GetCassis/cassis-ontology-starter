# Start a Cassis ontology on your own data

Cassis does context maintenance for analytics agents. Your business definitions live in a Git
repository you own, changes run against evals, and merging an approved pull request publishes the
context your agents read over MCP.

Use this template to start that repository for **your own schema**. If you want to see the loop first
on sample data, without touching yours, start at
[cassis-demo-stallora](https://github.com/GetCassis/cassis-demo-stallora) instead — fifteen minutes,
nothing of yours involved.

This repository is deliberately close to empty. It ships the wiring — the CI gates, the MCP config,
the modeling guide your agent reads — and none of the content, because the content is yours.

```
cassis/
  AGENTS.md            the modeling guide; your agent must read it first
  domains/README.md    the root domain. Replace this file first
  tables/              one YAML per table
  metrics/             one YAML per governed metric
.github/workflows/     validation + eval suite on every PR, publish on merge
.mcp.json              MCP config, so your agent can ask questions
.env.example           the two environment variables everything reads
```

## 1. Get a project and a key

Connecting your own warehouse is not self-serve yet. [Talk to us](https://getcassis.com/contact/)
and you get a project id back — one domain's schema is enough to begin, either a read-only
connection string or a DDL dump, and Cassis never needs the data itself.

Then in Cassis under **Settings → API keys**, create a key. One `sk-k6-…` key serves both the CLI and
the MCP server.

```bash
cp .env.example .env      # put your key in CASSIS_API_KEY, and your project id
set -a; . ./.env; set +a
pip install -U cassis-cli
```

For CI, set the same two values on the repository: `CASSIS_API_KEY` as an Actions **secret**,
`CASSIS_PROJECT_ID` as an Actions **variable**.

## 2. Bind the checkout and see what Cassis already knows

```bash
cassis ontology pull      # writes cassis/project.yml and the current tree
cassis schema pull        # gitignored snapshot of your source schema, for grepping
```

On a new project `pull` returns almost nothing — that is the expected starting point. `schema pull`
is the important one: it writes every table and column Cassis can see to `cassis/.schema.json`, which
is what your agent should read instead of guessing.

## 3. Have an agent write the first draft

Point a coding agent at this repository. With Claude Code, install the authoring skill and let it
drive:

```bash
/plugin marketplace add GetCassis/skills
/plugin install cassis-ontology-authoring@cassis
```

Any other agent works too — tell it to read `cassis/AGENTS.md` before it edits anything.

The order matters more than the tooling. Settle the **domain structure** first and get it reviewed:
structure is cheap to review and expensive to redo. Then fill bottom-up — table and column facts,
then metrics and joins, then each domain's README body. Feed it one source and one domain at a time;
a pass covering ten schemas at once produces mush.

Whatever context you already have is the raw material: dbt YAML and docs, the queries people run
most, dashboard SQL, runbooks, wiki pages, a glossary. That existing material is the whole difference
between a schema dump and an ontology.

Do not start from a blank tree. The [context bootstrap
kit](https://github.com/GetCassis/ontology-bootstrap) drafts the first one from that material —
warehouse schema, dbt models, dashboards, query history, docs — keeps the evidence behind every
claim, and stops at four checkpoints where you decide. Run it, review what it wrote, then continue
here.

**Flag, don't guess.** If a column's meaning is not provable from the schema or a document in front
of you, write a factual description and record the open question for a human. An invented definition
poisons answers silently, and it is the most expensive mistake available in this tree. Unresolved
questions belong in the pull request, not hidden in a description.

## 4. Prove it before it ships

```bash
cassis ontology fmt      # canonical form; refreshes AGENTS.md and domain navigation
cassis verify            # fmt --check, then check, then the eval suite
```

`verify` is the gate. Run `fmt` first — `check` fails on its own if a new metric is not yet linked
into its domain README. To see the effect of a change on a real question before publishing anything:

```bash
cassis ontology test -q "a question this change should now answer"
```

When someone who knows the data confirms an answer, pin it so the next change cannot quietly undo it:

```bash
cassis eval add-case -q "the confirmed question" --gold-sql "the correct SQL"
cassis eval run
```

On a warehouse-connected project the suite compares results, so different SQL reaching the same
numbers passes. On a schema-only project it compares the SQL itself with a judge.

## 5. Make merging the way it publishes

Open a pull request. The bundled workflow runs the same gates, a human reads the diff, and merging
publishes a new version.

Two ways to wire the publish step, and you want exactly one of them:

- **Cassis GitHub App** — in Cassis under Settings → GitHub, connect the app on this repository, then
  in Project configuration select your project, enter the repository as `owner/name` with path
  `cassis`, and save. The webhook imports on merge. **Delete `.github/workflows/publish.yml`**, or
  you publish twice.
- **Any other provider, or no app** — keep the `publish` job. It runs `cassis ontology upload` on
  merge to `main`. This is the provider-agnostic route and works identically on GitLab or Bitbucket.

## 6. Let it find its own gaps

Once people ask questions, Cassis clusters those conversations into issues: a missing definition, an
ambiguous term, a join it had to guess. Your agent reads that queue over MCP (`list_issues`,
`get_issue`, `get_issue_evidence`), fixes the root cause in these files, proves it with
`cassis verify`, and opens a pull request. Merging is the one step it cannot take.

## Reference

- [docs.getcassis.com](https://docs.getcassis.com) — [start here](https://docs.getcassis.com/start/overview/),
  [ontology in Git](https://docs.getcassis.com/build/git-workflow/),
  [CLI](https://docs.getcassis.com/reference/cli/),
  [CI recipes](https://docs.getcassis.com/build/ci/)
- Two complete worked ontologies, minimal and fully authored:
  [cassis-ontology-examples](https://github.com/GetCassis/cassis-ontology-examples)
- The same loop on sample data:
  [cassis-demo-stallora](https://github.com/GetCassis/cassis-demo-stallora)
