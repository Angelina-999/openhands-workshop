# CLAUDE.md

## Execution modes

Two ways this repo is worked on. Before you touch git, detect which one applies by running:

```bash
test -d /openhands && echo "OpenHands dev" || echo "Local dev"
```

- **OpenHands dev** — the session runs in an OpenHands sandbox. Deliver work as a merge request.
- **Local dev** — anything else: a human runs the agent on their own machine.

This check decides it. Never ask which mode applies, and never infer it from the session feeling
interactive, from the branch name, or from the task being small.

Everything in this file applies to both modes except "Committing and pushing", which splits them explicitly.

## Codebase map (TypeScript)
- Main app code lives in `src/` (`components/`, `pages/`, `data/`, `utils/`, `routes.ts`).
- Many files in `src/pages/` are thin re-export wrappers; shared page logic is usually in `src/components/`.
- Shared JSON shape/type declarations are in `src/data/json-structure.ts` and `declarations.d.ts`.

## Architecture notes
- App state is centralized in `src/data/redux-state.ts`; route-driven indicator switching is in `src/data/indicator-class.ts`.
- JSON lookup mappings are in `src/data/variable-list.js` (JavaScript). Keep TypeScript changes compatible with this JS module unless a task explicitly includes migrating it.

## Context-loading rules
- Do not load JSON into context by default; load JSON only when it is required for the current task.

## Build verification
- After any non-trivial TypeScript modification, run `pnpm exec tsc`.
- After any page modification (files under `src/pages/`), run `pnpm exec tsc && gatsby build`.

## UI verification (Playwright)
The `develop` script serves on port 8100, so screenshot against `http://localhost:8100/`:

```bash
corepack pnpm run develop
```

If 8100 already answers, reuse it instead of starting a second server.

Tooling: Playwright is a devDependency; its browser binaries are not, so run `pnpm exec playwright install chromium` once if `~/.cache/ms-playwright` is empty; run its CLI directly, e.g. `pnpm exec playwright screenshot <url> <out.png>`. Only write an ad-hoc Node script when a task needs chained interactions (clicks, downloads) beyond what the CLI covers.

Navigation:
- Reach the relevant page state by direct route and URL-parameter navigation. Prefer encoding reproducible state in the URL over click-navigation whenever the app supports it — it costs fewer tokens and makes the steps easy to repeat.
- Fall back to UI interactions only for state that cannot be set directly via the URL.

Procedure for verifying a change:
1. Before the change, screenshot the page in the state you are about to modify (the baseline).
2. Make the change.
3. After the change, screenshot the same state.
4. Verify whether the goal has been reached.
5. If yes, compare the before/after screenshots to detect unwanted UI side effects elsewhere on the page.

If a screenshot shows the wrong state, iterate with targeted navigation or interaction only while the screenshots are still adding evidence. Stop once the goal is confirmed, or once the loop is becoming fruitless and likely to waste screenshots or tokens.

Never use a Playwright MCP server (or similar browser-automation MCP) for this — always the CLI/script approach above.

Retention of screenshots:
Place the screenshots into the folder <before-after/name-of-issue/before-iso-date.png> and <before-after/name-of-issue/after-iso-date.png>
Those screenshots should be committed into git after manual approval!

## Change focus
- Keep edits minimal and targeted.
- Prefer changing canonical shared components over duplicative page wrappers when behavior should apply across routes.
- Do not change generated output under `public/` manually, because those JSONs are served from other systems and will be overwritten frequently.

## Committing and pushing

### Local dev
- Do not commit until a human explicitly asks you to.
- When asked to commit, automatically push as well after the commit succeeds.
- Pushes go directly to the main branch (`develop`) — do not open merge requests or work on feature branches.

### OpenHands dev
Never push to `develop` and never wait to be asked, even if someone is watching the session. Deliver finished work as a merge request carrying a deployed preview link, so a human can review the diff and click through to the running result.

1. **Branch** off `develop`, never off whatever branch happens to be checked out, and fetch first so the
   base is current: `git fetch origin && git switch -c openhands/<short-task-slug> origin/develop`.
2. **Verify first.** Run the checks the change calls for (`corepack pnpm exec tsc`, plus `gatsby build` for anything under `src/pages/`, plus the Playwright before/after screenshots for UI changes). If verification fails, stop and report — do not open an MR for unverified work.
3. **Commit**, including the `before-after/` screenshots for UI changes.
4. **Deploy a preview.** Skip if nothing in the MR lands in the build (docs-only) — note that in the
   description instead of linking a preview. Otherwise build, then push a Netlify *draft* deploy under a
   branch alias so the link stays stable across follow-up commits:
   ```bash
   corepack pnpm exec gatsby build
   ALIAS=$(git branch --show-current | tr '/_' '--' | cut -c1-37)
   NTOK=$(node -e 'console.log(/--auth="([^"]+)"/.exec(require("./package.json").scripts.deploy)[1])')
   SITE=$(node -e 'console.log(/--site="([^"]+)"/.exec(require("./package.json").scripts.deploy)[1])')
   corepack pnpm exec netlify deploy --dir=public --alias="$ALIAS" \
     --auth="$NTOK" --site="$SITE" --json > /tmp/deploy.json
   jq -r '.deploy_url' /tmp/deploy.json
   ```
   Never pass `--prod`. Promotion to https://gcci.earth/ is a manual step in Netlify.
5. **Open the MR** and put the preview link in its description. The GitLab PAT is already in the push remote:
   ```bash
   git push -u origin HEAD
   TOK=$(git remote get-url origin | sed -n 's#.*oauth2:\([^@]*\)@.*#\1#p')
   curl -s -X POST -H "PRIVATE-TOKEN: $TOK" \
     --data-urlencode "source_branch=$(git branch --show-current)" \
     --data-urlencode "target_branch=develop" \
     --data-urlencode "title=<type>: <short summary>" \
     --data-urlencode "description=$(cat /tmp/mr-body.md)" \
     --data-urlencode "remove_source_branch=true" \
     "https://wegcgitlab.uni-graz.at/api/v4/projects/arsclisys%2Fgcci%2Fgccivis/merge_requests" | jq -r '.web_url'
   ```
6. **Report** both the MR URL and the preview URL in your final message.

Notes:
- The Netlify site is not linked to GitLab, so there are no automatic per-MR previews. The draft deploy in step 4 *is* the preview, and it only reflects the commit you built it from.
- When a follow-up commit changes anything that lands in the build, re-run step 4; the alias URL updates in place, so the link already in the description keeps working. Docs-only follow-ups produce an identical site and do not need a re-deploy.
- Write the MR description according to "Documentation style" below.

## Dependency Hygiene and Security

No new packages!!
If a task looks difficult, do not go for a radical goal-achieval-quest that involves downloading untrusted npm packages or Python scripts.
Instead, make a hardstop and notify the user if a task cannot be accomplished with the existing packages and without any crazy workarounds.

## Documentation style

In MR-descriptions and other places, follow this style: Short, precise, and close to the prompt-author. The original writing style of the author should still be "felt" even after you translate a prompt into an MR-description.

Do not bloat up MR-descriptions by making a bunch of assumptions. An incomplete description is better than a bloated description with freely invented assumptions.

What should never be missing:
- the steps to verify the MR,
- before/after screenshots in the MR-description if needed,
- a link to the Netlify deployment.

## Reporting progress in GitLab-comments

Do NOT dump in a long, bloated AI-typical description of everything in detail what you did.
Instead, write only one or two sentence per action-point together with the respective images "before" and "after".
