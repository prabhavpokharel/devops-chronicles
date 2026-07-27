# Introduction to DevOps — The Story Behind the Practice

## The Divided City

Picture a city with two tribes living on opposite sides of a wall.

On one side: the **builders**, who design and construct new things. On the other: the **keepers**, who maintain the city and make sure nothing falls apart. They rarely speak to each other. Builders finish something, toss it over the wall, and consider their job done. Keepers catch whatever lands, often with no idea how it was built or why it behaves the way it does.

This is roughly the state software development was in before DevOps existed — developers on one side, operations teams on the other, each with different goals and almost no communication between them.

## Why the Old Way Broke Down

That divide created a predictable set of problems:

- **The silo problem.** Development and operations worked in isolation, often using different tools and having no shared visibility into each other's work.
- **"Works on my machine."** Code would run perfectly on a developer's laptop, then fail the moment it reached a different environment — different OS versions, different configurations, different dependencies.
- **Slow, risky releases.** Because releasing software was a big, manual, high-stakes event, teams avoided doing it often. Releases piled up, which made each one riskier still.
- **Manual everything.** Builds, tests, and deployments were done by hand — slow, repetitive, and prone to human error.
- **No feedback loop.** A bug introduced today might not surface until months later, by which point nobody remembers what changed or why.
- **Blame culture.** When something broke, the instinct was to find out whose fault it was rather than fix the process that allowed it to happen in the first place.

## What DevOps Actually Is

DevOps is the practice of tearing down that wall — merging development and operations into a single, continuous workflow where the same people (or tightly collaborating teams) are responsible for writing code, deploying it, and keeping it running.

It isn't a single tool or a job title. It's three things working together:

- **Culture** — shared ownership, collaboration, no more finger-pointing when something breaks.
- **Practices** — continuous integration, continuous deployment, automation, monitoring.
- **Tools** — Git, Jenkins, Docker, Kubernetes, AWS, and the rest of what this course will cover.

## The Payoff

Done well, DevOps delivers on five fronts at once:

| Before | After |
| --- | --- |
| Releases are rare and risky | Releases are small, frequent, and routine |
| Failures are common | Failures happen far less often |
| Recovery takes days | Recovery takes minutes |
| Teams work in tension | Teams share ownership and trust |
| Customers wait for fixes | Customers get fixes fast |

The core insight is that speed and stability aren't actually in tension. Done right, automation and good practice deliver both — you don't have to trade one for the other.

## Measuring It: DORA Metrics

You can't manage what you don't measure, and the industry-standard way to measure DevOps maturity is the four DORA metrics (from Google's DevOps Research and Assessment team):

1. **Deployment frequency** — how often code ships to production.
2. **Lead time for changes** — how long from a commit to that code running live.
3. **Change failure rate** — what percentage of deployments cause a production failure.
4. **Mean time to recovery (MTTR)** — how fast the team recovers once something does break.

High-performing teams score well across all four — not just one. A team that deploys constantly but breaks production every time isn't actually doing well by DORA's standard.

## Who Can Learn This

DevOps draws people from several different starting points, and none of them is a strict requirement on its own:

- System administrators moving toward automation and cloud
- Developers who want to own deployment, not just write code
- QA engineers moving into automated testing and pipelines
- Complete beginners with genuine curiosity about how systems run end to end

**Prerequisites** are modest — comfort with a command line, a basic grasp of networking concepts (IP addresses, ports), and a willingness to learn by doing rather than by reading alone. None of it needs to be expert-level going in.

## The DevOps Engineer's Workflow, Start to Finish

A simplified view of the full cycle, from an idea to a running product:

1. **Plan** — define what needs to be built.
2. **Code** — developers write it, push to version control.
3. **Build** — the code is packaged into a deployable artifact (often a Docker image).
4. **Test** — automated tests run against the build.
5. **Release** — the build is versioned and approved.
6. **Deploy** — it's pushed to live infrastructure.
7. **Operate** — the application runs in production.
8. **Monitor** — logs, metrics, and alerts track its health.
9. **Feedback** — anything found in monitoring flows back into planning, closing the loop.

This isn't a one-time project. It's a continuous cycle — that's the entire point of the discipline.

## Product Companies vs Service Companies

A **product-based company** owns one system and lives with it for years — the same codebase, evolving continuously, deeply optimized for that one product's scale and reliability needs.

A **service-based company** works across many different clients, each with their own stack, their own constraints, and often tighter deadlines. The DevOps work here is less about mastering one system deeply and more about adapting quickly to many different environments.

Same underlying skill set, very different day-to-day rhythm.

## Joining a Company Where Everything Is Already Built

If you join a team where a senior engineer has already built out the pipelines and infrastructure, the job in your first weeks isn't to rebuild anything — it's to understand what's already there without breaking it:

1. Read whatever documentation exists, and review the existing pipelines and architecture.
2. Get access — credentials, VPN, repo and cloud console permissions.
3. Shadow the team before touching anything critical.
4. Make small, low-risk changes first to build familiarity.
5. Learn what "normal" looks like on the monitoring dashboards before you're expected to respond to "abnormal."
6. Gradually take ownership of specific services or pipelines as trust builds.

Reading and safely operating inside someone else's system is, in practice, a more common and arguably harder skill than building one from scratch.

## The Toolbox

| Category | Examples |
| --- | --- |
| Version control | Git, GitHub, GitLab |
| CI/CD | Jenkins, GitHub Actions, GitLab CI |
| Containerization | Docker |
| Orchestration | Kubernetes |
| Infrastructure as code | Ansible, Terraform |
| Cloud platforms | AWS, Azure, GCP |
| Monitoring | Prometheus, Grafana, ELK stack |
| Scripting | Bash, Python |

## Career Paths This Opens Up

| Role | Focus |
| --- | --- |
| DevOps Engineer | The generalist — owns the pipeline end to end |
| Platform Engineer | Builds internal tools so other developers can self-serve infrastructure |
| Site Reliability Engineer (SRE) | Uptime and reliability, treated as an engineering discipline rather than firefighting |
| DevSecOps Engineer | DevOps with security built into the pipeline itself, not added afterward |
| System Administrator | The more traditional operations role, less automation-focused |
| CloudOps Engineer | Managing and optimizing infrastructure specifically in the cloud |
| Developer at a startup | Often does all of the above at once, out of necessity |

**Is CloudOps part of DevOps?** Not exactly a subset — more of an overlapping specialty. DevOps is the broader culture and practice spanning the entire delivery lifecycle, applicable whether infrastructure lives on-prem, hybrid, or in the cloud. CloudOps is specifically what operations looks like once everything lives in the cloud. In practice, most modern DevOps work is cloud-based, so the line blurs — but conceptually, DevOps is the wider discipline and CloudOps is one context it's applied in.

## Assignment: Fork Bomb

Research and write up:

- What a fork bomb actually is — a process that recursively spawns copies of itself until it exhausts system resources.
- Why it matters to a DevOps engineer — it's a clean example of how resource limits (`ulimit`, process caps) protect a system from a single runaway script.
- How it ties back to today's theme — a tiny, unguarded mistake can spiral into a full outage. That's precisely the class of failure DevOps practices (automation, monitoring, safeguards) are designed to catch before it happens.
