# Working in this repository

This repository holds a Cassis ontology: the business context that grounds AI agents when they query
this organization's data. These files are the source of truth, and merging to `main` publishes a new
version, so nothing reaches production answers until a human approves the pull request.

**Read `cassis/AGENTS.md` before editing anything.** It is the modeling guide — where rules belong,
how to write descriptions, metric and join conventions. It is managed by `cassis-cli` and rewritten by
`cassis ontology fmt`, so never hand-edit it.

## Order of work

Structure before content. Review the domain tree, decide whether new material extends a domain or
deserves a new one, and get that shape agreed before writing content. Then fill bottom-up: table and
column facts, then metrics and joins, then each domain README body. One source and one domain at a
time.

`cassis schema pull` writes `cassis/.schema.json`, a gitignored snapshot of every table and column the
source has. Grep it. Do not guess at column names.

## Flag, don't guess

If a column's meaning is not provable from the schema or from a document you were given, write a
factual description and record the open question for a human. An invented definition poisons answers
silently. Unresolved questions go in the pull request body, never hidden in a description.

## Gates before a pull request

```bash
cassis ontology fmt      # run first: check fails if a new metric is not linked into its domain
cassis verify            # fmt --check, then check, then the eval suite
cassis ontology test -q "a question this change should now answer"
```

`CASSIS_API_KEY` is required for every CLI command — validation runs server-side.

## Maintenance

Cassis clusters real questions into a ranked queue of context problems. Read it over MCP with
`list_issues`, `get_issue` and `get_issue_evidence`, fix the root cause here, prove it with
`cassis verify`, and open a pull request. Merging is the only step to leave to a human.
