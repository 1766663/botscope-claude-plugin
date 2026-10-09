# Botscope for Claude

Botscope answers questions about web crawlers and AI agents from a maintained catalogue
instead of from memory. Ask who a bot is, whether a User-Agent string is what it claims
to be, or for a robots.txt fragment that keeps search engines in and training crawlers out.

## What it does

- **Look up a crawler** by name or slug: operator, category, how it can be verified
  (reverse DNS, published IP ranges, Web Bot Auth signatures) and its default policy,
  with a link to the full catalogue record.
- **Identify a User-Agent**: which catalogue entry the string claims to be and, if you
  give an IP, whether that IP falls inside the entry's published ranges. A matching
  User-Agent alone is reported as an unverified claim, because anyone can send one.
- **Review your own site** after signing in to Botscope: which bots and AI agents reached
  it over the last 1 to 28 days, how much of the traffic is bots, the policy each agent
  gets, and how your generated robots.txt is composed.
- **Set one agent's policy** on your site — allow, observe, challenge or block — when you
  name the site, the agent and the action. One agent per change, recorded in the site's
  audit log. It never changes the site's mode, rules or bans.
- **Draft robots.txt** for three intents: block AI training, block all AI agents, or
  follow the catalogue defaults. robots.txt is a request to crawlers, not enforcement.

## Install

In Claude Code:

```
/plugin marketplace add 1766663/botscope-claude-plugin
/plugin install botscope@botscope
```

Or add only the connector: `claude mcp add --transport http botscope https://mcp.botscope.io/mcp`.
In claude.ai and Claude Desktop, add `https://mcp.botscope.io/mcp` as a custom connector.

## How to use it

Install the plugin and ask in plain language, for example "Who is GPTBot and what happens
if I block it in robots.txt?", "Is this User-Agent really Googlebot?" followed by the
string, "Build a robots.txt that keeps search but blocks training crawlers.", or, with a
Botscope account, "Which AI bots visited example.com this week?" or "Set GPTBot to block
on example.com."

## What data is sent

Tool calls go to `mcp.botscope.io`, which forwards them to the Botscope catalogue API.
Catalogue questions need no Botscope account and carry only the tool parameters: a bot
name or slug, a search term, a User-Agent string and optional IP, or a robots.txt intent.
Nothing is sent about your conversation beyond those parameters. The catalogue API does
not write what you asked to its application log; the web server's access log records
request paths, which include search terms, under its normal retention. Questions about your
own site ask you to sign in to Botscope first, on botscope.io, where you choose which
permissions to grant (list sites, read traffic summaries, read policies, change agent
policies — the last one is optional). Those calls send only the site identifier and the
tool parameters, and return aggregates — never visitor IP addresses or query strings. The
only change the connector can make is one agent's policy on one site, and only with that
permission and a role on the site that allows it.

## License

The plugin files are MIT licensed. The Botscope service itself is not part of this
repository.
