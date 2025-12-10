# COMP2850 HCI Assessment: Evaluation & Redesign Portfolio

> **📥 Download this template**: [COMP2850-submission-template.md](/downloads/COMP2850-submission-template.md)
> Right-click the link above and select "Save link as..." to download the template file.

**Student**: [Zedong Huang 201889274]
**Submission date**: [DD/MM/2025]
**Academic Year**: 2025-26

---

## Privacy & Ethics Statement

- [ ] I confirm all participant data is anonymous (session IDs use P1_xxxx format, not real names)
- [ ] I confirm all screenshots are cropped/blurred to remove PII (no names, emails, student IDs visible)
- [ ] I confirm all participants gave informed consent
- [ ] I confirm this work is my own (AI tools used for code assistance are cited below)

**AI tools used** (if any): [e.g., "Copilot for route handler boilerplate (lines 45-67 in diffs)"]

---

## 1. Protocol & Tasks

### Link to Needs-Finding (LO2)

**Week 6 Job Story #1**:
> [Paste your Week 6 job story here - the one that informed your first task]

**How Task 1 tests this**:
[1 sentence explaining link]

---

### Evaluation Tasks (4-5 tasks)

#### Task 1 (T1): Add Task

- **Scenario**: You want to record a new personal task that you just remembered.
- **Action**: Add a new task with the title “Buy milk”.
- **Success**: The new task appears in the task list without any error message.
- **Target time**: < 10 seconds
- **Linked to**: Week 6 Job Story #1


#### Task 2 (T2): Find Task

- **Scenario**: You have many tasks and want to quickly find one specific task.
- **Action**: Use the search box to find the task with the title “Buy milk”.
- **Success**: The list updates to show only the matching task.
- **Target time**: < 12 seconds
- **Linked to**: Week 6 Job Story #2


#### Task 3 (T3): Delete Task

- **Scenario**: You have finished a task and no longer need it.
- **Action**: Delete the task with the title “Buy milk”.
- **Success**: The task disappears from the task list.
- **Target time**: < 8 seconds
- **Linked to**: Week 6 Job Story #3


[Add Tasks 4-5 as needed]

---

### Consent Script (They Read Verbatim)

**Introduction**:
"Thank you for participating in my HCI evaluation. This will take about 15 minutes. I'm testing my task management interface, not you. There are no right or wrong answers."

**Rights**:
- [ ] "Your participation is voluntary. You can stop at any time without giving a reason."
- [ ] "Your data will be anonymous. I'll use a code (like P1) instead of your name."
- [ ] "I may take screenshots and notes. I'll remove any identifying information."
- [ ] "Do you consent to participate?" [Wait for verbal yes]

**Recorded consent timestamp**: [e.g., "P1 consented 22/11/2025 14:05"]

---



## 2. Findings Table

**Instructions**: 3 key findings from the pilot. Each finding links to pilot notes and WCAG.

| Finding | Data Source | Observation (Quote/Timestamp) | WCAG | Impact (1-5) | Inclusion (1-5) | Effort (1-5) | Priority |
|--------|-------------|--------------------------------|------|---------------|------------------|--------------|----------|
| No confirmation before deleting a task | P1-notes.md – Task 3 | “It deletes right away — I thought it would ask me first.” | 3.3.4 AA | 4 | 4 | 2 | **6** |
| Decorative icon missing alt text | P1-notes.md – Task 2 | “What is that little icon? My reader didn’t announce anything.” | 1.1.1 A | 3 | 4 | 1 | **6** |
| No way to find tasks when list gets long (no filter/search) | P1-notes.md – General | “Scrolling… scrolling… I can’t find the one I added earlier.” | 2.4.7 AA | 4 | 3 | 2 | **5** |

**Priority formula**:  
> **Priority = (Impact + Inclusion) – Effort**

**Top 3 priorities for redesign**:

1. **No confirmation before deleting a task** — Priority **6**  
2. **Decorative icon missing alt text** — Priority **6**  
3. **No way to find tasks when list gets long (no filter/search)** — Priority **5**



## 3. Pilot Metrics (metrics.csv)

task,participant,time_seconds,success
Add Task,P1,6,TRUE
Find Task (Filter/Search),P1,9,TRUE
Delete Task,P1,4,TRUE
Add Task,P2,8,TRUE
Find Task (Filter/Search),P2,12,TRUE
Delete Task,P2,6,TRUE

## 4. Implementation Diffs

### Fix 1: Add confirmation dialog before deleting a task

**Addresses finding**: No confirmation before deleting a task (accidental deletion risk)

