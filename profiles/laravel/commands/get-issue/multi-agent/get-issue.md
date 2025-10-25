---
description: Process GitHub issues and route to appropriate workflow
globs:
alwaysApply: false
version: 1.0
encoding: UTF-8
---

# Get Issue Command

<ai_meta>
  <parsing_rules>
    - Process XML blocks first for structured data
    - Execute instructions in sequential order
    - Use GitHub CLI for all issue operations
    - Delegate to specialized agents for different issue types
  </parsing_rules>
  <file_conventions>
    - encoding: UTF-8
    - line_endings: LF
    - indent: 2 spaces
    - markdown_headers: no indentation
  </file_conventions>
</ai_meta>

## Overview

<purpose>
  - Retrieve and process GitHub issues
  - Classify issues by type (Bug, Feature Request, Patch, Question)
  - Route to appropriate workflow based on issue type
</purpose>

<context>
  - Part of Agent OS framework
  - Uses GitHub CLI for issue management
  - Delegates to spec creation for complex work
</context>

<prerequisites>
  - GitHub CLI (gh) installed and authenticated
  - Git repository with GitHub remote
  - Access to Agent OS spec creation workflow
</prerequisites>

<process_flow>

<step number="1" name="issue_retrieval">

### Step 1: Issue Retrieval

<step_metadata>
  <requires>GitHub CLI</requires>
  <accepts>issue number, title, or interactive selection</accepts>
</step_metadata>

<retrieval_logic>
  <if_argument_provided>
    <check_numeric>
      <action>gh issue view [NUMBER]</action>
      <error_handling>proceed to interactive selection</error_handling>
    </check_numeric>
    <check_string>
      <action>gh issue list --search "[STRING]"</action>
      <error_handling>proceed to interactive selection</error_handling>
    </check_string>
  </if_argument_provided>
  <if_no_argument_or_no_match>
    <action>gh issue list --limit 20</action>
    <display>recent issues for selection</display>
    <wait>user selection</wait>
  </if_no_argument_or_no_match>
</retrieval_logic>

<instructions>
  ACTION: Attempt to retrieve issue from provided argument
  FALLBACK: Show list of recent issues for selection
  RETRIEVE: Full issue details once identified
</instructions>

</step>

<step number="2" name="issue_classification">

### Step 2: Issue Classification

<step_metadata>
  <analyzes>issue content</analyzes>
  <classifies>into one of four types</classifies>
</step_metadata>

<classification_criteria>
  <bug>
    - Reports unexpected behavior or errors
    - Describes something that should work but doesn't
    - Contains error messages or stack traces
    - Uses language like "broken", "fails", "doesn't work"
  </bug>
  <feature_request>
    - Proposes new functionality
    - Describes enhancement or improvement
    - Uses language like "would be nice", "should add", "request"
    - Contains user stories or use cases
  </feature_request>
  <patch>
    - Includes code changes or diffs
    - References specific files/lines to modify
    - Contains solution in issue body
    - May include "patch" or "fix" in description
  </patch>
  <question>
    - Asks for help or clarification
    - Seeks documentation or guidance
    - Uses question marks
    - Not clearly a bug or feature request
  </question>
</classification_criteria>

<instructions>
  ACTION: Analyze issue title and body
  CLASSIFY: Determine which type best fits
  PROCEED: To appropriate workflow based on classification
</instructions>

</step>

<step number="3" name="bug_workflow">

### Step 3: Bug Workflow

<step_metadata>
  <applies_to>issues classified as bugs</applies_to>
  <determines>bug complexity and routing</determines>
</step_metadata>

