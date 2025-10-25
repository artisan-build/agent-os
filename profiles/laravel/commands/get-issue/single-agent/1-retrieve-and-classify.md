---
description: Retrieve and classify GitHub issue
step: 1
total_steps: 2
version: 1.0
encoding: UTF-8
---

# Step 1: Retrieve and Classify GitHub Issue

<ai_meta>
  <parsing_rules>
    - Process XML blocks first for structured data
    - Execute instructions in sequential order
    - Use GitHub CLI for all issue operations
  </parsing_rules>
  <step_context>
    - current_step: 1
    - total_steps: 2
    - next_step: 2-process-issue.md
  </step_context>
</ai_meta>

## Step Overview

<purpose>
  - Retrieve GitHub issue using provided argument or interactive selection
  - Analyze issue content and classify type
  - Prepare for processing workflow
</purpose>

<process_flow>

<substep number="1.1" name="retrieve_issue">

### Substep 1.1: Retrieve Issue

<retrieval_instructions>
  **Check if issue argument was provided:**

  IF user provided a number:
    1. Run: `gh issue view [NUMBER] --json number,title,body,author,url`
    2. IF issue found: store issue details
    3. IF issue not found: proceed to interactive selection

  IF user provided a search string:
    1. Run: `gh issue list --search "[STRING]" --json number,title,body,author,url --limit 10`
    2. IF matches found: show list and ask user to select
    3. IF no matches: proceed to interactive selection

  IF no argument provided OR no matches found:
    1. Run: `gh issue list --json number,title,body,author,url --limit 20`
    2. Display list to user with numbers and titles
    3. Ask user: "Which issue would you like to work on? (Enter issue number)"
    4. Wait for user selection
    5. Retrieve full details: `gh issue view [SELECTED_NUMBER] --json number,title,body,author,url`
</retrieval_instructions>

<issue_data_structure>
  Store the following information:
  - issue_number: numeric ID
  - issue_title: string
  - issue_body: full text content
  - issue_author: username
  - issue_url: GitHub URL
</issue_data_structure>

</substep>

<substep number="1.2" name="classify_issue">

### Substep 1.2: Classify Issue Type

<classification_instructions>
  **Analyze the issue title and body to determine type:**

  **BUG Classification:**
  - Look for: "bug", "error", "broken", "fails", "doesn't work", "not working"
  - Look for: stack traces, error messages, unexpected behavior descriptions
  - Look for: "steps to reproduce", "expected behavior", "actual behavior"
  - Pattern: describes something that should work but doesn't

  **FEATURE REQUEST Classification:**
  - Look for: "feature", "enhancement", "would be nice", "should add", "request"
  - Look for: user stories (as a... I want... so that...)
  - Look for: "it would be great if", "we should have", "can we add"
  - Pattern: proposes new functionality or improvements

  **PATCH Classification:**
  - Look for: code blocks with changes
  - Look for: file paths and line numbers to modify
  - Look for: "here's a fix", "patch", "this should work"
  - Pattern: includes solution code in issue body

  **QUESTION Classification:**
  - Look for: question marks, "how do I", "why does", "what is"
  - Look for: "help", "documentation", "unclear", "confused"
  - Pattern: asks for clarification or guidance
  - Default: if none of the above clearly apply

  **Make Classification Decision:**
  1. Read full issue title and body
  2. Identify strongest indicators
  3. Choose ONE classification
  4. Store: issue_type = [bug|feature|patch|question]
</classification_instructions>

</substep>

<substep number="1.3" name="display_classification">

### Substep 1.3: Display Classification

<display_template>
  Display to user:

  ```
  Issue #{issue_number}: {issue_title}
  Author: {issue_author}
  Classification: {issue_type}

  {first_200_chars_of_body}...

  URL: {issue_url}
  ```

  Then state: "I've classified this as a {issue_type}. Proceeding to process this issue..."
</display_template>

</substep>

</process_flow>

## Step Completion

<completion_checklist>
  Before proceeding to Step 2, verify:
  - [ ] Issue retrieved successfully
  - [ ] Issue data stored (number, title, body, author, url)
  - [ ] Issue classified into one type
  - [ ] Classification displayed to user
</completion_checklist>

<transition>
  **Next Step:** Proceed to @.agent-os/commands/get-issue/single-agent/2-process-issue.md with the stored issue data and classification.
</transition>
