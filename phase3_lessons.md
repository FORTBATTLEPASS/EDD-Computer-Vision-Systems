# 🎨 Phase 3 — Design & Planning (Days 13–15)

**PLTW Engineering Design & Development | Lynwood High School**
**Computer Vision Systems Unit**

With all four CV skill modules from Phase 2 complete, Phase 3 shifts from
writing detection code to designing the final system your team will build
and test in Phase 4. This is a pure design-thinking phase — no new code is
required, but the decisions made here will shape everything you build next.

> **Key:** 🎨 = Design deliverable | 📓 = Notebook entry due

***

## 📅 Phase 3 Schedule

| Day | Topic | Deliverables |
|---|---|---|
| **Day 13** | Design brief + decision matrix | Design brief + scored decision matrix 🎨 📓 Entry #7 |
| **Day 14** | Gantt chart + risk planning | Gantt chart (Google Sheets) shared with instructor 🎨 📓 Entry #8 |
| **Day 15** | System architecture + FOV/mounting design | Architecture diagram + camera placement calc 🎨 📓 Entry #9 |

***

## Day 13: Design Brief + Decision Matrix

### Learning Objectives
- Write a clear, testable design brief for your final project
- Compare multiple concept options objectively using a scored decision
  matrix

### Activity

As a team, revisit the individual concept sketches from Day 4 (Entry #2).
Narrow these down to **3 or more finalist concepts** — you can combine
ideas from both partners, or bring in a new idea inspired by what you
learned in Phase 2.

For each finalist concept, identify:
- The problem it solves and who the user is
- Which Phase 2 module(s) it depends on (color tracking, TFLite object
  detection, pose estimation, face recognition — or a combination)
- What "the system works" means in concrete, testable terms

### Write Your Design Brief

Save this to `design/design_brief.md` in your repo:

```markdown
# Design Brief — [Your Project Name]

## Problem Statement
[What real-world problem does this solve? Who is the user?]

## Proposed Solution
[One paragraph describing what your system does.]

## CV Technique(s) Used
[Which Phase 2 module(s) — color tracking, object detection, pose
estimation, face recognition — and why this technique fits the problem.]

## Success Criteria
[What does the system need to detect/count/recognize, and how reliably?
Be specific and measurable — e.g., "detect a red object within 2 meters
with at least 90% accuracy across 10 trials."]
```

### Build a Decision Matrix

Score each finalist concept (1–5) against shared criteria. Save to
`design/decision_matrix.md` or a spreadsheet:

| Concept | Feasibility on Pi Hardware | Time Available in Unit | Usefulness to Real User | Technical Interest | Total |
|---|---|---|---|---|---|
| Concept A | | | | | |
| Concept B | | | | | |
| Concept C | | | | | |

Write a short justification paragraph explaining why the highest-scoring
concept was chosen (or why you chose a different one, if the numbers
don't tell the whole story).

Push your work:
```bash
git add design/design_brief.md design/decision_matrix.md
git commit -m "Add design brief and scored decision matrix

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

### 📓 Notebook Entry #7 (Due today)
- Completed design brief
- Scored decision matrix with justification

***

## Day 14: Gantt Chart + Risk Planning

### Learning Objectives
- Identify project risks before they become problems
- Build a realistic schedule for the remaining build days (Phase 4)

### Risk Log

Identify **at least 5 risks** specific to your chosen concept. For each,
log a mitigation plan. Example format:

| Risk | Likelihood | Impact | Mitigation Plan |
|---|---|---|---|
| Camera mounting angle affects detection accuracy | Medium | High | Test at 3 candidate angles before final mount; document results |
| Lighting in demo room differs from lab testing | High | Medium | Test under classroom lighting specifically, not just lab conditions |
| Partner unavailable outside class for testing | Low | Medium | Schedule fixed testing windows during class time only |
| TFLite model doesn't recognize target object reliably | Medium | High | Have a fallback CV technique (e.g., color tracking) ready |
| SD card corruption / Pi failure before Demo Day | Low | High | Push to GitHub daily; keep a spare tested SD card |

### Build a Gantt Chart

Create a Gantt chart in Google Sheets mapping the remaining lesson days
(16–20) to project milestones: build sprints, testing, notebook entries,
and presentation prep. Share the sheet with your instructor with edit or
comment access.

Link it in your repo — save to `design/gantt_chart.md`:

```markdown
# Gantt Chart

[Link to shared Google Sheet]

## Summary
[2-3 sentences describing your overall timeline and any key milestones.]
```

Push your work:
```bash
git add design/gantt_chart.md
git commit -m "Add risk log and Gantt chart summary

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

### 📓 Notebook Entry #8 (Due today)
- Risk log (5+ risks with mitigation plans)
- Gantt chart summary

***

## Day 15: System Architecture + FOV/Mounting Design

### Learning Objectives
- Diagram the full data flow of your final system, end to end
- Apply the FOV formula from Day 2 to calculate real mounting distance
  and angle for your specific use case

### System Architecture Diagram

Diagram your system's full pipeline, labeling where each Phase 1–2
script's logic fits into the final design:

```
[Camera] → [Picamera2 capture] → [CV processing: e.g., pose_angles.py logic]
    → [Decision logic: e.g., rep counted / face matched / object detected]
    → [Output/action: e.g., on-screen display, CSV log, audio alert]
```

Save this to `design/system_architecture.md` — a simple text diagram like
the one above, or an image if you prefer to draw it, is fine.

### FOV and Mounting Distance Calculation

Using the FOV formula from Day 2:

\[
FOV = 2 \arctan\left(\frac{d}{2f}\right)
\]

Calculate the **mounting distance** needed so your specific target (a
person, an object, a doorway) is fully within frame. Show your work:

1. What is the physical size of the thing you need to detect?
2. What is your camera's FOV (from Day 2's calculation)?
3. Using basic trigonometry, what distance/angle puts your target fully
   in frame with margin?

Document this in `design/system_architecture.md` alongside your diagram.

Push your work:
```bash
git add design/system_architecture.md
git commit -m "Add system architecture diagram and FOV mounting calculation

Co-authored-by: Partner Full Name <partner@email.com>"
git push
```

### 📓 Notebook Entry #9 (Due today)
- System architecture diagram
- FOV/mounting distance calculation

***

## End of Phase 3 Checklist

- [ ] `design/design_brief.md` pushed
- [ ] `design/decision_matrix.md` pushed with justification
- [ ] `design/gantt_chart.md` pushed, Google Sheet shared with instructor
- [ ] `design/system_architecture.md` pushed with FOV/mounting calculation
- [ ] Notebook Entries #7–#9 complete for both partners

Your design is locked in. Phase 4 is where you build, code, and test the
real thing — starting with Build Sprint 1 on Day 16.
