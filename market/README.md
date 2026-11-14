# Plugin market index

`index.json` is a format-1 DSH plugin market index (`@deepseek-ai/dsh-plugin-market-index`). It lists the bundles in this repository so DSH's Discover tab can search and install them.

Read it from one of:

- `https://cdn.jsdelivr.net/gh/migubird/dsh-superagent-Windows@main/market/index.json` (CDN, reachable where GitHub is blocked; branch content is cached for a few hours)
- `https://raw.githubusercontent.com/migubird/dsh-superagent-Windows/main/market/index.json` (always fresh)

In DSH, turn on `@deepseek-ai/dsh-experimental-plugin-market-bundle`, open **Plugins → Discover**, and add the URL under **Index sources**.

## Adding a version

1. Bump `version` in `package.json`, run `pnpm run build`, and commit both `package.json` and the rebuilt `lib/`.
2. Push, and tag the commit (`vX.Y.Z`) so the Release workflow also publishes a tarball.
3. Add a `versions` entry whose `source` pins that commit:

   ```json
   { "version": "X.Y.Z", "publishTimestamp": 1794636825,
     "source": { "type": "git", "url": "https://github.com/migubird/dsh-superagent-Windows", "commit": "<40-hex commit>" },
     "engines": { "dsh": "^0.2.1-alpha.2 || ^0.2.2-rc.1" } }
   ```

   `engines.dsh` must be a SemVer range; keep it to the DSH versions you tested.
4. Validate before committing:

   ```js
   import { parseMarketIndex } from '@deepseek-ai/dsh-plugin-market-index'
   parseMarketIndex(JSON.parse(readFileSync('market/index.json', 'utf8')), 'https://example.com/index.json')
   ```

Never remove a published version: installed copies compare against it for updates.
