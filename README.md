# Buny agents

Claude Code automation for the Buny reading ecosystem: the `source-dev` agent plus its
`write-source`, `test-source`, and `doctor-source` skills, for scaffolding, verifying, and fixing
Buny WASM sources.

## Install

```
claude plugin marketplace add Buny-Community/agents
claude plugin install source-dev@buny-agents
```

Run this from a `buny-sources` checkout.

## Usage

```
/write-source <url>       scaffold a new source from a website URL
/test-source <id>         verify an existing source builds, lints, and packages
/doctor-source <id>       diagnose selector drift against the live site and fix it
```

See `AGENTS.md` for the fuller repo context (stack, layout, hard rules) behind this tooling.
