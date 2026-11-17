# live-agent

A customer support and operations agent for a fictional SaaS company. It answers
questions from product documentation, looks up and changes records (tickets,
orders, refunds), and pauses for human approval before risky actions.

## Architecture

| Layer | Choice |
| --- | --- |
| Web page | Next.js (TypeScript) on Vercel. Serves the chat UI only and holds no model API keys. |
| Agent service | Python, FastAPI, LangGraph and LangChain, in a container on Google Cloud Run. |
| Database | Supabase Postgres: business tables, pgvector for documents, LangGraph checkpoints, feedback. |
| Sign-in and access | Supabase Auth with row-level security. |
| Models | Anthropic as the main model. DeepSeek for cheap tasks, fallback and the eval judge. |
| Tracing | Langfuse. |
| Tests and evals | pytest. Eval scores are logged to Langfuse and gated in CI. |
| Red teaming | promptfoo. |
| CI | GitHub Actions. |

The browser loads the page from Vercel. The page calls the agent service with
the signed-in user's Supabase token. Only the agent service calls the models,
the database and the tracing service.

## Planned layout

The repo is scaffolded step by step. Update this section when the layout changes.

```
agent/        Python agent service (FastAPI, LangGraph, tools)
web/          Next.js chat page
supabase/     Database migrations and seed data
evals/        Eval datasets, judge rubrics, result history
docs/         Spec, threat model, runbook, post-mortems
docs/adr/     Architecture decision records
```

## Commands

None yet. Add the real install, run, test, lint and eval commands here as soon
as they exist, and keep them current.

## Conventions

- Python: type hints everywhere, ruff for lint and format, pyright for types.
- Tests live next to the code they cover and run with pytest. Tests never call
  a real model; use a fake model. Evals are the only place real models run.
- Every real design choice gets a short record in `docs/adr/` (context,
  decision, alternatives rejected, consequences).
- One change per branch and pull request. `main` stays deployable.
- No change merges unless tests pass and eval scores hold.
- Prompts are versioned files in the repo, not strings scattered through code.

## Security rules

- Never commit secrets. `.env*` files are ignored; `.env.example` lists names
  with no values.
- Model API keys live only in Google Secret Manager for the agent service.
  They never go in Vercel, in browser code or in this repo.
- The Supabase service-role key is never used in the web page.
- Tools take the user's identity from the verified token, never from model
  output. Authorization is enforced in the database with row-level security.
- All customers, orders, tickets and documents are synthetic. No real personal
  data and nothing from any employer.
- Actions with side effects (refunds, record changes) need human approval and
  write an audit row.
- Treat retrieved documents and tool outputs as untrusted input.
