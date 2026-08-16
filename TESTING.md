# Testing a template before handing it off

Before sending a template repo (e.g. `kaylie-project-template`) to the
citizen developer it's for, verify the whole flow works end to end using
your own account. Don't skip this — it's the only way to catch a broken
`SessionStart` hook or a bad pinned commit before they do.

## 1. Create a test copy

On the template repo's GitHub page, click **"Use this template" → "Create a
new repository"**. Name it something obviously temporary (e.g.
`<name>-test`); private is fine for a test run.

## 2. Open it in claude.ai/code

Go to [claude.ai/code](https://claude.ai/code), connect it to the new test
repo, and start a session.

## 3. Watch the first minute or two

The `SessionStart` hook is cloning the pinned skill fork and running its
installer in the background. If Claude seems slow to respond to your very
first message, that's expected — this is the "a few minutes of setup" the
template's own README warns the end user about, not a hang.

## 4. Confirm the skills actually installed

Once it's responsive, ask directly — e.g. "what gstack skills do you have
available?" — or invoke one (`/qa`). If the hook worked, it recognizes them.
If it doesn't, the hook silently failed (by design, so a network blip
doesn't block the person's whole session) — check the pinned commit SHA in
`.claude/settings.json` is still valid and the setup command still runs
cleanly (see the isolated-`$HOME` test described in the main README).

## 5. Build something trivial

Ask it to create a simple file — a to-do list, a one-page HTML page,
anything. Confirm the file actually appears, then check the repo on GitHub
afterward to confirm it got committed. If it's not obvious how to trigger a
commit in that UI, just ask Claude directly to commit and push — same as a
normal Claude Code session.

## 6. Clean up

Delete the test repo (or keep it, if useful as a reference) so it doesn't
get confused for the real thing later.

## If something breaks

Note exactly what failed — hook didn't run, skills missing, commit didn't
land — and fix it in the template repo itself (on a branch, via PR) before
sending the link to the person you built it for.
