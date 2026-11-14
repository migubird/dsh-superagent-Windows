# dsh-superagent (Windows fork)

> **Fork of [deepseek-ai/dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype). Research prototype for reference only, not an official DeepSeek product.** The upstream project says the same of itself: it is research code, not production-ready software. This fork only adds native Windows support and packaging changes; review it before use.

dsh-superagent is a proactive coding agent for one GitHub repository, packaged as a bundle for DeepSeek Harness (DSH). It listens to GitHub and, optionally, Feishu. When someone allowed asks it for work, it opens a work item, works on it in its own agent Session and directory, and answers in the original conversation, for example with a pull request. A web panel shows what needs a person.

## What this fork changes

- **Native Windows support.** The state directory accepts Windows absolute paths (`D:/dsh/state`), setup probes no longer shell out to Unix `env`, and the worker command shims are generated as `.cmd` on Windows.
- **A default state directory.** Without a `stateDir` in the profile configuration the plugin now uses `<DSH_HOME>/state/superagent` (`~/.dsh/state/superagent` by default), so a fresh install starts without hand-editing `cordis.patch.yml`.
- **Wider compatibility range.** `engines.dsh` accepts both the `0.2.1-alpha.2` line and `0.2.2-rc.*`.

Everything else, including the panel, Sources, Policy, Execution and Prompts, is unchanged from upstream. The upstream documentation under [`docs/`](docs/), starting with [000 Philosophy](docs/000-philosophy.md), describes the design; [SETUP.md](SETUP.md) is the upstream step-by-step setup guide and assumes a POSIX host.

## Install

Pick one, in the DSH profile that should run the agent:

```sh
# From a Release tarball
dsh plugin --profile <profile> add https://github.com/migubird/dsh-superagent-Windows/releases/download/v0.1.0/dsh-superagent-0.1.0.tgz

# From a local checkout (build it first: pnpm install && pnpm build)
dsh plugin --profile <profile> add "link:<absolute checkout path>"

# From npm, once published
dsh plugin --profile <profile> add dsh-superagent
```

The same installs are available in the DSH plugin manager (Web: **Plugins → Install**). After installing into the running Web profile, reload the page: the sidebar then shows a hammer icon labelled **Superagent**.

### From the plugin market

This repository publishes a Discover index for the bundle:

- Index URL: `https://cdn.jsdelivr.net/gh/migubird/dsh-superagent-Windows@main/market/index.json`
- Mirror: `https://raw.githubusercontent.com/migubird/dsh-superagent-Windows/main/market/index.json`

Turn on the experimental market bundle (`@deepseek-ai/dsh-experimental-plugin-market-bundle`), open **Plugins → Discover**, add one of those URLs under **Index sources**, and install the entry. The index pins an exact git commit, so no npm publication is needed; a later index version installed from the same repository shows as an update.

Installing into your everyday profile also runs the plugin in that profile's process (see Security below). Upstream recommends a dedicated `superagent` profile, and this fork works there too.

## Configure

The panel keeps every setting except the machine configuration:

1. **Sources** — the repository (`owner/name`) and the agent's GitHub CLI. Requests normally arrive through that account's notifications. Feishu is optional.
2. **Policy** — which GitHub logins may ask for work. Until a list has someone, no request can become a Ticket.
3. **Execution** — the directory root and the `prepare` command that creates a checkout per Ticket.
4. **Prompts** — the team sections, including commit identity and auto-merge.

Prerequisites for real work: GitHub CLI 2.48+, a dedicated agent GitHub account signed in to a wrapper CLI, write access to the repository, and a model provider configured in the DSH profile. Until the repository and CLI are set, the overview reports `The GitHub Source names no repository.` and nothing is polled or executed.

To set machine limits, or pin another state directory, add a profile patch entry:

```yaml
- id: dsh-superagent
  config:
    stateDir: D:/dsh/state/superagent
```

### Windows notes

- `scripts/dsh-superagent` is POSIX-only (`tmux`, `ps`, `fcntl`). On Windows, start the profile yourself (`dsh --profile <profile>`) or run the plugin inside the DSH desktop/Web profile; there is no matching start/stop wrapper yet.
- Worker branches are pushed by the agent's GitHub CLI. Git for Windows runs the credential helper through its own `sh`, which this fork has not exercised end to end; watch the first pull request.
- The upstream cleanup and hourly-check instructions use the POSIX control script; adapt or skip them.

## Remove

```sh
dsh plugin --profile <profile> remove dsh-superagent
```

or remove the bundle from **Plugins** in the panel. The state directory (database and per-Ticket directories) stays on disk; delete it to discard the agent's data.

## Security

Both agent presets run with `danger-full-access` as your OS user; a worker can execute programs and read your home directory. The plugin also changes the host process's `PATH` and git configuration while it runs. Install it only from a source you trust, and prefer a dedicated OS account or profile when the agent acts for a team.

## Development

```sh
pnpm install
pnpm check   # typecheck, tests, build
```

`lib/` is committed on purpose: the plugin market installs this package from a pinned git commit, and a git install runs no build step. Rebuild and commit `lib/` for every release.

## License and attribution

MIT, as upstream. The original code is by the DeepSeek Harness Team; this fork's Windows changes are maintained at [migubird/dsh-superagent-Windows](https://github.com/migubird/dsh-superagent-Windows).
