# GUIDE.md — Student Hands-on Lab Guide

**Workshop:** Git 2: Automations, Workflow, & Pages

Welcome! In this lab you'll take a template repository containing a **College
GWA Calculator** web app, run it through a real two-branch Git workflow, watch
an automated CI Quality Gate catch a real bug, fix that bug with the help of a
repository-level AI mentor, and finish by deploying your own live, interactive
calculator to the public internet via GitHub Pages — all using only a free
GitHub account and your browser.

The entire app is plain **HTML, CSS, and JavaScript** — no frameworks, no
build tools, and no installs. Even the unit tests run with a command that's
already built into Node.js, which is already built into your Codespace.

By the end of this lab you will have practiced:

- Using a GitHub template to create your own repository
- Working entirely inside GitHub Codespaces (no local setup required)
- Running a real automated unit test suite (`node --test`)
- Branching, committing, and opening Pull Requests
- Reading and using repository-level AI instructions/agents
- Watching a CI pipeline and a CD (deployment) pipeline run automatically
- Visiting your own live GitHub Pages site

---

## Step 1 — Create Your Own Repository from the Template

1. Open the template repository link your facilitator gives you (it will look
   like `https://github.com/SLU-AWS-Cloud/git-series-automation-lab`).
2. Click the green **Use this template** button near the top-right of the
   page, then choose **Create a new repository**.
3. Set:
   - **Owner:** your own GitHub account.
   - **Repository name:** `gwa-calculator` (or any name you like).
   - **Visibility:** **Public** — this is required so your GitHub Pages site
     can be viewed by anyone later (and is free for public repos).
4. Click **Create repository**.
5. You now have your own independent copy of the repository, including the
   full commit history and, importantly, the `dev` branch as the default
   branch. Everything you do from here happens in *your* copy — nothing you
   do affects the original template or your classmates' repositories.

---

## Step 2 — One-Time GitHub Pages Configuration

Every new repository needs this set once, manually, before deployments can
publish anywhere.

1. In **your** new repository, go to the **Settings** tab.
2. In the left sidebar, click **Pages**.
3. Under **Build and deployment -> Source**, choose **GitHub Actions**
   (not "Deploy from a branch").
4. That's it — no save button needed, the selection applies immediately.
   You won't see a live URL yet; that appears after your first successful
   deployment in Step 8.

---

## Step 3 — Launch GitHub Codespaces from the `dev` Branch

1. Make sure you're on the `dev` branch (it should already be selected by
   default — check the branch dropdown near the top-left of the repository
   page; it should say `dev`).
2. Click the green **Code** button.
3. Select the **Codespaces** tab.
4. Click **Create codespace on dev**.
5. Wait for the Codespace to build. This repository has no special
   configuration file for Codespaces (no `.devcontainer/`) because it doesn't
   need one — the default Codespaces image already includes a current version
   of Node.js, which is all this project needs.
6. Once it finishes, you'll have a full VS Code environment running in your
   browser, with the repository already checked out. There is nothing to
   install — this project has zero external dependencies.

---

## Step 4 — Run the Tests and Observe the Intentional Failure