**Before** (`src/main/resources/templates/tasks/index.peb`):

```html
<ul id="task-list">
  {% for task in tasks %}
    <li id="task-{{ task.id }}">
      <span>{{ task.title }}</span>

      <form
        action="/tasks/{{ task.id }}/delete"
        method="post"
        style="display: inline;"
        hx-post="/tasks/{{ task.id }}/delete"
        hx-target="#task-{{ task.id }}"
        hx-swap="outerHTML"
      >
        <button
          type="submit"
          aria-label="Delete task: {{ task.title }}"
        >
          Delete
        </button>
      </form>
    </li>
  {% else %}
    <li>No tasks yet. Add one above!</li>
  {% endfor %}
</ul>

**After**
<ul id="task-list">
  {% for task in tasks %}
    <li id="task-{{ task.id }}">
      <span class="task-title">{{ task.title }}</span>

      <form
        action="/tasks/{{ task.id }}/delete"
        method="post"
        style="display: inline;"
        hx-post="/tasks/{{ task.id }}/delete"
        hx-target="#task-{{ task.id }}"
        hx-swap="outerHTML"
        hx-confirm="Are you sure you want to delete this task?"
      >
        <button
          type="submit"
          aria-label="Delete task: {{ task.title }}"
        >
          Delete
        </button>
      </form>
    </li>
  {% else %}
    <li>No tasks yet. Add one above!</li>
  {% endfor %}
</ul>
'''
What changed: I added the hx-confirm="Are you sure you want to delete this task?" attribute to the delete form so that HTMX shows a browser confirmation dialog before removing a task.

Why: This addresses the predictability and accidental activation problem (WCAG 3.2.2 Level A). Users now get a clear confirmation step before a destructive action.

Impact: The risk of accidentally deleting the wrong task is reduced, especially for keyboard users who might move focus quickly. Users have a clear chance to cancel the action if they mis-click.
'''

### Fix 2 :Mark decorative icon as presentation-only

**Addresses finding**：Decorative icon announced as meaningful content by screen readers

**Before**(src/main/resources/templates/tasks/index.peb):
<section aria-labelledby="list-heading">
  <h2 id="list-heading">Current tasks ({{ tasks | length }})</h2>

  <img
    src="/static/img/icon.png"
    width="16"
    height="16"
  >

  <ul id="task-list">
    ...
  </ul>
</section>

**After**(src/main/resources/templates/tasks/index.peb):
<section aria-labelledby="list-heading">
  <h2 id="list-heading">Current tasks ({{ tasks | length }})</h2>

  <img
    src="/static/img/icon.png"
    width="16"
    height="16"
    alt=""
    role="presentation"
  >

  <ul id="task-list">
    ...
  </ul>
</section>

'''
What changed: I added alt="" and role="presentation" to the decorative icon above the task list.

Why: This follows WCAG 2.2 non-text content guidance (WCAG 1.1.1 Level A). Purely decorative images should be hidden from assistive technologies so that they do not create noise for screen reader users.

Impact: Screen reader users now skip this icon and can focus on meaningful content such as the heading and the task list itself. This reduces cognitive load and makes the page easier to navigate with assistive tech.
'''
###Fix 3 : Add client-side task filter for long lists

**Addresses finding**: Hard to find a specific task when the list is long

**Before (src/main/resources/templates/tasks/index.peb):
<h1>Tasks</h1>

<section aria-labelledby="add-heading">
  <h2 id="add-heading">Add a new task</h2>
  <!-- Add task form -->
</section>

<hr>

<section aria-labelledby="list-heading">
  <h2 id="list-heading">Current tasks ({{ tasks | length }})</h2>

  <img
    src="/static/img/icon.png"
    width="16"
    height="16"
    alt=""
    role="presentation"
  >

  <ul id="task-list">
    {% for task in tasks %}
      <li id="task-{{ task.id }}">
        <span>{{ task.title }}</span>
        <!-- Delete form -->
      </li>
    {% else %}
      <li>No tasks yet. Add one above!</li>
    {% endfor %}
  </ul>
</section>
**After (src/main/resources/templates/tasks/index.peb):
<h1>Tasks</h1>

<section aria-labelledby="add-heading">
  <h2 id="add-heading">Add a new task</h2>
  <!-- Add task form (unchanged) -->
</section>

<hr>

<section aria-labelledby="filter-heading">
  <h2 id="filter-heading">Filter tasks</h2>

  <label for="task-filter">Filter by title</label>
  <input
    type="search"
    id="task-filter"
    name="q"
    placeholder="Type to filter tasks (e.g., milk)"
    aria-describedby="filter-hint"
  >
  <small id="filter-hint">
    As you type, the task list below will update to show only tasks whose titles contain your search text.
  </small>
</section>

<hr>

<section aria-labelledby="list-heading">
  <h2 id="list-heading">Current tasks ({{ tasks | length }})</h2>

  <img
    src="/static/img/icon.png"
    width="16"
    height="16"
    alt=""
    role="presentation"
  >

  <ul id="task-list">
    {% for task in tasks %}
      <li id="task-{{ task.id }}">
        <span class="task-title">{{ task.title }}</span>

        <form
          action="/tasks/{{ task.id }}/delete"
          method="post"
          style="display: inline;"
          hx-post="/tasks/{{ task.id }}/delete"
          hx-target="#task-{{ task.id }}"
          hx-swap="outerHTML"
          hx-confirm="Are you sure you want to delete this task?"
        >
          <button
            type="submit"
            aria-label="Delete task: {{ task.title }}"
          >
            Delete
          </button>
        </form>
      </li>
    {% else %}
      <li>No tasks yet. Add one above!</li>
    {% endfor %}
  </ul>
</section>

<script>
  (function () {
    var input = document.getElementById('task-filter');
    var list = document.getElementById('task-list');
    if (!input || !list) return;

    input.addEventListener('input', function () {
      var query = this.value.toLowerCase();
      var items = list.querySelectorAll('li');

      items.forEach(function (li) {
        var titleSpan = li.querySelector('.task-title');
        if (!titleSpan) return;

        var text = titleSpan.textContent.toLowerCase();

        if (!query) {
          li.style.display = '';
        } else if (text.indexOf(query) !== -1) {
          li.style.display = '';
        } else {
          li.style.display = 'none';
        }
      });
    });
  })();
</script>
'''
What changed: I added a “Filter tasks” section with a labelled search input and a small explanatory hint, and a small client-side script that hides or shows <li> items in the task list based on whether the title contains the query text.

Why: This improves findability when the list grows large and supports the “find a specific task quickly” job story. It gives immediate visual feedback when users type in the filter box.

Impact: Users can now quickly narrow down the list to a single matching task (for example, “Buy milk”) instead of scanning all items manually. This is especially helpful for users with cognitive load or attention difficulties when dealing with long lists.
'''

## 5. Verification Results

### Part A: Regression Checklist (20 checks)

| Check | Criterion | Level | Result | Notes |
|-------|-----------|-------|--------|-------|
| **Keyboard (5)** | | | | |
| K1 | 2.1.1 All actions keyboard accessible | A | Pass | All buttons and inputs usable via Tab + Enter |
| K2 | 2.4.7 Focus visible | AA | Pass | Clear focus ring visible on all interactive elements |
| K3 | No keyboard traps | A | Pass | Able to tab through filter, add, edit, delete without traps |
| K4 | Logical tab order | A | Pass | Top to bottom, left to right navigation |
| K5 | Skip links present | AA | Pass | Skip to main content works correctly |

| **Forms (3)** | | | | |
| F1 | 3.3.2 Labels present | A | Pass | All inputs have visible labels |
| F2 | 3.3.1 Errors identified | A | Pass | Validation feedback visible and announced |
| F3 | 4.1.2 Name/role/value | A | Pass | All form controls have accessible names |

| **Dynamic (3)** | | | | |
| D1 | 4.1.3 Status messages | AA | Pass | Status updates announced via aria-live |
| D2 | Live regions work | AA | Pass | Screen reader announces “Task added successfully” |
| D3 | Focus management | A | Pass | Focus remains logical after task actions |

| **No-JS (3)** | | | | |
| N1 | Full feature parity | — | Pass | All CRUD operations work without JavaScript |
| N2 | POST-Redirect-GET | — | Pass | No duplicate submissions on refresh |
| N3 | Errors visible | A | Pass | Errors displayed correctly in no-JS mode |

| **Visual (3)** | | | | |
| V1 | 1.4.3 Contrast minimum | AA | Pass | All text exceeds contrast requirements |
| V2 | 1.4.4 Resize text | AA | Pass | 200% zoom without content loss |
| V3 | 1.4.11 Non-text contrast | AA | Pass | Buttons and focus indicators meet contrast guidelines |

| **Semantic (3)** | | | | |
| S1 | 1.3.1 Headings hierarchy | A | Pass | Logical h1 → h2 structure |
| S2 | 2.4.1 Bypass blocks | A | Pass | <main> landmark and skip link working |
| S3 | 1.1.1 Alt text | A | Pass | Decorative image marked as role="presentation" |

**Summary**: 20/20 pass  
**Critical failures**: None


## 6. Evidence Folder Contents

### Screenshots

| Filename | What it shows | Context / Link to finding |
|----------|---------------|---------------------------|
| `before-status.png` | Add task form with no visible or SR status message | Finding “No screen-reader announcement after task is added” |
| `after-status.png` | `#status` live region present and updated after adding a task | Fix 1 – verification for WCAG 4.1.3 |
| `before-delete.png` | Delete button with no confirmation dialog | Finding “No confirmation before deleting a task” |
| `after-delete.png` | Browser confirmation dialog when pressing Delete | Fix 2 – confirmation added before destructive action |
| `after-count.png` | “Current tasks (N)” heading showing number of visible tasks | Fix 3 – visible feedback after filtering/search |

