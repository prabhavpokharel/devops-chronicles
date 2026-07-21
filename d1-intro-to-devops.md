# Introduction to DevOps

**Day 1 Notes**

---

## 1. What is DevOps?

DevOps is a set of practices, culture, and tooling that combines **Dev**elopment and **Op**erations into one continuous workflow, instead of treating them as separate teams with separate goals.

Traditionally:
- **Developers** wanted to ship new features fast.
- **Operations** wanted stability and didn't want anything to break production.

These two goals were in constant tension. DevOps is the discipline that removes that tension by making both teams (or one merged team) responsible for the *entire* lifecycle of software — writing it, testing it, deploying it, and keeping it running.

It's not a single tool or job title — it's a combination of:
- **Culture** (shared ownership, collaboration, no blame games)
- **Practices** (CI/CD, automation, monitoring)
- **Tools** (Git, Jenkins, Docker, Kubernetes, AWS, etc. — the tools we'll use throughout this course)

### Why is it needed?

Without DevOps, software delivery tends to break down in predictable ways as teams and products grow. The next section covers exactly what those breakdowns look like.

---

## 2. Problems DevOps Was Built to Solve

| Problem | What it looks like in practice |
|---|---|
| **The Silo Problem** | Dev and Ops sit in separate teams, don't talk, and often don't even use the same tools. Dev throws code "over the wall" to Ops and considers their job done. |
| **"Works on My Machine"** | Code runs fine on a developer's laptop but breaks in staging or production due to differing environments, dependencies, or configurations. |
| **Slow, Risky Releases** | Releases happen rarely (monthly/quarterly) because each one is a big, manual, high-risk event — so teams avoid releasing often, which makes each release even riskier when it does happen. |
| **Manual Everything** | Building, testing, and deploying are done by hand — slow, inconsistent, and error-prone. Humans doing repetitive tasks eventually make mistakes. |
| **No Feedback Loop** | Developers don't find out something broke in production until much later (if at all), so the same mistakes get repeated. |
| **Blame Culture** | When something breaks, the instinct is to find out *whose fault* it was (Dev vs Ops) rather than *how the system allowed it to happen* — which discourages people from taking risks or being transparent about issues. |

DevOps exists specifically to break this cycle — by unifying teams, automating the repetitive/error-prone parts, and building fast feedback loops so problems are caught early, not after the damage is done.

---

## 3. Why We Do DevOps: The Payoff

| Benefit | What it means |
|---|---|
| **Ship faster** | Smaller, more frequent releases instead of big risky ones. |
| **Break less** | Automation and testing catch problems before they reach users. |
| **Recover faster** | When something does break, automated pipelines and monitoring make it fast to detect, roll back, or fix. |
| **Happier teams** | Less manual grunt work, less blame, more ownership and trust between Dev and Ops. |
| **Happier customers** | Faster fixes, more reliable products, and features delivered quicker. |

The core idea: **speed and stability aren't actually opposites** — done right, automation and good practices give you both at once, instead of forcing a tradeoff.

---

## 4. How We Measure It: DORA Metrics

DORA (DevOps Research and Assessment) metrics are the industry-standard way to measure how well a team is actually "doing DevOps" — not by opinion, but by data. Four key metrics:

1. **Deployment Frequency** — how often code is deployed to production (daily? weekly? monthly?)
2. **Lead Time for Changes** — how long it takes from code being committed to it running in production
3. **Change Failure Rate** — what percentage of deployments cause a failure in production
4. **Mean Time to Recovery (MTTR)** — how long it takes to recover once something does break

High-performing DevOps teams score well on all four: frequent deployments, short lead times, low failure rates, and fast recovery. These metrics are what companies actually track to know if their DevOps practices are working.

---

## 5. Who Can Learn DevOps?

DevOps draws from multiple backgrounds, and none of them is a strict requirement on its own:

- **System Administrators** transitioning into automation and cloud
- **Software Developers** who want to own deployment and infrastructure, not just write code
- **QA/Testers** moving into automated testing and CI/CD pipelines
- **Fresh graduates / beginners** with a genuine interest in infrastructure, automation, and cloud computing

What matters more than background is comfort with the command line, a willingness to learn scripting, and curiosity about how systems actually run end-to-end.

---

## 6. Prerequisites

Roughly, a working comfort with:

- **Basic Linux / command-line usage** — navigating, managing files, running commands
- **Basic networking concepts** — IP addresses, ports, DNS basics
- **Basic programming/scripting logic** — doesn't need to be advanced, but shouldn't be a first-time experience
- **Willingness to work hands-on** — DevOps is learned by doing, not just reading

None of these need to be expert-level going in — the course builds them up — but zero exposure to a terminal will make Week 1 harder.

---

## 7. Workflow of a DevOps Engineer (Step 0 to Complete)

A simplified end-to-end view of what a DevOps engineer's pipeline looks like, from nothing to a running product:

1. **Plan** — requirements and work are defined (often alongside Dev/Product teams)
2. **Code** — developers write code, pushed to a version control system (Git)
3. **Build** — code is compiled/packaged into a deployable artifact (e.g. a Docker image)
4. **Test** — automated tests run against the build (unit, integration, etc.)
5. **Release** — the build is approved and versioned for deployment
6. **Deploy** — the release is pushed to servers/infrastructure (e.g. via Jenkins to Kubernetes/AWS)
7. **Operate** — the application runs in production; infrastructure is maintained
8. **Monitor** — logs, metrics, and alerts track the application's health
9. **Feedback loop** — issues found in monitoring flow back into planning/coding, closing the loop

This cycle (often drawn as the "infinity loop") repeats continuously — that's the whole point of DevOps: it's not a one-time project, it's an ongoing process.

---

## 8. Product-Based vs Service-Based Companies

| Aspect | Product-Based Company | Service-Based Company |
|---|---|---|
| **What they build** | Their own product, used by external customers (e.g. a SaaS platform) | Custom solutions/infrastructure for *client* companies |
| **DevOps focus** | Deep ownership of one system's scalability, uptime, and continuous improvement over a long timeframe | Setting up and adapting DevOps practices repeatedly across different client environments and tech stacks |
| **Pace/scope** | Long-term, iterative — same codebase evolves for years | Often project-based — different clients, different stacks, tighter deadlines |
| **Tooling** | Tends to standardize deeply on one set of tools/cloud provider | Needs to be flexible across multiple clients' existing tools and cloud providers |

In short: product-based DevOps is about mastering and evolving *one* system deeply; service-based DevOps is about adapting DevOps practices across *many* different systems and clients.

---

## 9. Joining a Company Where DevOps Is Already Set Up

If a senior DevOps engineer has already built the pipelines, infrastructure, and automation, a new joiner's first responsibilities typically look like:

1. **Understand the existing setup** — read documentation (if it exists), review the CI/CD pipelines, infrastructure-as-code, and architecture diagrams
2. **Get environment access** — credentials, VPN, cloud console access, repo access
3. **Shadow/observe** — watch how deployments, incidents, and on-call work currently happen
4. **Make small, safe changes first** — minor pipeline tweaks or documentation fixes to build familiarity without high risk
5. **Learn the monitoring/alerting setup** — know what "normal" looks like before you're expected to respond to "abnormal"
6. **Gradually take ownership** — of specific services, pipelines, or infrastructure components as trust and understanding build

The key skill here isn't building from scratch — it's **reading and understanding someone else's system safely**, which is arguably a harder and more common real-world skill than greenfield setup.

---

## 10. Tools Used by a DevOps Engineer

A non-exhaustive overview of the tool categories (many of these we'll cover hands-on in this course):

| Category | Example Tools |
|---|---|
| Version Control | Git, GitHub, GitLab |
| CI/CD | Jenkins, GitHub Actions, GitLab CI |
| Containerization | Docker |
| Orchestration | Kubernetes |
| Configuration Management / IaC | Ansible, Terraform |
| Cloud Platforms | AWS, Azure, GCP |
| Monitoring/Logging | Prometheus, Grafana, ELK stack |
| Scripting | Bash, Python |

---

## 11. Career Roles After This Training

DevOps skills open doors into several related but distinct roles. The tools overlap heavily, but the day-to-day focus differs:

| Role | Focus |
|---|---|
| **DevOps Engineer** | The generalist — owns CI/CD pipelines, automation, and the Dev↔Ops bridge end to end |
| **Platform Engineer** | Builds internal tools/platforms that let *other* developers self-serve infrastructure (e.g. an internal deployment portal) rather than working pipeline-by-pipeline |
| **Site Reliability Engineer (SRE)** | Focuses on uptime, reliability, and incident response — applies software engineering practices to operations problems (error budgets, SLOs/SLAs, on-call) |
| **DevSecOps Engineer** | DevOps + security baked into the pipeline itself — automated vulnerability scanning, secrets management, compliance as part of CI/CD, not bolted on after |
| **System Administrator (SysAdmin)** | The more traditional Ops role — manages servers, OS-level config, and infrastructure, often with less automation/coding focus than DevOps |
| **CloudOps Engineer** | Focuses specifically on managing and optimizing cloud infrastructure (AWS/Azure/GCP) — cost, scaling, security, uptime in a cloud-native context |
| **Developer (Startups)** | In smaller companies, "developer" often *includes* DevOps responsibilities by necessity — one person writing code, deploying it, and keeping it running, since there's no dedicated Ops team |

### Is CloudOps part of DevOps?

Not exactly a subset, but heavily overlapping — think of it this way:

- **DevOps** is a broader *practice/culture* spanning the whole software delivery lifecycle (code → build → test → deploy → operate → monitor), and can apply on-prem, hybrid, or cloud.
- **CloudOps** is more specifically about *operating and optimizing cloud infrastructure* — it's what Ops looks like when everything lives in the cloud.

In practice, most modern DevOps roles *are* heavily cloud-based, so the line blurs — many companies use "DevOps Engineer" and "CloudOps Engineer" almost interchangeably, but CloudOps is technically narrower (infrastructure-and-cloud focused) while DevOps is the wider discipline (culture + automation + collaboration + infrastructure).

---

## Assignment: Explore — What is a Fork Bomb?

**To research and write up:**
- What a fork bomb actually is (a process that recursively spawns copies of itself)
- Why it's relevant to a DevOps engineer (resource exhaustion, system stability, why process/resource limits like `ulimit` matter)
- How it connects back to today's theme — a *tiny* mistake (a bad script, an infinite loop) can spiral into a full system outage if there's no safeguard in place, which is exactly the kind of "manual/no-safety-net" failure DevOps practices are designed to prevent