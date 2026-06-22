# Roadmap – Thought‑Bridge

> **Thought‑Bridge** is an Axentx product that bridges high‑level AI reasoning with low‑level code generation, prioritising correctness over raw speed.  
> This document outlines the release plan from the MVP launch to the first two major feature waves.

| Phase | Release | Theme | Key Deliverables | MVP‑Critical |
|-------|---------|-------|------------------|--------------|
| **MVP** | **v0.1.0 – Launch** | Core correctness engine | • Dual‑stage pipeline: <br> 1️⃣ High‑level reasoning (LLM + verifier) <br> 2️⃣ Low‑level code generation (LLM + static‑analysis guard) <br>• Built‑in unit‑test generator and automated test harness <br>• Web UI for prompt → code → test feedback loop <br>• CI/CD pipeline with automated regression tests | ✔ |
| **v1.0** | **Q3 2026** | Reliability & Productivity | • **Version‑controlled code review**: Git‑based diff preview + auto‑merge suggestions <br>• **Context‑aware knowledge base**: Pull in project docs, API specs, and previous commits to inform generation <br>• **Custom LLM fine‑tuning**: On‑the‑fly fine‑tuning with user‑supplied examples <br>• **Performance monitoring**: Latency & accuracy dashboards <br>• **Marketplace integration**: Exportable snippets for VS Code & JetBrains | ✔ |
| **v2.0** | **Q1 2027** | Collaboration & Scaling | • **Multi‑user collaboration**: Shared workspaces, role‑based permissions <br>• **AI‑driven pair programming**: Real‑time suggestions & conflict resolution <br>• **Enterprise‑grade security**: End‑to‑end encryption, audit logs, SSO <br>• **Plugin ecosystem**: Third‑party extensions for CI tools, cloud providers, and data pipelines <br>• **Self‑healing**: Automatic rollback on failing tests, sandboxed execution |  |

---

## Detailed Milestones

### MVP (v0.1.0)

| Milestone | Description | Acceptance Criteria |
|-----------|-------------|---------------------|
| **1. Dual‑stage pipeline** | Implement a two‑step generation: (a) LLM produces a high‑level plan; (b) LLM generates code from the plan. | • Plan and code are produced within 30 s for a 200‑line code sample. |
| **2. Verification layer** | Add a verifier that checks the plan against the code and runs static analysis. | • No false positives on 100 unit tests from the `auto` dataset. |
| **3. Test harness** | Auto‑generate unit tests and run them in a sandbox. | • 95 % of generated tests pass on the initial code. |
| **4. Web UI** | Simple interface: prompt → plan → code → tests → results. | • Users can see the plan, code, and test output in a single page. |
| **5. CI/CD** | GitHub Actions that run the pipeline on every PR. | • Pipeline passes on 10 public repos with no manual intervention. |

### v1.0 – Reliability & Productivity

| Feature | Goal | Success Metric |
|---------|------|----------------|
| **Version‑controlled code review** | Enable diff‑based review and auto‑merge. | 80 % of PRs merged automatically after passing tests. |
| **Context‑aware knowledge base** | Use project docs to improve relevance. | 30 % reduction in manual edits for generated code. |
| **Custom LLM fine‑tuning** | Allow users to fine‑tune on their own codebases. | Fine‑tuned model improves accuracy by 15 % on internal benchmarks. |
| **Performance monitoring** | Provide latency & accuracy dashboards. | 90 % of requests served < 2 s. |
| **Marketplace integration** | Export snippets to IDEs. | 50 % of users install VS Code extension. |

### v2.0 – Collaboration & Scaling

| Feature | Goal | Success Metric |
|---------|------|----------------|
| **Multi‑user collaboration** | Shared workspaces with role permissions. | 70 % of teams use shared workspace in beta. |
| **AI‑driven pair programming** | Real‑time suggestions & conflict resolution. | 60 % reduction in merge conflicts. |
| **Enterprise security** | End‑to‑end encryption, audit logs, SSO. | 100 % compliance with ISO 27001. |
| **Plugin ecosystem** | Third‑party extensions. | 10 + plugins in marketplace. |
| **Self‑healing** | Automatic rollback on failing tests. | 95 % of failures automatically reverted. |

---

## Roadmap Principles

1. **Correctness First** – Every release must maintain or improve the accuracy of generated code.  
2. **Incremental Delivery** – Each milestone is shippable and provides measurable value.  
3. **Data‑Driven** – Use the `auto`, `instr-resp`, `messages`, and `query-resp` datasets to benchmark and iterate.  
4. **Open‑Source Friendly** – Leverage existing verified frameworks (e.g., `vllm`, `sglang`) and publish our own libraries under MIT.  
5. **Customer‑Centric** – Gather feedback from early adopters via the web UI and GitHub issues to refine priorities.

---

### Next Steps

1. **Finalize MVP scope** – Confirm feature list with PM and QA.  
2. **Set up CI/CD** – Integrate with GitHub Actions and `arkashira/surrogate-1-harvest`.  
3. **Build MVP UI** – Prototype with React + Tailwind.  
4. **Run internal beta** – Deploy to a staging environment and collect metrics.  
5. **Launch** – Public release on GitHub and publish documentation.

---