<bug_process>
  <information_check>
    <verify>
      - Steps to reproduce
      - Expected behavior
      - Actual behavior
      - Environment details (if relevant)
    </verify>
    <if_insufficient>
      <action>gh issue comment [NUMBER] --body "[REQUEST]"</action>
      <tag>issue reporter in comment</tag>
      <exit>workflow</exit>
    </if_insufficient>
  </information_check>

  <complexity_assessment>
    <simple_bug>
      - Single component affected
      - Clear reproduction steps
      - Likely single file/method fix
      - Can verify with single test
    </simple_bug>
    <complex_bug>
      - Multiple components involved
      - Unclear root cause
      - Requires multiple file changes
      - Needs multiple tests for coverage
    </complex_bug>
  </complexity_assessment>

  <simple_bug_flow>
    <create_branch>
      <name_format>{issue_number}-{kebab-case-title}</name_format>
      <base>main</base>
    </create_branch>
    <create_regression_test>
      <purpose>demonstrate the bug</purpose>
      <verify>test fails initially</verify>
    </create_regression_test>
    <fix_bug>
      <apply>minimal fix</apply>
      <verify>regression test passes</verify>
    </fix_bug>
    <if_fix_successful>
      <create_pr>
        <target>main</target>
        <link>issue number</link>
        <auto_close>on merge</auto_close>
      </create_pr>
    </if_fix_successful>
    <if_fix_unsuccessful>
      <reclassify>complex bug</reclassify>
      <proceed>to complex bug flow</proceed>
    </if_fix_unsuccessful>
  </simple_bug_flow>

  <complex_bug_flow>
    <delegate>
      <command>@.agent-os/instructions/create-spec.md</command>
      <context>bug issue details</context>
      <purpose>document fix requirements</purpose>
    </delegate>
  </complex_bug_flow>
</bug_process>

<agent_delegation>
  <for>simple bug fixes</for>
  <agent_type>bug-fixer</agent_type>
  <agent_prompt>
    You are a bug fixer agent. Your task:

    1. Create branch: {issue_number}-{issue-title}
    2. Write regression test that demonstrates the bug
    3. Run test to verify it fails
    4. Fix the bug with minimal changes
    5. Run test to verify it passes
    6. Run `composer ready` to ensure no breakage
    7. Create PR linking to issue #{issue_number}

    Issue Details:
    {issue_body}

    If you cannot fix the bug after reasonable attempts, report back that this should be reclassified as a complex bug.
  </agent_prompt>
</agent_delegation>

<instructions>
  ACTION: Check for sufficient information
  ASSESS: Bug complexity
  DELEGATE: To bug-fixer agent for simple bugs
  ROUTE: To spec creation for complex bugs
</instructions>

</step>

<step number="4" name="feature_request_workflow">

### Step 4: Feature Request Workflow

<step_metadata>
  <applies_to>issues classified as feature requests</applies_to>
  <routes_to>spec creation</routes_to>
</step_metadata>

<feature_process>
  <context_check>
    <verify>
      - Clear description of desired functionality
      - Use cases or user stories
      - Acceptance criteria (if provided)
      - Any technical considerations mentioned
    </verify>
    <if_insufficient>
      <action>gh issue comment [NUMBER] --body "[REQUEST]"</action>
      <tag>issue reporter in comment</tag>
      <exit>workflow</exit>
    </if_insufficient>
  </context_check>

  <spec_creation>
    <delegate>
      <command>@.agent-os/instructions/create-spec.md</command>
      <context>feature request details</context>
      <purpose>document feature requirements</purpose>
    </delegate>
  </spec_creation>
</feature_process>

<instructions>
  ACTION: Verify sufficient context exists
  REQUEST: Missing information if needed
  DELEGATE: To spec creation workflow
</instructions>

</step>

<step number="5" name="patch_workflow">

### Step 5: Patch Workflow

<step_metadata>
  <applies_to>issues containing patches</applies_to>
  <applies_quickly>patch and verify</applies_quickly>
</step_metadata>

<patch_process>
  <create_branch>
    <name_format>{issue_number}-{kebab-case-title}</name_format>
    <base>main</base>
  </create_branch>
  <apply_patch>
    <extract>code changes from issue</extract>
    <apply>to appropriate files</apply>
  </apply_patch>
  <verify>
    <run>composer ready</run>
    <ensure>all checks pass</ensure>
  </verify>
  <create_pr>
    <target>main</target>
    <link>issue number</link>
    <auto_close>on merge</auto_close>
  </create_pr>