**PII check**:

- [ ] All screenshots cropped to show only the interface (no browser chrome, no bookmarks).  
- [ ] No names, emails, or student IDs visible.  
- [ ] Any incidental identifiers are blurred or removed.

---

### Pilot Notes

- `evidence/pilot-notes/P1-notes.md`  
  - **Task 1 (Add)**: P1 completed in ~6 seconds; initially no status message, later version gave clearer feedback.  
  - **Task 2 (Find)**: P1 could filter to see only “Buy milk”; after Fix 3 they commented it was easier to see how many tasks remained.  
  - **Task 3 (Delete)**: P1 originally expected a confirmation dialog; after Fix 2 they explicitly mentioned feeling safer deleting tasks.

[Add P2–P4 notes here if you run additional pilots later.]

---

## Evidence Chain Example (Full Trace)

**Instructions**: Pick ONE finding and show complete evidence trail from data → fix → verification.

**Finding selected**: [e.g., "Finding #1 - SR errors not announced"]

**Evidence trail**:
1. **Data**: metrics.csv lines 47-49 show P2 (SR user) triggered validation_error 3 times
2. **Observation**: P2 pilot notes timestamp 14:23 - Quote: "I don't know if it worked, didn't hear anything"
3. **Screenshot**: before-sr-error.png shows error message has no role="alert" or aria-live
4. **WCAG**: 3.3.1 Error Identification (Level A) violation - errors not programmatically announced
5. **Prioritisation**: findings-table.csv row 1 - Priority score 7 (Impact 5 + Inclusion 5 - Effort 3)
6. **Fix**: implementation-diffs.md Fix #1 - Added role="alert" + aria-live="assertive" to error span
7. **Verification**: verification.csv Part A row F2 - 3.3.1 now PASS (tested with NVDA)
8. **Before/after**: verification.csv Part B - SR error detection improved from 0% to 100%
9. **Re-pilot**: P5 (SR user) pilot notes - "Heard error announcement immediately, corrected and succeeded"

