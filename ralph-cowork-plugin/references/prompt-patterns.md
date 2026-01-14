# Prompt Patterns for Success Criteria

Templates and patterns for constructing effective autonomous loop prompts.

## Core Prompt Structure

Every loop prompt must include:

```markdown
## Task
[Clear description of what to accomplish]

## Working Folder
You have access to: [folder path]
[Optional: list of key files]

## Success Criteria
Complete ALL of the following:
- [ ] Criterion 1 (specific, measurable)
- [ ] Criterion 2 (specific, measurable)
- [ ] Criterion 3 (specific, measurable)

When ALL criteria are met, output exactly:
<complete>[COMPLETION_CODE]: [summary]</complete>

## If Stuck
If you cannot proceed due to:
- Missing information
- Corrupted files
- Ambiguous requirements
- Technical limitations

Output exactly:
<stuck>REASON: [specific description of blocker]</stuck>

## Progress Tracking
After each iteration, update progress.txt with:
- What you accomplished
- What remains
- Any issues encountered

## Iteration Context
Current iteration: [N] of [MAX]
[Previous progress summary if available]
```

## Signal Patterns

### Completion Signals

Use consistent, parseable format:

```
✅ Good: <complete>RECEIPTS_DONE: 47 processed, $3,241.50 total</complete>
✅ Good: <complete>REPORT_READY</complete>
✅ Good: <complete>ANALYSIS_COMPLETE: 5 insights generated</complete>

❌ Bad: "I'm done with the task"
❌ Bad: <complete>Done!</complete>  (too vague)
❌ Bad: The task is complete.  (not parseable)
```

### Stuck Signals

Include actionable information:

```
✅ Good: <stuck>CANNOT_READ: receipt_042.jpg is corrupted or not an image</stuck>
✅ Good: <stuck>MISSING_DATA: No date field found in invoice_7.pdf</stuck>
✅ Good: <stuck>AMBIGUOUS: Multiple possible interpretations for "organize by project"</stuck>

❌ Bad: <stuck>I'm stuck</stuck>  (no actionable info)
❌ Bad: <stuck>Error occurred</stuck>  (too vague)
```

### Progress Signals (Optional)

For long-running tasks, include progress updates:

```
<progress>ITERATION_3: Processed 12/47 receipts. Categories assigned. Starting expense calculations.</progress>
```

## Task-Specific Templates

### Document Processing Template

```markdown
## Task
Process all [document type] files in the working folder and create [output].

## Success Criteria
- [ ] All [N] [file type] files have been processed
- [ ] [Output file] exists with required structure:
  - [Column/Section 1]
  - [Column/Section 2]
  - [Column/Section 3]
- [ ] No files remain unprocessed
- [ ] [Validation check, e.g., totals calculated]

## Output Format
[Describe expected structure of output file]

## Handling Edge Cases
- If a file cannot be read: Mark as "MANUAL_REVIEW" in output
- If data is ambiguous: Use best interpretation, note uncertainty
- If duplicate found: [specific handling instruction]

## Completion Signal
<complete>DOCS_PROCESSED: [count] files, [summary stat]</complete>
```

### Content Creation Template

```markdown
## Task
Create [content type] based on source materials in the working folder.

## Source Materials
- [file1]: [description]
- [file2]: [description]

## Success Criteria
- [ ] [Output file] exists
- [ ] Content meets specifications:
  - Length: [word/page count]
  - Sections: [required sections]
  - Tone: [style guidance]
- [ ] Sources properly cited/referenced
- [ ] [Quality check, e.g., no placeholder text]

## Quality Standards
- [Specific quality requirement 1]
- [Specific quality requirement 2]

## Iteration Approach
1. First iteration: Create outline and structure
2. Subsequent iterations: Fill sections, refine
3. Final iterations: Polish and verify criteria

## Completion Signal
<complete>CONTENT_READY: [word count] words, [section count] sections</complete>
```

### Data Analysis Template

