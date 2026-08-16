# Testing a template before handing it off

Before sending a template repo (e.g. `citizen-project-template`) to the
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

If you search for the repo and get **"No repos match,"** that's not a bug in
the template — it means the "Claude" GitHub App is installed with access
limited to specific repos, and your brand-new one isn't in that list yet.
Fix it at [github.com/settings/installations](https://github.com/settings/installations)
→ **Claude** → **Configure** → switch to **All repositories** (this is a
one-time fix that covers every future repo too, so it's worth doing right
now rather than per-project). This is common enough that the template's own
README covers it up front — but you may hit it here first, on your own
account, before the citizen developer ever does.

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

While you're here, specifically check the two behaviors `CLAUDE.md` is
supposed to enforce, don't just assume they worked:

- **Did Claude show you the thing it built, not just describe it?** If you
  asked for anything visual (a webpage, a button), it should proactively
  publish a Claude Artifact and hand you a clickable link — this
  environment has no preview panel or port-forwarding, so if Claude doesn't
  do this, the person testing has no way to see their own work.
- **Did it explain any git/GitHub action in plain language before asking?**
  If it offers to commit, push, or open a pull request, it should say what
  that means and why in one plain sentence — not just ask "want a PR?" and
  assume the term is self-explanatory.
- **Did it actually tell you what to click, not just that it "saved"
  something?** A plain-language explanation that reads like the save
  already happened, with no instruction to click a specific button, is a
  gap — the person is left unsure whether they need to do anything. It
  should name the literal button (e.g. "Create PR").
- **Did it ask you to verify the result before merging, rather than merging
  immediately?** And did it explain what "merge" means at that point, not
  before? Skipping straight from "saved" to "merged" removes the one point
  where the person actually confirms their work still works.
- **Did it update the project's own README.md to describe what you just
  built?** After a save that meaningfully changes what the project is or
  does, check the repo's README — not this recipe's, the new project's own.
  If it still just says the generic "starting a new project" onboarding
  text with no mention of what's actually in the project now, that's a gap:
  the README should track reality, updated as part of the same save.

If any of these didn't happen, that's a `CLAUDE.md` gap — tighten the
instructions and retest, don't just note it and move on.

## 6. Clean up

Delete the test repo (or keep it, if useful as a reference) so it doesn't
get confused for the real thing later.

## If something breaks

Note exactly what failed — hook didn't run, skills missing, commit didn't
land — and fix it in the template repo itself (on a branch, via PR) before
sending the link to the person you built it for.
