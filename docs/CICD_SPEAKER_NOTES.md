# CI/CD for CDF - Speaker Notes

**Show & Tell + Handoff for Sylvamo Platform Team**

---

## PRESENTATION FLOW TRACKER

Use this to track where you are. Check off each section as you complete it.

| # | Section | Time | Status |
|---|---------|------|--------|
| 0 | [Framing (internal)](#framing-how-to-position-this-meeting) | - | ⬜ |
| 1 | [Opening](#slide-opening) | 2 min | ⬜ |
| 2 | [What's Already Done](#slide-whats-already-done-cognite-setup) | 3 min | ⬜ |
| 3 | [Live Demo - Repo](#slide-live-demo---the-repository) | 5 min | ⬜ |
| 4 | [Live Demo - PR Workflow](#slide-live-demo---a-pr-workflow) | 5 min | ⬜ |
| 5 | [What YOUR Team Owns](#slide-what-your-team-owns) | 2 min | ⬜ |
| 6 | [Action Items](#your-action-items) | 8 min | ⬜ |
| 7 | [Developer Workflow](#slide-the-workflow-your-developers-will-follow) | 3 min | ⬜ |
| 8 | [Quick Reference](#slide-quick-reference---what-we-covered) | 2 min | ⬜ |
| 9 | [Questions & Next Steps](#slide-questions--next-steps) | 5 min | ⬜ |
| | **TOTAL** | **~35 min** | |

---

## LINKS TO HAVE OPEN (Prep Before Meeting)

Open these tabs before the meeting starts:

| Tab | URL | When to Show |
|-----|-----|--------------|
| **ADO Repo** | [Industrial-Data-Landscape-IDL](https://dev.azure.com/SylvamoCorp/_git/Industrial-Data-Landscape-IDL) | Slides 3, 4 |
| **ADO Variable Groups** | Project Settings → Pipelines → Library | Slide 3 |
| **ADO Pipelines** | Pipelines section | Slide 4 |
| **GitHub Docs** | [CICD_OVERVIEW.md](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md) | Reference if needed |
| **GitHub README** | [sylvamo-data-model](https://github.com/fbarsoba-cognite/sylvamo-data-model#cicd-for-cdf) | Reference if needed |

---

## FRAMING: How to Position This Meeting

**INTERNAL NOTE (don't say this out loud):**

The goal is to:
1. **Show what we built** - demo the working CI/CD setup
2. **Transfer understanding** - explain how it works so they can own it
3. **Give them clear ownership** - specific things THEY need to do going forward

Avoid making it sound like "we did everything, you're done." Instead: "we set up the foundation, here's what you own now."

---

## SLIDE: Opening

**⏱️ ~2 minutes**

**📺 SHOW:** Title slide or just your face (no screen share yet)

**📄 GITHUB REF:** [CICD_OVERVIEW.md - TL;DR section](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#cicd-for-cognite-data-fusion-cdf)

**SPEAKER NOTES:**

Thanks for joining. Today I want to walk you through the CI/CD setup we've put in place for CDF deployments.

I'll show you:
1. **What we've built** - the pipelines, the configuration, how it all works
2. **How you'll use it** - the day-to-day workflow
3. **What you own going forward** - the pieces that your team will manage

By the end, you'll have a clear picture of the system and know exactly what actions you need to take.

---

## SLIDE: What's Already Done (Cognite Setup)

**⏱️ ~3 minutes**

**📺 SHOW:** Can share screen now - show the table below or a simple slide

**📄 GITHUB REF:** [CICD_OVERVIEW.md - Tech Stack](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#the-cicd-tech-stack)

**SPEAKER NOTES:**

Let me start by showing you what's already in place. We've set up the core CI/CD infrastructure so you don't have to build it from scratch.

**What Cognite has configured:**

| Component | Status | Details |
|-----------|--------|---------|
| Toolkit repository | Done | Industrial-Data-Landscape-IDL in your ADO |
| Pipeline YAML files | Done | `.devops/` folder with dry-run and deploy pipelines |
| Docker image reference | Done | Using `cognite/toolkit:0.5.35` |
| Config file structure | Done | `config.dev.yaml`, `config.staging.yaml`, `config.prod.yaml` |
| Variable Groups | Done | `dev-toolkit-credentials` created with all required vars |
| Service Principal | Done | App registration in Entra ID with CDF access |

This is the foundation. You don't need to set any of this up - it's working today.

Let me show you where everything lives...

*[Navigate to ADO repo and show the structure]*

---

## SLIDE: Live Demo - The Repository

**⏱️ ~5 minutes**

**📺 NAVIGATE TO:** [ADO Repo - Industrial-Data-Landscape-IDL](https://dev.azure.com/SylvamoCorp/_git/Industrial-Data-Landscape-IDL)

**📄 GITHUB REF:** [CICD_OVERVIEW.md - Repository Structure](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#repository-structure)

**SHOW IN ORDER:**
1. Root folder structure → `sylvamo/` directory
2. Config files → `config.dev.yaml`, `config.staging.yaml`, `config.prod.yaml`
3. Modules folder → `sylvamo/modules/`
4. Pipeline files → `.devops/` folder
5. Variable Groups → Project Settings → Pipelines → Library

**SPEAKER NOTES:**

*[Share screen - show ADO repo]*

Here's the Industrial-Data-Landscape-IDL repository. Let me walk you through the key pieces:

**The config files:**
- `sylvamo/config.dev.yaml` - points to your DEV CDF project
- These define which modules get deployed and to which environment

**The modules folder:**
- `sylvamo/modules/` - this is where all the CDF resources are defined
- Data models, transformations, access groups - everything is YAML

**The pipeline files:**
- `.devops/dry-run-pipeline.yml` - validates PRs
- `.devops/deploy-pipeline.yml` - deploys on merge

*[Show the Variable Groups in ADO Project Settings]*

And here are the Variable Groups we created - `dev-toolkit-credentials`. This contains your service principal credentials. The secret is masked - you can see it says "(secret)" - so it's secure.

---

## SLIDE: Live Demo - A PR Workflow

**⏱️ ~5 minutes**

**📺 NAVIGATE TO:** ADO Pipelines → Find a recent pipeline run (or create a PR beforehand)

**📄 GITHUB REF:** [CICD_OVERVIEW.md - CI/CD Flow](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#cicd-flow-overview)

**SHOW IN ORDER:**
1. A PR (existing or create one before meeting)
2. Pipeline run triggered by the PR
3. `cdf build` step output
4. `cdf deploy --dry-run` output (the key part!)
5. Show that nothing changed in CDF yet

**💡 TIP:** Create a dummy PR before the meeting so you have a clean example to show.

**SPEAKER NOTES:**

Let me show you what happens when someone makes a change.

*[Create a small PR or show an existing one]*

When a PR is created, the dry-run pipeline triggers automatically. Let's look at the output...

*[Show pipeline run]*

You can see:
1. `cdf build` - validated the configuration, no errors
2. `cdf deploy --dry-run` - shows what WOULD change

This output tells reviewers exactly what will happen if this PR is merged. Nothing actually changes in CDF yet - this is just a preview.

*[If possible, show the dry-run output with resource changes]*

When the PR is merged to main, the deploy pipeline runs and actually applies these changes.

This is the workflow your developers will use every day.

---

## SLIDE: What YOUR Team Owns

**⏱️ ~2 minutes**

**📺 SHOW:** Transition slide - "What You Own" or just say it

**📄 GITHUB REF:** [CICD_OVERVIEW.md - Key Takeaways](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#key-takeaways-for-platform-teams)

**🎯 THIS IS THE KEY TRANSITION:** Shift from "show and tell" to "your responsibilities"

**SPEAKER NOTES:**

Now let's talk about what your team is responsible for going forward. We built the foundation, but **you own the system now**.

Here's what falls under your team's responsibility:

---

## YOUR ACTION ITEMS

**⏱️ ~8 minutes (spend ~1 min per item)**

**📺 SHOW:** The action items list (you can screen share this doc or a slide)

**📄 GITHUB REF:** See [Cheat Sheet](#cheat-sheet-action-items-summary) at the bottom for quick reference

**🎯 GOAL:** They should leave knowing exactly what THEY need to do

**SPEAKER NOTES:**

Let me be specific about what you need to do:

### 1. Enable Branch Policies (Required)

**What:** Configure the dry-run pipeline as a required check on PRs to main.

**Why:** This prevents broken configurations from being merged. Right now the pipeline runs, but PRs can still be merged even if it fails.

**How:** 
- Go to Repos → Branches → main → Branch policies
- Add a Build Validation policy
- Select the dry-run pipeline
- Set to "Required"

**Who:** Your ADO admin

**When:** This week

---

### 2. Set Up Approval Gates for Production (Required)

**What:** Add manual approval before deploying to staging and prod.

**Why:** You probably don't want every merge to automatically deploy to production. Approvals give you a checkpoint.

**How:**
- Create ADO Environments: `cdf-staging`, `cdf-prod`
- Add approvers to each environment
- Update deploy pipeline to use environments

**Who:** Your platform team

**When:** Before you start deploying to staging/prod

---

### 3. Own the Variable Groups (Ongoing)

**What:** You now own the credentials in the Variable Groups.

**Your responsibilities:**
- **Access control** - manage who can view/edit the Variable Groups
- **Secret rotation** - when you rotate the service principal secret, update it here
- **Audit** - periodically review access and usage

**Where:** Project Settings → Pipelines → Library → Variable Groups

**Who:** Your platform/security team

---

### 4. Monitor Pipeline Runs (Ongoing)

**What:** Keep an eye on pipeline success/failure rates.

**Why:** Failed pipelines mean changes aren't deploying. You need visibility.

**How:**
- Check the Pipelines section regularly
- Consider setting up notifications for failed runs
- Review dry-run output before approving PRs

**Who:** Your platform team + developers doing code review

---

### 5. Onboard Your Developers (This Month)

**What:** Train your developers on the workflow.

**Key things they need to know:**
- All CDF changes go through Git - no manual changes in the UI
- Create a branch, make changes, open a PR
- Review the dry-run output before approving
- Merge to main to deploy

**Who:** You, with our support if needed

---

### 6. Extend to Staging and Production (Future)

**What:** We've set up DEV. You'll need to extend to staging and prod.

**What's needed:**
- Create Variable Groups for staging and prod credentials
- Update pipelines with multi-stage deployment
- Add the approval gates mentioned above

**Who:** Your platform team (we can assist)

**When:** When you're ready to promote beyond dev

---

## SLIDE: The Workflow Your Developers Will Follow

**⏱️ ~3 minutes**

**📺 SHOW:** The workflow diagram below (can share this doc)

**📄 GITHUB REF:** [CICD_OVERVIEW.md - CI/CD Flow diagram](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#cicd-flow-overview)

**🎯 GOAL:** Simple mental model they can explain to their developers

**SPEAKER NOTES:**

Let me summarize the day-to-day workflow for your developers:

```
1. Create a feature branch
2. Make changes to YAML files in the modules/ folder
3. Push and create a PR
4. Pipeline runs automatically - check the dry-run output
5. Code review - reviewer checks the dry-run to see what will change
6. Merge to main
7. Deploy pipeline runs - changes applied to CDF
```

This is GitOps for CDF. Everything is tracked in Git, reviewable, auditable.

The key mindset shift: **no more manual changes in the CDF UI**. If it's not in Git, it doesn't exist.

---

## SLIDE: Quick Reference - What We Covered

**⏱️ ~2 minutes**

**📺 SHOW:** Summary slide or the recap below

**📄 GITHUB REF:** [README - CI/CD for CDF section](https://github.com/fbarsoba-cognite/sylvamo-data-model#cicd-for-cdf)

**🎯 GOAL:** Reinforce the done vs. their responsibility split

**SPEAKER NOTES:**

Let me recap:

**What Cognite set up (done):**
- Repository structure
- Pipeline YAML files
- Variable Groups with credentials
- Service principal with CDF access

**What Sylvamo owns (your responsibility):**
- Branch policies - enable them
- Approval gates - set them up for prod
- Variable Groups - manage access and rotate secrets
- Pipeline monitoring - watch for failures
- Developer onboarding - train your team
- Staging/prod extension - when ready

**Key URLs:**
- Repo: https://dev.azure.com/SylvamoCorp/_git/Industrial-Data-Landscape-IDL
- Variable Groups: Project Settings → Pipelines → Library
- Branch Policies: Repos → Branches → main → ⋮ → Branch policies

---

## SLIDE: Questions & Next Steps

**⏱️ ~5 minutes**

**📺 SHOW:** The checklist below (can share this doc or send afterward)

**📄 SEND AFTER MEETING:** 
- Link to this doc: [CICD_SPEAKER_NOTES.md](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_SPEAKER_NOTES.md)
- Or just the [Cheat Sheet](#cheat-sheet-action-items-summary)

**🎯 GOAL:** Confirm who owns what, get verbal commitment on timelines

**SPEAKER NOTES:**

Before we wrap up, let me confirm next steps:

**This week:**
- [ ] Enable branch policies on main (your ADO admin)
- [ ] Review Variable Group access (your security team)

**This month:**
- [ ] Onboard 1-2 developers to try the workflow
- [ ] Plan staging/prod rollout timeline

**We're here to help:**
- Questions on the setup - reach out anytime
- Staging/prod extension - we can pair on this
- Developer training - we can join a session if helpful

Any questions?

---

## APPENDIX: Technical Details (If They Ask)

**📄 GITHUB REF:** [CICD_OVERVIEW.md - Authentication Model](https://github.com/fbarsoba-cognite/sylvamo-data-model/blob/main/docs/CICD_OVERVIEW.md#authentication-model)

**💡 USE ONLY IF:** Someone asks technical questions about auth, secrets, or commands. Don't present this proactively.

### How Authentication Works

The pipeline authenticates to CDF using OAuth2 client credentials:

1. Variable Group contains: `IDP_CLIENT_ID`, `IDP_CLIENT_SECRET`, `IDP_TENANT_ID`
2. Pipeline injects these as environment variables
3. Toolkit CLI reads them and gets an access token from Entra ID
4. Token is used for all CDF API calls

### The Commands

- `cdf build` - Validates YAML, compiles configuration
- `cdf deploy --dry-run` - Shows what would change (no actual changes)
- `cdf deploy` - Applies changes to CDF

### Environment Variables Reference

| Variable | Value | Where It Comes From |
|----------|-------|---------------------|
| `LOGIN_FLOW` | `client_credentials` | Variable Group |
| `CDF_CLUSTER` | `westeurope-1` | Variable Group |
| `CDF_PROJECT` | `sylvamo-dev` | Variable Group |
| `IDP_CLIENT_ID` | `<app-id>` | Variable Group (from Entra ID) |
| `IDP_CLIENT_SECRET` | `<secret>` | Variable Group (secret, masked) |
| `IDP_TENANT_ID` | `<tenant-id>` | Variable Group |

---

## CHEAT SHEET: Action Items Summary

| # | Action | Owner | Timeline | Priority |
|---|--------|-------|----------|----------|
| 1 | Enable branch policies on main | ADO Admin | This week | Required |
| 2 | Set up approval gates for staging/prod | Platform Team | Before prod deploy | Required |
| 3 | Review Variable Group access controls | Security Team | This week | Required |
| 4 | Set up pipeline failure notifications | Platform Team | This month | Recommended |
| 5 | Onboard 1-2 developers to workflow | Platform Team | This month | Required |
| 6 | Plan staging/prod extension | Platform + Cognite | This quarter | Future |

---

*Speaker notes prepared: February 4, 2026*
