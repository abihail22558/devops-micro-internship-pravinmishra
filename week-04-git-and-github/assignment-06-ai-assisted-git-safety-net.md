# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![task 0](./screenshots/ass6task0.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

A dedicated feature branch keeps the main branch stable and protected while I work on the assignment. It allows me to develop, test, and review the changes independently before merging them, reducing the risk of introducing incomplete or unwanted changes into main.


---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![git status](./screenshots/ass6task1.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

This assignment used an obviously fake AWS access key because the purpose of this task is to safely test the Git safety checks without exposing real credentials. Using a fake value allows the pre-commit hook to detect a secret-like pattern while preventing accidental access to an actual AWS account or resources. This demonstrates how secret detection can be tested safely in a controlled environment.


---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![hooks pre commit](./screenshots/ass6task2m1.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![hooks ](./screenshots/ass6task2m2.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

The hook is tracked in the repository so the Git safety rule is version-controlled and can be shared consistently with anyone working on the project. A hook stored only in `.git/hooks/` is local to one developer's machine and is not included when the repository is cloned. Keeping it in the repository makes the guardrail visible, reviewable, and reproducible across the development environment.


---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

Both hooks act as preventive guardrails, but they protect different boundaries. The Git pre-commit hook runs before a Git commit is created and checks staged changes for conditions such as secret-like strings and oversized files. The PreToolUse hook from Week 2 runs before Claude Code executes a tool action and can prevent unsafe commands or operations. Both follow the same principle of detecting risky actions before they cross an important execution boundary.


---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![access blocked](./screenshots/ass6task3.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

The line that triggered the hook was `AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP`. The hook detected it because the pre-commit rule searches for the pattern `AKIA[0-9A-Z]{16}`, which matches the `AKIA` prefix followed by 16 uppercase letters or numbers. This is a common pattern used to identify AWS access-key-shaped strings.


---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No. The hook uses fixed regular-expression patterns, so it only detects strings that match the patterns it was specifically designed to identify. A secret stored under a poorly named variable or using a different credential format could pass this check. This demonstrates the limitation of deterministic rules and why additional analysis, such as the `/pr-ready` Claude Code skill, can provide another layer of review.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![allowed tools](./screenshots/ass6task4m1.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![flagged ](./screenshots/ass6task4m2.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

Bash and Read are included because the skill needs to inspect the repository state and staged Git changes. Write is intentionally excluded because `/pr-ready` is designed to be a read-only review and drafting tool. Preventing file-writing access creates a clear safety boundary: Claude can identify problems and prepare a PR draft, but the human must make the actual changes.


---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

Both checks identified the credential-shaped AWS key, but `/pr-ready` provided broader analysis. The pre-commit hook uses fixed patterns and blocked the commit because the staged file contained an AWS key-shaped string. `/pr-ready` also identified the debug `echo` statement and explained why the credential-shaped value should be removed or replaced. This shows the difference between deterministic rule-based protection and broader AI-assisted review.


---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![git commit successful](./screenshots/ass6task5m1.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![pr ready](./screenshots/ass6task5m2a.png)
![pr ready](./screenshots/ass6task5m2b.png)
![pr ready](./screenshots/ass6task5m2c.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the hardcoded secret from `scripts/notify.sh` and removed the debug `echo` statement. This satisfied the pre-commit hook because the script no longer contained a credential or debug statement that could trigger the safety checks. After making the changes, the commit completed successfully without a `BLOCKED` message.


---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![pull request](./screenshots/ass6task6.png)

---

#### PR Link

https://github.com/abihail22558/devops-micro-internship-pravinmishra/pull/1

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I reviewed the AI-generated PR description and edited parts that did not accurately reflect the changes I made. I wanted the final description to clearly match the work completed in Assignment 6 instead of blindly using the generated content.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The PR description could contain incorrect, incomplete, or misleading information about the changes. It could also describe work that was not actually completed, which would make the PR less accurate and could cause confusion during review.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

The PR should target my own fork because the changes were made in my personal branch and should be reviewed and merged within my fork. Targeting the shared upstream repository could unintentionally request changes to the main project instead of keeping my assignment work within my own repository.
---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

The Gather stage happens when the pre-commit hook and the /pr-ready skill inspect the repository and collect information about the current Git state, changed files, and possible issues such as secrets, debug statements, or TODO/FIXME comments.

---

**2. Which step(s) represent Analyze?**

The Analyze stage happens when /pr-ready reviews the gathered information and checks the changes against the defined safety rules. It then identifies potential risks and prepares a PR title and description based on what it found.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

The Human Act stage is when I review the AI's findings and then manually run commands such as git commit and git push and open the Pull Request. A human must perform these actions because the AI should not have control over publishing changes or creating commits without human review and approval. This keeps the human responsible for the final decision and prevents an AI mistake from being automatically pushed to the repository.

---

**4. Which step is Verify?**

The Verify stage happens when the pre-commit hook checks the changes before the commit is accepted and when /pr-ready is run again to confirm that the issues have been fixed and the repository is ready for a Pull Request.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

The fixed-rule pre-commit hook provides a consistent safety check that automatically blocks specific problems before they are committed. The AI skill provides a broader review and helps explain the changes, identify potential risks, and prepare the Pull Request, so the two tools provide different layers of protection.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

https://lnkd.in/p/drv9KJgC

---

## Key Learnings

What I learned this week:
Add 3-5 bullet points on what you learned this week.

- Fixed-rule checks provide consistent protection against known risks.
- AI can review changes more broadly and help identify potential issues that rules may not cover.
- AI-generated suggestions still need to be reviewed by a human for accuracy.
- Critical Git actions such as git commit, git push, and opening a Pull Request should remain under human control.
- Combining automation with human judgment can make development workflows safer without removing accountability.

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

https://github.com/abihail22558/devops-micro-internship-pravinmishra

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
