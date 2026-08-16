# Onboarding citizen developers onto Claude + GitHub

This repo tracks the setup used to give a **citizen developer** — someone
building real software without a formal engineering background — the same
core workflow used here: building real files with Claude, backed by real
version control, without exposing them to VS Code or raw GitHub.

The working example is [`apratsunrthd/citizen-project-template`](https://github.com/apratsunrthd/citizen-project-template),
built for one specific collaborator. This doc is the general recipe, so it
can be repeated for the next person — nothing about the pattern below is
specific to that first example.

**Note:** this repo (`n00b-claude-coder`) is not itself a template — it's the
write-up of the pattern. The thing someone actually clicks "Use this
template" on is a separate repo you build by following the recipe below
(e.g. `citizen-project-template`).

## The pattern

1. **Base tool: claude.ai/code, not VS Code.** File-native and versioned like
   Claude Code, but browser-based and far less intimidating for a first-timer.
2. **Distribution: a GitHub "template repository."** The person clicks
   **"Use this template"** on GitHub and gets their own copy — no `git clone`,
   no terminal.
3. **gstack, auto-installed per session.** claude.ai/code cloud sessions are
   ephemeral (no persistent `~/.claude/skills/` between sessions), and
   gstack's own `--local` install mode is deprecated/non-portable — it can't
   just be committed into the repo. The fix: a `SessionStart` hook in the
   template's `.claude/settings.json`, which *is* committed and *does* travel
   with the GitHub template. It clones gstack, checks out **a pinned commit
   SHA** (not a moving branch), and runs `./setup --quiet`, guarded so it only
   runs when `~/.claude/skills/gstack/bin` is missing.
   - Known cost: because sessions are ephemeral, this reinstall (including
     ~250MB of Playwright browser binaries) reruns every session, not just
     the first — expect a few minutes of setup time per session.
   - Known tradeoff, accepted deliberately: the hook auto-executes shell code
     and fetches from the network on every session start, without the user
     seeing it. Pinning to a reviewed commit (rather than tracking a branch)
     limits — but doesn't eliminate — that exposure. Bumping gstack versions
     later means manually reviewing and editing the pinned SHA, not an
     automatic update.
4. **A plain-language README in the template**, explaining "click this
   button, go here, describe what you want" — no git/GitHub/CLI vocabulary.
5. **Branch protection on `main`** in whatever repo the person ends up in
   (PRs required, no direct/force pushes) if you want history to stay clean —
   optional, since the goal here is removing friction, not enforcing process
   on someone who isn't shipping to a team.
6. **A heads-up about GitHub App repo access.** claude.ai/code reaches GitHub
   through a "Claude" GitHub App installation. If that's installed with
   access limited to specific repos (rather than "All repositories"), any
   brand-new repo — including ones made from this template — won't show up
   when the person searches for it in claude.ai/code ("No repos match").
   This is real friction a citizen developer will hit and won't know how to
   read. **There's no way to fix this on their behalf programmatically** —
   confirmed by trying: the GitHub API for managing a GitHub App
   installation's repo access
   (`PUT /user/installations/{id}/repositories/{id}`) requires a token
   issued through the App's own OAuth flow, not a personal access token like
   `gh` uses. The fix has to happen in the person's own GitHub settings, so
   put it in front of them proactively (as a one-time "before your first
   project" step, not just buried in troubleshooting): go to
   [github.com/settings/installations](https://github.com/settings/installations),
   find **Claude**, click **Configure**, switch to **All repositories**.
   Done once, it covers every future project.
7. **A `CLAUDE.md` in the template that governs how Claude talks to the
   person, not just what they read.** A live test surfaced two gaps
   documentation alone wouldn't have caught: Claude built a webpage but
   never told the tester how to view it (claude.ai/code has no preview panel
   or port-forwarding — Claude has to proactively publish a Claude Artifact
   and hand back the link), and it asked "want a PR?" with zero explanation,
   meaningless to someone with no git background. `CLAUDE.md` instructions
   apply to every session against the repo, so they're a much stronger lever
   than a README the person might skim past — use it to require
   plain-language explanations before any git/GitHub action, and to require
   showing (not just describing) anything visual that gets built.

## To replicate for someone new

1. Copy the structure of an existing template repo (e.g.
   `citizen-project-template`): `README.md`, `.gitignore`,
   `.claude/settings.json` (SessionStart hook), `CLAUDE.md` (plain-language
   behavior instructions).
2. Get the current commit SHA of the gstack fork you want pinned:
   `git -C ~/.claude/skills/gstack log -1 --format=%H`, and put it in the
   hook command.
3. `gh repo create <name> --public --source=. --push`, then
   `gh api repos/<owner>/<name> -X PATCH -f is_template=true`.
4. Test the hook end-to-end before handing it off. See
   [TESTING.md](TESTING.md) for the full walkthrough — at minimum, run the
   hook's exact command in an isolated `$HOME` and confirm the skills install
   cleanly, then do a real "Use this template" run on claude.ai/code yourself.
5. Send them the repo link and the two-sentence version: click "Use this
   template," then go to claude.ai/code and describe what you want to build.
