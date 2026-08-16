# Onboarding non-technical builders onto Claude + GitHub

This repo tracks the setup used to give someone with no coding-tool experience
the same core workflow used here — building real files with Claude, backed by
real version control — without exposing them to VS Code or raw GitHub.

The working example is [`apratsunrthd/kaylie-project-template`](https://github.com/apratsunrthd/kaylie-project-template),
built for Kaylie. This doc is the general recipe, so it can be repeated for
the next person.

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

## To replicate for someone new

1. Copy the structure of `kaylie-project-template`: `README.md`,
   `.gitignore`, `.claude/settings.json` (SessionStart hook).
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
