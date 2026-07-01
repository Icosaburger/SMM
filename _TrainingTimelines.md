# Administrator Documentation: Training Timeline Matrix Engine

This matrix is powered by a custom JavaScript rendering engine that converts flat data arrays into a dynamic, dependency-mapped Gantt chart. 

By structuring data inside the script's `sectionsData` array, the system automatically handles timeline positioning, dependency scheduling, global alignment, and accordion animations without requiring manual HTML or CSS layout modifications.

---

## 1. The Core Data Blueprint

All modifications take place within the `sectionsData` array found inside the `<script>` tags. The structure uses standard JSON formatting divided into **Sections** and **Items**.

### Code Template
[INSERT_START_CODE]
const sectionsData = [
    {
        sectionTitle: "NAME OF THE DISCIPLINE SECTION",
        items: [
            { 
                technique: "Exact Name of Technique", 
                prereq: "None", 
                wait: 0, 
                cat2: 0, 
                results: 0 
            }
        ]
    }
];
[INSERT_END_CODE]

### Parameter Reference Table

| Key Parameter | Expected Value | Behavior & Engine Logic |
| :--- | :--- | :--- |
| sectionTitle | String (e.g., "XRM SECTION") | Creates a sticky, full-width accordion header block. |
| technique | String (e.g., "SEM Imaging") | Displays as the left-hand text label. Must be unique if used as a prerequisite elsewhere. |
| prereq | String ("None", or explicit milestone string) | Defines the scheduling trigger point for this specific line block. |
| wait | Integer (Weeks) | Renders a Light Pink Bar. Represents processing/onboarding latency. |
| cat2 | Integer (Weeks) | Renders a Blue Bar. Represents core technical instruction tracks. |
| results | Integer (Weeks) | Renders a Green Bar. Represents independent user capability testing. |

---

## 2. Setting Up Prerequisites & Dependencies

The calculation engine supports dynamic cascading timelines. If a technique depends on another, it will automatically shift downstream to start exactly when its prerequisite milestone is met.

The engine looks for two specific string formats in the `prereq` value:

### A. The "Category 2" Milestone Trigger
If a technique can begin as soon as the user finishes core training on a parent tool, format the prerequisite like this:
> prereq: "Category 2 in [Parent Technique Name]"

* Example Case: Data Reconstruction requires basic physical X-Ray training to finish first.
[INSERT_START_CODE]
{ technique: "X-Ray tomography", prereq: "None", wait: 2, cat2: 2, results: 4 },
{ technique: "Data Reconstruction", prereq: "Category 2 in X-Ray tomography", wait: 0, cat2: 1, results: 8 }
[INSERT_END_CODE]

* Engine Result: X-Ray tomography starts at Week 0. Its wait (2) + cat2 (2) ends at Week 4. Data Reconstruction detects this milestone and sets its starting line precisely at Week 4.

### B. The "Meaningful Results" Milestone Trigger
If a technique requires the user to fully master a parent tool and gain independent experience before moving to advanced configurations, format the prerequisite like this:
> prereq: "Meaningful Results in [Parent Technique Name]"

* Example Case: FIB-SEM Basic requires completely wrapping up all mastery phases of basic SEM imaging.
[INSERT_START_CODE]
{ technique: "SEM Imaging", prereq: "None", wait: 2, cat2: 1, results: 4 },
{ technique: "FIB-SEM Basic", prereq: "Meaningful Results in SEM Imaging", wait: 3, cat2: 3, results: 8 }
[INSERT_END_CODE]

* Engine Result: SEM Imaging ends all phases at Week 7 (2 + 1 + 4). FIB-SEM Basic identifies this full completion milestone and schedules its initial block to start at Week 7.

> Crucial Rule for Admins: The text inside your prerequisite string following "Category 2 in " or "Meaningful Results in " must exactly match the technique name of the parent block, including spacing and capitalization. If they do not match perfectly, the engine will default the start position back to Week 0.

---

## 3. Handling Blank Entries or "No Bar" Scenarios

If a specific technique does not require certain phases (e.g., it has no induction waiting period or does not have an independent "results" tracking block), do not leave fields blank or drop the keys. Instead, pass a value of 0 or set the prerequisite string directly to "None". 

* Correct entry for a tool with no wait time and no results tracking:
[INSERT_START_CODE]
{ technique: "Formvar Grid Casting", prereq: "None", wait: 0, cat2: 2, results: 0 }
[INSERT_END_CODE]

* Correct entry for simple access milestones with zero training hours allocated:
[INSERT_START_CODE]
{ technique: "Diamond Saw", prereq: "None", wait: 0, cat2: 0, results: 0 }
[INSERT_END_CODE]

The calculation block handles 0 entries smoothly by suppressing block rendering layout generation while maintaining structural grid integrity.

---

## 4. Troubleshooting for Future Admins

* The timeline bar is starting at Week 0 instead of shifting downstream: Check the spelling of your prereq parameter. Ensure it accurately incorporates either "Category 2 in ..." or "Meaningful Results in ..." followed by a verbatim character match of the parent technique block label.
* A new accordion section is completely frozen or won't click open: This typically means a comma or structural bracket was accidentally omitted when pasting data into sectionsData. Open your web browser's Developer Tools Console (F12) to identify the exact code line syntax error.
* The weeks on the right side are cut off: You do not need to expand the column sizes manually. The code scans every element dynamically on load, finds the single longest total timeframe cumulative pathway, and automatically generates the perfect width and column count across all panels.