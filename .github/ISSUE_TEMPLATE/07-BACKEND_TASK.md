---
name: Backend Task
about: A backend task. Defaults to a subtask of a parent issue
title: "[Backend] "
labels: kind/feature

---
<!-- Defaults to a subtask of a parent (tracking) issue. To use as a standalone/normal issue, overwrite the requirement section below with the actual details. -->

**What is the requirement and Why is this needed**:

The requirements are detailed in the parent issue.

---

**Completion checklist**:

- [ ] **Technical design (doc)**: <!-- link the design/approach; get maintainer review before coding -->
- [ ] **UT** <!-- unit tests for new domain/jobserver logic; patch coverage gate must pass -->
- [ ] **E2E** <!-- relevant e2e scenarios pass (smoke or targeted) -->
- [ ] **API change**: <!-- list new/changed proto or HTTP APIs, or "none" -->

The artifacts above should be described or linked in this checklist or the associated PR.
