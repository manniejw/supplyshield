# SupplyShield

A scheduled job (`scan.mjs`) that uses the [Base44](https://base44.com) SDK to
scan recent supply-chain disruption news and create deduplicated `Alert`
records for the configured suppliers.

## How it works

`scan.mjs`:

1. Logs into Base44 with `BASE44_EMAIL` / `BASE44_PASSWORD`.
2. Loads suppliers, products, and active alerts.
3. Calls `base44.integrations.Core.InvokeLLM` with internet context to find
   confirmed disruption events from the last 30 days.
4. Filters/dedupes results and creates new `Alert` entities.

It runs every 6 hours via `.github/workflows/main.yml` (or on
`workflow_dispatch`).

## Configuring secrets

**Secrets are never committed to this repository.** They are stored as GitHub
Actions secrets and injected at runtime.

Add them under **Settings → Secrets and variables → Actions → New repository
secret**, or with the GitHub CLI:

```bash
gh secret set BASE44_EMAIL
gh secret set BASE44_PASSWORD
```

The workflow references them as `${{ secrets.BASE44_EMAIL }}` and
`${{ secrets.BASE44_PASSWORD }}`.

### Anthropic API key

The current code does **not** call the Anthropic API directly — the LLM step
goes through Base44's `InvokeLLM`, so no `ANTHROPIC_API_KEY` is required to run
`scan.mjs` today.

If you later switch to a direct Anthropic API call, configure the key the same
secure way — **never hardcode it in source or commit it**:

```bash
gh secret set ANTHROPIC_API_KEY
```

and add it to the workflow `env:` block as
`ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}`.

> If an API key has ever been pasted into a chat, an issue, a commit, or any
> other shared channel, treat it as compromised and rotate it at
> <https://console.anthropic.com/settings/keys>.

## Local development

```bash
cp .env.example .env   # fill in values; .env is gitignored
npm install
node scan.mjs
```
