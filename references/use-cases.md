# Use Cases Library

Real-world scenarios for testing and demonstrating Ralph-Cowork. Each includes task description, success criteria, expected iterations, and edge cases.

## Category: Document Processing

### Use Case 1: Receipt Organization

**User persona**: Small business owner, non-technical

**Task**: "Organize all my receipt photos into a spreadsheet for tax purposes"

**Input folder contents**:
- 47 receipt images (.jpg, .png)
- Mixed naming conventions (IMG_001.jpg, receipt_starbucks.png, etc.)

**Success criteria**:
```
- expenses.xlsx exists
- Contains columns: Date, Vendor, Amount, Category, Original_Filename
- All 47 images processed (row count = 47)
- Total row at bottom with SUM formula
- Categories assigned (Food, Travel, Office, etc.)
```

**Completion signal**: `<complete>ALL_RECEIPTS_PROCESSED: 47 items</complete>`

**Expected iterations**: 8-15 (depends on OCR accuracy)

**Edge cases to test**:
- Blurry receipt image → should mark as "MANUAL_REVIEW_NEEDED"
- Foreign currency receipt → should note currency conversion needed
- Duplicate receipt → should flag duplicates

---

### Use Case 2: Meeting Notes Consolidation

**User persona**: Project manager

**Task**: "Combine all meeting notes from this month into a single summary document with action items"

**Input folder contents**:
- 12 .txt files with meeting notes
- Inconsistent formatting
- Action items scattered throughout

**Success criteria**:
```
- monthly_summary.docx exists
- Contains section for each meeting (date + attendees + summary)
- Action items extracted into separate table
- Action items have: Owner, Due Date, Status
- Table of contents at top
```

**Completion signal**: `<complete>SUMMARY_COMPLETE: 12 meetings, 34 action items</complete>`

**Expected iterations**: 5-8

**Edge cases**:
- Meeting note with no clear action items
- Duplicate action items across meetings
- Missing date in note filename

---

### Use Case 3: Invoice Batch Processing

**User persona**: Accounts payable clerk

**Task**: "Extract data from all invoices and create a payment schedule"

**Input folder contents**:
- 23 PDF invoices from various vendors
- Different invoice formats

**Success criteria**:
```
- payment_schedule.xlsx exists
- Contains: Vendor, Invoice#, Amount, Due Date, Payment Priority
- Sorted by due date
- Invoices due within 7 days marked HIGH priority
- Total payables calculated
```

**Completion signal**: `<complete>INVOICES_PROCESSED: 23 invoices, $47,832.50 total</complete>`

**Expected iterations**: 10-20 (PDF parsing is harder)

---

## Category: Content Creation

### Use Case 4: Blog Post from Research

**User persona**: Marketing manager

**Task**: "Turn these research notes into a polished blog post about AI trends"

**Input folder contents**:
- research_notes.txt (rough bullet points)
- competitor_analysis.txt
- 3 source article PDFs

**Success criteria**:
```
- blog_post.md exists
- Word count: 1200-1800 words
- Has: Title, Introduction, 3-5 sections, Conclusion, CTA
- Cites at least 3 sources
- Includes suggested meta description
- draft_versions/ folder with iteration history
```

**Completion signal**: `<complete>BLOG_READY: 1,547 words, 4 sources cited</complete>`

**Expected iterations**: 4-7

---

### Use Case 5: Presentation from Document

**User persona**: Sales executive

**Task**: "Create a 10-slide presentation from this proposal document"

**Input folder contents**:
- proposal.docx (15 pages)
- company_logo.png
- brand_guidelines.pdf

**Success criteria**:
```
- presentation.pptx exists
- Exactly 10 slides
- Includes: Title slide, Agenda, 7 content slides, Thank you/Contact
- Logo on each slide
- Follows brand color palette
- Speaker notes on each slide
```

**Completion signal**: `<complete>PRESENTATION_READY: 10 slides with notes</complete>`

**Expected iterations**: 6-10

---

## Category: Data Analysis

### Use Case 6: Survey Results Analysis

**User persona**: HR director

**Task**: "Analyze employee survey results and create an executive summary"

**Input folder contents**:
- survey_responses.csv (500 rows)
- previous_survey_2023.csv (for comparison)
- questions.txt (survey questions reference)

**Success criteria**:
```
- executive_summary.docx exists
- analysis.xlsx with pivot tables
- Charts: Overall satisfaction trend, Department breakdown, Top concerns
- Key findings: At least 5 insights
- Recommendations: At least 3 actionable items
- YoY comparison included
```