```markdown
## Task
Analyze [data source] and produce [deliverables].

## Input Data
- [data file]: [description, row/column count if known]
- [supporting file]: [description]

## Success Criteria
- [ ] Analysis complete for all data points
- [ ] [Output file] contains:
  - [Required analysis 1]
  - [Required analysis 2]
  - [Required visualization/chart]
- [ ] Insights documented: minimum [N] findings
- [ ] Recommendations provided: minimum [N] items

## Analysis Requirements
- Statistical methods: [if specific methods required]
- Comparison baseline: [if comparing to something]
- Segments to analyze: [if segmentation required]

## Output Structure
[Describe expected structure]

## Completion Signal
<complete>ANALYSIS_DONE: [key metric], [insight count] insights</complete>
```

### File Organization Template

```markdown
## Task
Organize files in working folder according to [organizational scheme].

## Current State
[Description of current folder state]

## Target Structure
```
folder/
├── [Category1]/
│   └── [subcategory if applicable]/
├── [Category2]/
├── [Category3]/
└── [special folders like duplicates/, review/]
```

## Success Criteria
- [ ] All files sorted into appropriate folders
- [ ] No files lost (verify count: [original count])
- [ ] [Special handling] folder contains [what]
- [ ] [Report file] documents all changes

## Sorting Rules
- [File type] → [destination]
- [Pattern] → [destination]
- Unknown/ambiguous → [handling]

## Safety Requirements
- Do NOT delete any files without moving to [review folder] first
- Create [backup/log] before major operations
- Preserve original filenames unless [condition]

## Completion Signal
<complete>FILES_ORGANIZED: [count] files sorted, [special count] flagged</complete>
```

## Wizard: Natural Language → Structured Criteria

Implement a wizard that converts user's natural language into structured criteria:

### Input Processing

```typescript
function parseUserIntent(naturalLanguage: string): ParsedIntent {
  // Identify task type
  const taskType = classifyTask(naturalLanguage);
  // Extract key entities (files, outputs, quantities)
  const entities = extractEntities(naturalLanguage);
  // Identify implicit requirements
  const implicit = inferRequirements(taskType, entities);
  
  return { taskType, entities, implicit };
}
```

### Example Transformations

**User input**: "organize my receipts into a spreadsheet"

**Wizard output**:
```markdown
## Task
Process receipt images and create expense spreadsheet.

## Detected Files
Found: 47 image files (.jpg, .png) in folder

## Success Criteria
- [ ] expenses.xlsx created
- [ ] All 47 images processed
- [ ] Columns: Date, Vendor, Amount, Category, Filename
- [ ] Total row with sum

## Suggested Settings
- Max iterations: 15
- Stuck threshold: 3

Does this look right? [Edit] [Start]
```

---

**User input**: "summarize these meeting notes"

**Wizard output**:
```markdown
## Task
Consolidate meeting notes into summary document.

## Detected Files
Found: 8 text files (.txt, .md) in folder

## Success Criteria
- [ ] summary.docx created
- [ ] All 8 meetings summarized
- [ ] Action items extracted into table
- [ ] Key decisions highlighted

## Suggested Settings
- Max iterations: 8
- Stuck threshold: 3

Does this look right? [Edit] [Start]
```

## Anti-Patterns to Avoid

### Vague Criteria

```
❌ "Make a good presentation"
❌ "Clean up the folder"
❌ "Analyze the data thoroughly"

✅ "Create 10-slide presentation with executive summary"
✅ "Sort files into Documents/, Images/, Archives/ folders"
✅ "Identify top 5 trends with supporting data points"
```

### Unmeasurable Success

```
❌ "Make it professional"
❌ "Ensure quality"
❌ "Do a good job"

✅ "No spelling errors, consistent formatting"
✅ "All required sections present, word count 1000-1500"
✅ "All data points accounted for, no nulls in required fields"
```

### Open-Ended Tasks

```
❌ "Keep improving until perfect"
❌ "Do as much as you can"
❌ "Make it better"

✅ "Complete these 5 specific criteria"
✅ "Process all 47 files"
✅ "Create document with these 4 sections"
```

## Prompt Testing Checklist

Before starting a loop, verify the prompt:

- [ ] Task is clearly defined
- [ ] Success criteria are specific and measurable
- [ ] Completion signal format is specified
- [ ] Stuck signal format is specified
- [ ] Edge cases have handling instructions
- [ ] Progress tracking method is defined
- [ ] Max iterations is reasonable for task complexity
