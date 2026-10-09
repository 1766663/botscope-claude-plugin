---
name: site-bot-review
description: Use when the user asks which bots, crawlers or AI agents visited their own website, how much of their traffic is bots, what policy is set for an agent on their site, asks to set a specific agent to allow, observe, challenge or block on their site, or asks to see their site's robots.txt as Botscope generates it.
---

# Reviewing bots on the user's own Botscope site

These tools read the user's own Botscope account. The first call asks them to sign in;
that is expected. Everything they return is an aggregate — there are no IP addresses or
visitor records to look for.

## 1. Find the site

Call `list_sites`. If the user has more than one site and has not named a domain, ask
which one they mean. Do not pick the first site on your own. If they named a domain, match
it against `domain`.

## 2. Traffic

Call `traffic_summary` for that site with `days` = 7, unless the user asked for another
period. The tool accepts at most 28 days; if they asked for longer, say that 28 is the most
available and use 28.

Report the bot versus human split, the main categories, the top agents with their hit
counts, and `delta_pct` where it is notable. For per-agent policy alongside the counts,
use `list_agent_hits`.

If the result is empty or all zeros, say the site has no recorded agent traffic in that
period. If the tool returns `unavailable`, say that summaries are unavailable right now —
do not present that as zero traffic.

## 3. Policy for one agent

For a specific agent, call `get_agent_policy` with the site and the catalogue slug (use
`lookup_crawler` or `search_crawlers` from the crawler catalogue if you only have a name).

- State `effective_action`, the catalogue `default_action` and `site_mode`.
- If `pinned` is true, say the policy is fixed by the Botscope catalogue and cannot be
  changed for this site.
- If `site_mode` is `observe`, say the site is in observe mode: the policy is recorded but
  requests are not refused because of it.

## 4. Changing a policy

`set_agent_policy` changes one agent on one site. Call it only when all three are true:

- the user has, in this conversation, named the **site** (a domain or site you already
  showed them from `list_sites`),
- named the **agent** (a slug or an unambiguous name you resolved to a slug),
- and named **one action**: allow, observe, challenge or block.

"Clean it up", "block the bad ones", "block all bots" or "do what you recommend" do not
name an agent and an action. Do not turn them into writes. Show the agents and current
policies instead and ask which agent should get which action. One call changes one agent;
if the user names several, confirm the list back to them and make one call per agent.

Before calling, if you have not already, call `get_agent_policy` for that agent. If
`pinned` is true, say the catalogue fixes the policy and do not call `set_agent_policy`.

After the call, repeat from the result: `effective_action`, `site_mode` and `enforced`.
If `enforced` is false, say plainly that the site is in observe mode, so the new policy is
saved but requests are not refused because of it. Do not suggest switching the site to
enforce mode, and do not describe how to.

If the result is an error:

- `pinned`: the catalogue fixes this agent's policy; repeat the current action from the message.
- `forbidden`: the user's role on this site cannot change policies; someone with a
  owner, admin or analyst role in their organization can (a read-only role cannot).
- `insufficient_scope`: they connected without permission to change policies; they can
  reconnect the Botscope connector and allow it.

## 5. robots.txt

Call `get_site_robots_txt` without `include_body` to describe it (counts per action,
mode). Set `include_body` to true only when the user asks to see the file itself. If
`truncated` is true, say the file is too large to show here.

## 6. Errors

- `insufficient_scope`: the user did not grant that permission; ask them to reconnect the
  Botscope connector and allow it.
- `forbidden`: the site is not one they can access; call `list_sites` again.
- `login_required`: ask them to sign in to Botscope when Claude prompts.