1. In the Codespace, open a terminal: **Terminal -> New Terminal** (or
   `` Ctrl+` ``).
2. Run:
   ```bash
   node --test tests/
   ```
3. You should see most tests pass, but **one test fails** — look for a line
   like:
   ```
   not ok 11 - computeSemesterGwa — zero included units throws InvalidInputError
   ```
   with an `AssertionError` underneath it.
4. This failure is **intentional** — it's the checkpoint for this lab. Your
   job over the next few steps is to understand why it fails and fix it
   properly, the same way you'd fix a real bug reported by a real test suite.

   Take a moment and open `public/gwaCalculator.js`. Find the function
   `computeSemesterGwa`. Notice the comment above the final `return`
   statement marked `INTENTIONAL BUG`.
5. Curious what this looks like from a user's perspective? Open
   `public/index.html` using the **Live Preview** or **Open in Browser**
   option in the Codespace (or wait until Step 8, when you'll see it live on
   the real internet). If you add a semester containing *only* an NSTP
   course, that semester's GWA will display as a literal **"NaN"** instead of
   a helpful message — that's the same bug the failing test caught, just
   visible in the UI instead of in a test report.

---

## Step 5 — Branch, Commit, and Open a Pull Request to See CI Fail

Even though you already see the failure locally, the workshop workflow wants
you to experience the same failure show up **automatically** on a Pull
Request, exactly like it would in a real team project.

1. Create a new branch for your fix:
   ```bash
   git checkout -b fix/gwa-boundary-checks
   ```
2. For now, commit *without* fixing anything yet, just to see the automated
   Quality Gate run and fail on GitHub (you'll push the real fix in Step 6):
   ```bash
   git commit --allow-empty -m "chore: open PR to observe CI Quality Gate"
   git push origin fix/gwa-boundary-checks
   ```
3. GitHub will print a URL in the terminal to open a Pull Request, or you can
   go to your repository on github.com — you'll see a yellow banner
   suggesting **Compare & pull request**. Click it.
4. Set the PR to go **from** `fix/gwa-boundary-checks` **into** `dev` (this
   should be the default base branch already, since `dev` is the default
   branch of the repo).
5. Give it a title like `fix: handle zero-units boundary condition` and click
   **Create pull request**.
6. Scroll down on the PR page. You'll see a check called **CI Quality Gate /
   quality-gate** running automatically. Click **Details** to watch it live.
7. It will finish with a red ✗, failing on the same zero-units test you saw
   locally. This is the **CI Quality Gate catching the bug automatically**,
   before any code reaches `dev` — exactly what it's designed to do.

Leave this Pull Request open; you'll push your real fix to the same branch in
Step 6 and watch this same check turn green.

---

## Step 6 — Consult the AI Instructions, Then Fix the Boundary Condition

### 6a. Read the repository's AI instructions and agent

Before writing the fix, read the two AI-facing files that live inside
`.github/` in your repository. These aren't just documentation for a human —
they're read by AI coding assistants (like GitHub Copilot, if you have it
enabled) before they offer you help on this repo.

1. Open `.github/instructions/academic-policy.md`.
   - Read **Section 1 (Grading Domain Rules)** and **Section 4 (Defensive
     Programming Constraints)**, specifically rule #1: *"JavaScript does NOT
     throw on division by zero... must explicitly check... and throw
     InvalidInputError."* This is the exact rule the failing test enforces.
2. Open `.github/agents/code-mentor.md`.
   - Notice this file defines an agent persona called **Code Mentor**, whose
     job is to help you understand *why* your code is wrong — not to hand you
     a finished patch. Read the section **Anti-Vibe-Coding Principles**.
   - "Vibe coding" means pasting an error message into an AI tool and getting
     back a working fix with zero understanding of what changed or why. This
     workshop is deliberately designed against that. The Code Mentor agent is
     instructed to never paste you a corrected function, and instead to ask
     you diagnostic questions until you find the fix yourself.
3. If you have access to an AI assistant configured to use this repository's
   agents (for example, GitHub Copilot Chat with agent support, or your
   facilitator's provided tool), you can literally ask it to act as
   **Code Mentor** and describe your failing test. Expect it to respond with
   a guiding question rather than a code block — that's it working as
   designed.
4. Why does this matter? This is **Explainable AI (XAI)** in practice: every
   hint the agent gives you is required to point back to a specific written
   rule (in `academic-policy.md`) and a specific failing test (in
   `tests/gwaCalculator.test.js`), so you always know *why* something is
   considered correct or incorrect — not just that an AI said so. You remain
   accountable for the actual fix you write and commit.

### 6b. Fix the boundary condition yourself

1. Open `public/gwaCalculator.js` and look at `computeSemesterGwa`:
   ```javascript
   const totalWeighted = included.reduce((sum, c) => sum + c.grade * c.units, 0);
   const totalUnits = included.reduce((sum, c) => sum + c.units, 0);

   // INTENTIONAL BUG (workshop checkpoint): totalUnits can
   // legitimately be 0 (e.g. an NSTP-only semester). This should
   // guard and throw InvalidInputError, but it does not yet.
   return totalWeighted / totalUnits;
   ```
2. Following rule #1 in `academic-policy.md` ("No unguarded division... must
   explicitly check... and throw InvalidInputError"), add a guard **before**
   the `return` that throws `new InvalidInputError(...)` when `totalUnits
   === 0`, with a message that explains what happened (e.g. that the
   semester has no gradable units once NSTP courses are excluded).
3. Save the file.
4. Run the syntax check and the full test suite again locally:
   ```bash
   node --check public/gwaCalculator.js
   node --check public/app.js
   node --test tests/
   ```
   All tests, including the zero-units test, should now pass.
5. Commit and push your real fix to the same branch:
   ```bash
   git add public/gwaCalculator.js
   git commit -m "fix: throw InvalidInputError on zero total units"
   git push origin fix/gwa-boundary-checks
   ```

---

## Step 7 — Watch CI Turn Green and Merge into `dev`

1. Go back to the Pull Request you opened in Step 5. Your new push
   automatically re-triggers the **CI Quality Gate** check.
2. Watch the **Actions** tab or the PR's checks section. This time both the
   **Syntax check** step and the **Run unit tests** step should complete with
   a green ✓.
3. Once the check is green, click **Merge pull request** (then **Confirm
   merge**) to merge your fix into `dev`.

---

## Step 8 — Open a Release PR into `main` and Watch the Live Deployment

1. Now create the release Pull Request: from the repository's **Pull
   requests** tab, click **New pull request**, and set it to go **from**
   `dev` **into** `main`.
   - Title it something like `release: deploy verified GWA calculator`.
   - Click **Create pull request**.
2. You'll see the **CI Quality Gate** check run on this PR too (it triggers
   on pull requests targeting `main` as well as `dev`), confirming one more
   time that everything going into production is verified.
3. Once that check is green, click **Merge pull request**, then **Confirm
   merge**.
4. This push to `main` automatically triggers the **Deploy GWA Calculator to
   GitHub Pages** workflow. Go to the **Actions** tab to watch it.
5. You'll see two jobs run in sequence:
   - **build** — packages everything inside `public/` (your HTML, CSS, and
     the two JavaScript files) as a deployable Pages artifact.
   - **deploy** — publishes that artifact to GitHub Pages.
6. When both jobs finish with a green ✓, click into the **deploy** job and
   look for the **page_url** output, or simply go back to **Settings ->
   Pages** in your repository — you'll now see a banner: *"Your site is live
   at `https://<your-username>.github.io/<your-repo-name>/`."*

---

## Step 9 — Visit Your Live Calculator

1. Click your GitHub Pages link. You should see your own, fully interactive
   **College GWA Calculator**, live on the internet:
   - Add a semester, add a few courses with units and grades.
   - Watch the Semester GWA and Overall Cumulative GWA update live.
   - Check "Is NSTP?" on a course and confirm it's excluded from the GWA and
     from Academic Standing, and that the notice about NSTP exclusion is
     visible.
   - Enter a high enough overall GWA and confirm the Academic Standing
     readout correctly shows Summa / Magna / Cum Laude at the 93 / 91 / 88
     boundaries.
2. Try the exact scenario the bug was about: create a semester with *only*
   an NSTP course in it. Before your fix, this showed a confusing "NaN."
   Now, because `computeSemesterGwa` throws a clear `InvalidInputError`,
   `app.js`'s error handling catches it and shows you a readable message in
   the page instead — the same defensive-programming fix you wrote is now
   protecting real users of your deployed app, not just passing a test.

**Congratulations** — you've just completed a full real-world software
delivery cycle: a failing automated test caught a genuine bug, repository-level
AI instructions guided (without doing) your fix, a CI Quality Gate verified
your work on every Pull Request, and a CD pipeline deployed your verified code
to a public URL automatically the moment it reached `main`. This is the same
pattern used by professional software teams — you just ran it yourself, end to
end, for free, entirely in the browser, with nothing to install.