**Completion signal**: `<complete>ANALYSIS_COMPLETE: 5 insights, 3 recommendations</complete>`

**Expected iterations**: 8-12

---

### Use Case 7: Competitive Pricing Analysis

**User persona**: Product manager

**Task**: "Analyze competitor pricing data and recommend our positioning"

**Input folder contents**:
- competitor_prices.xlsx (our collected data)
- our_current_pricing.xlsx
- market_segments.txt

**Success criteria**:
```
- pricing_analysis.docx with recommendations
- comparison_matrix.xlsx
- Charts: Price positioning map, Feature/price scatter
- Recommendations by segment
- Identified gaps and opportunities
```

**Completion signal**: `<complete>PRICING_ANALYSIS_DONE: 3 segments analyzed</complete>`

**Expected iterations**: 5-8

---

## Category: File Organization

### Use Case 8: Photo Library Cleanup

**User persona**: Individual user

**Task**: "Organize my photos by year and month, remove duplicates"

**Input folder contents**:
- 2,847 photos in flat structure
- Mix of .jpg, .png, .heic
- Many duplicates

**Success criteria**:
```
- Folder structure: YYYY/MM/
- All photos sorted by EXIF date (or filename date if no EXIF)
- duplicates/ folder with identified duplicates
- duplicates_report.txt listing all duplicates found
- No files lost (count verification)
```

**Completion signal**: `<complete>PHOTOS_ORGANIZED: 2847 files, 234 duplicates found</complete>`

**Expected iterations**: 15-25 (large file count)

**Special consideration**: This is a high-iteration task. Test cost controls.

---

### Use Case 9: Downloads Folder Triage

**User persona**: Anyone with a messy downloads folder

**Task**: "Sort my downloads folder - group by type, delete obvious junk, flag important items"

**Input folder contents**:
- 847 files of various types
- Many installer .dmg/.exe files
- PDFs, images, documents mixed

**Success criteria**:
```
- Subfolders: Documents/, Images/, Installers/, Archives/, Other/
- to_delete/ folder with obvious junk (old installers, duplicate downloads)
- important/ folder with recent documents
- cleanup_report.txt with actions taken
- Nothing deleted without user confirmation
```

**Completion signal**: `<complete>DOWNLOADS_SORTED: 847 files organized, 156 flagged for deletion</complete>`

**Expected iterations**: 10-15

---

## Category: Research & Synthesis

### Use Case 10: Literature Review

**User persona**: Consultant, researcher

**Task**: "Review these research papers and create a synthesis document"

**Input folder contents**:
- 8 PDF research papers
- research_question.txt (the question to answer)

**Success criteria**:
```
- literature_review.docx exists
- Summary of each paper (1 paragraph each)
- Synthesis section answering research question
- Comparison table of methodologies
- Bibliography in APA format
- Key quotes extracted with citations
```

**Completion signal**: `<complete>REVIEW_COMPLETE: 8 papers synthesized</complete>`

**Expected iterations**: 10-15

---

## Testing Matrix

| Use Case | Complexity | Expected Iterations | Tests |
|----------|------------|--------------------| ------|
| Receipt Organization | Medium | 8-15 | OCR, categorization |
| Meeting Notes | Low | 5-8 | Text extraction, summarization |
| Invoice Processing | High | 10-20 | PDF parsing, date handling |
| Blog Post | Medium | 4-7 | Content creation, iteration |
| Presentation | Medium | 6-10 | Multi-file output, formatting |
| Survey Analysis | High | 8-12 | Data analysis, visualization |
| Pricing Analysis | Medium | 5-8 | Comparative analysis |
| Photo Cleanup | High | 15-25 | Large file count, duplicates |
| Downloads Triage | Medium | 10-15 | File type detection |
| Literature Review | High | 10-15 | PDF comprehension, synthesis |

## Stuck Scenario Tests

Test these scenarios to ensure proper stuck detection:

1. **Corrupted file**: Include one corrupted PDF → should output `<stuck>CANNOT_READ: filename.pdf</stuck>`

2. **Missing dependencies**: Request chart creation without data → should report what's missing

3. **Ambiguous instructions**: Vague task without clear criteria → should ask for clarification

4. **Impossible task**: "Delete files from server" when only local access → should recognize scope limitation

5. **Infinite loop potential**: Task that can never complete (e.g., "make it perfect") → should hit iteration limit gracefully