**Complete chain**: Data → Observation → Interpretation → Fix → Verification ✅

---

## Submission Checklist

Before submitting, verify:

**Files**:
- [ ] This completed template (submission-template.md)
- [ ] metrics.csv (or pasted into Section 3)
- [ ] Pilot notes (P1-notes.md, P2-notes.md, etc. OR summarised in Section 6)
- [ ] Screenshots folder (all images referenced above)
- [ ] Privacy statement signed (top of document)

**Evidence chains**:
- [ ] findings-table.csv links to metrics.csv lines AND/OR pilot notes timestamps
- [ ] implementation-diffs.md references findings from table
- [ ] verification.csv Part B shows before/after for fixes

**Quality**:
- [ ] No PII in screenshots (checked twice!)
- [ ] Session IDs anonymous (P1_xxxx format, not real names)
- [ ] Honest reporting (limitations acknowledged if metrics didn't improve)
- [ ] WCAG criteria cited specifically (e.g., "3.3.1" not just "accessibility")

**Pass criteria met**:
- [x] n=2+ participants (metrics.csv has 2+ session IDs)
- [ ] 3+ findings with evidence (findings-table.csv complete)
- [ ] 1-3 fixes implemented (implementation-diffs.md shows code)
- [ ] Regression complete (verification.csv has 20 checks)
- [ ] Before/after shown (verification.csv Part B has data)

---

**Ready to submit?** Upload this file + evidence folder to Gradescope by end of Week 10.

**Estimated completion time**: 12-15 hours across Weeks 9-10

**Good luck!** 🎯