</patch_process>

<agent_delegation>
  <for>patch application</for>
  <agent_type>patch-applier</agent_type>
  <agent_prompt>
    You are a patch applier agent. Your task:

    1. Create branch: {issue_number}-{issue-title}
    2. Apply the patch from the issue
    3. Run `composer ready` to verify nothing broke
    4. Create PR linking to issue #{issue_number}

    Issue Details:
    {issue_body}

    Patch to Apply:
    {patch_code}
  </agent_prompt>
</agent_delegation>

<instructions>
  ACTION: Create branch for patch
  APPLY: Code changes from issue
  VERIFY: composer ready passes
  CREATE: PR linking to issue
</instructions>

</step>

<step number="6" name="question_workflow">

### Step 6: Question Workflow

<step_metadata>
  <applies_to>issues classified as questions</applies_to>
  <provides>guidance or clarification</provides>
</step_metadata>

<question_process>
  <analyze>
    <understand>what user is asking</understand>
    <check>documentation for answer</check>
    <check>codebase for examples</check>
  </analyze>
  <respond>
    <action>gh issue comment [NUMBER] --body "[ANSWER]"</action>
    <include>
      - Clear explanation
      - Code examples if relevant
      - Documentation links if available
      - Suggestions for further help
    </include>
  </respond>
  <close_or_convert>
    <if_answered>
      <suggest>closing issue (let reporter close)</suggest>
    </if_answered>
    <if_actually_bug_or_feature>
      <reclassify>to appropriate type</reclassify>
      <proceed>to appropriate workflow</proceed>
    </if_actually_bug_or_feature>
  </close_or_convert>
</question_process>

<instructions>
  ACTION: Understand the question
  RESEARCH: Find answer in docs or code
  RESPOND: With helpful comment
  SUGGEST: Closing or reclassification
</instructions>

</step>

</process_flow>

## Agent Orchestration

<multi_agent_strategy>
  <coordinator_role>
    - Retrieve and classify issue
    - Determine workflow routing
    - Delegate to specialized agents
    - Monitor completion and summarize
  </coordinator_role>

  <specialized_agents>
    <bug_fixer>
      <handles>simple bug fixes</handles>
      <creates>branches, tests, fixes, PRs</creates>
    </bug_fixer>
    <patch_applier>
      <handles>patch application</handles>
      <creates>branches, applies changes, PRs</creates>
    </patch_applier>
    <spec_creator>
      <handles>complex bugs and features</handles>
      <delegates_to>create-spec workflow</delegates_to>
    </spec_creator>
  </specialized_agents>
</multi_agent_strategy>

## Error Handling

<error_scenarios>
  <scenario name="issue_not_found">
    <condition>Invalid issue number or no matches</condition>
    <action>Show list of recent issues</action>
  </scenario>
  <scenario name="insufficient_information">
    <condition>Cannot proceed without more details</condition>
    <action>Comment on issue requesting information</action>
  </scenario>
  <scenario name="fix_unsuccessful">
    <condition>Simple bug cannot be fixed</condition>
    <action>Reclassify as complex and create spec</action>
  </scenario>
  <scenario name="patch_fails_checks">
    <condition>composer ready fails after patch</condition>
    <action>Report failure, suggest reclassification</action>
  </scenario>
</error_scenarios>

## Completion Summary

<summary_template>
  ## Issue Processing Complete: #{issue_number}

  **Classification:** {issue_type}
  **Title:** {issue_title}

  ### Actions Taken

  {actions_list}

  ### Next Steps

  {next_steps}

  ### Links

  - Issue: {issue_url}
  - PR: {pr_url} (if created)
  - Spec: {spec_path} (if created)
</summary_template>

<final_checklist>
  <verify>
    - [ ] Issue retrieved and classified
    - [ ] Appropriate workflow executed
    - [ ] Branch created (if applicable)
    - [ ] Tests written/run (if applicable)
    - [ ] PR created or spec documented
    - [ ] Issue linked to resolution
  </verify>
</final_checklist>
