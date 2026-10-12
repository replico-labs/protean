# protean-site — Landing Page & Docs

The public-facing marketing and documentation site for Protean —
the governance layer for social messengers.

## What this is

A single, self-contained static HTML page (`index.html`) covering two
things:

- **Home** — what Protean is, how it works, the ten governance models
  plus Opportunity Market, the five use cases, where it's deployed, and
  a "Get started" section linking to the live bots.
- **Docs** — architecture, a full writeup of every governance model,
  Opportunity Market, the Guard Wrapper, the complete command
  reference, and real, verified factory contract addresses per chain.

No React, no build step, no bundler — plain HTML/CSS/JS, with Google
Fonts (Inter + JetBrains Mono) loaded via `<link>`. This was a
deliberate choice: this site has no wallet-connect flow, no backend
data of its own, and no interactivity beyond switching between two
views and smooth-scrolling to an anchor — a full framework would be
more machinery than the actual job needs.

## Brand

Colors, type, and the logo (`public/logo-mark.svg`,
`public/logo-wordmark.svg`, `public/favicon.svg`) all match the
project's real, established identity — deep violet background
(`#12072a`), lilac accent (`#b39cf5`), a classical column/temple mark.
If the brand changes, the color tokens are all defined once, at the
top of `index.html`'s `<style>` block, under `:root`.

## Running it locally

No install or build step needed:

```bash
python -m http.server 8000
```

or open `index.html` directly in a browser. Either works, since there's
no server-side logic at all.

## Deploying

Built for Vercel as a plain static site — no framework, no build
command. `vercel.json` explicitly sets `framework: null` and
`buildCommand: null` so Vercel's zero-config detection doesn't get
confused by the `public/` folder (a common convention for build output
in other frameworks) into looking for a compiled site that doesn't
exist here.

Deploy via the Vercel dashboard (import the repo) or the CLI:

```bash
npx vercel --prod
```

Any other static host (Netlify, GitHub Pages) would also work directly
off `index.html` with no changes.

## Updating content

Everything — copy, the command reference, chain addresses — lives
directly in `index.html`. There's no CMS or data file to edit
separately. Search for the relevant section by its `id` (`#models`,
`#applications`, `#bots`, `#chains`, `#doc-commands`, etc.) and edit
the HTML in place.

### Adding a chain's factory addresses

Each verified address in the docs' "Chains & contracts" section links
directly to that chain's own block explorer, using the pattern
`<explorer>/address/<address>`:

| Network | Chain ID | Explorer |
|---|---|---|
| Monad testnet | 10143 | `https://testnet.monadvision.com` |
| Base Sepolia | 84532 | `https://sepolia.basescan.org` |
| HyperEVM testnet | 998 | `https://testnet.hyperevm-explorer.xyz` |
| Ethereum Sepolia | 11155111 | `https://sepolia.etherscan.io` |

When a new chain is added, confirm its real explorer URL first rather
than reusing another chain's pattern, since each explorer has its own
domain and URL shape.

### Bot links

All three platform cards in `#bots` (Telegram, Discord, Slack) are live
and link to their real install/invite URLs. To change a link, edit the
`href` on that platform's `<a class="bot-cta live">` directly. To add a
new platform later, copy an existing card and, until its link exists,
use a muted placeholder instead: `<span class="bot-cta">Add to X —
coming soon</span>`, then swap it for a real `<a class="bot-cta live"
href="...">` once the link is ready.

Note that the Discord invite URL contains `&amp;` rather than `&` —
that's correct HTML for an ampersand inside an attribute, and browsers
decode it back to a normal `&` when the link is clicked.

## Proposal pages

`proposal.html` is the page for one DAO proposal, at `/p/<network>/<dao>/<id>` (the rewrite is in `vercel.json`). The bot links to it after every proposal and in `/proposal <id>`.

- **Data comes from the bot**, not from this site: the page fetches `GET <bot>/api/proposals/<network>/<dao>/<id>` for the proposer's details and live on-chain state (refreshed every 20 seconds), and the edit form `POST`s back there. The bot's URL is the `protean-api` meta tag at the top of `proposal.html` - change it there if the bot moves.
- **Details are submitted once**, only through the private link the bot sends the proposer (`?edit=<signed token>`, checked by the bot), and never change afterwards. They can only be submitted before anyone votes or backs the proposal, within 72 hours.
- **No secrets here.** Everything sensitive (the signing secret, Supabase) lives in the bot. Everything a proposer writes is rendered as text, never HTML.
- The bot needs `PROPOSAL_SITE_URL` set to this site's URL (it builds the links from it, and only accepts requests from it).

## Related repositories

- **Governance contracts** (Solidity, Foundry)
- **Bot** (Node.js — Telegram, Discord, Slack interfaces)
- **protean-connect** — a separate, paused project for non-custodial
  wallet linking via Privy; not currently used by the bot, unrelated
  to this site