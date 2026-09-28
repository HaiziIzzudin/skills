---
name: playwright-with-docs
description: Automated visual testing with Playwright and PDF documentation generation. Execute visual tests against a local application (localhost:8000), capture screenshots during test flows, and generate comprehensive PDF documentation with step-by-step instructions, screenshots, and manual verification guidance. Use this skill whenever you need to test an application visually, create test evidence through screenshots, and produce user-facing or QA documentation. Trigger on requests like "test this feature visually", "create test documentation", "visual regression testing", or when a user provides a PRD/Spec and wants automated testing paired with PDF output.
---

# Playwright with Docs

Automated visual testing with Playwright combined with PDF documentation generation. This skill executes visual tests, captures and organizes screenshots, and produces comprehensive PDF guides with step-by-step instructions and visual evidence.

## Workflow Overview

```
1. Check @TEST-ACCOUNT.md existence
   ├─ If missing: Warn user, pause, offer auto-generation
   └─ If exists: Parse credentials

2. Verify server connectivity (localhost:8000)
   ├─ Check if service is running
   └─ Exit if unreachable

3. Read PRD/Spec/Tickets
   └─ Extract test scenarios and requirements

4. Execute visual tests with Playwright
   ├─ Screenshot each step (auto-numbered 01, 02, 03...)
   ├─ Store in .playwright-screenshots/<##-topic-name>/
   └─ Stop on first failure (report error immediately)

5. Generate PDF documentation
   ├─ Title + functionality overview
   ├─ Table of contents
   ├─ Step-by-step instructions with inline screenshots
   └─ Save to docs/pdf/<filename>.pdf
```

## Input Requirements

When invoking this skill, provide:

1. **@TEST-ACCOUNT.md location**: Path to credentials file (plain markdown format)
   ```markdown
   email: test.user@example.com
   password: SecurePassword123
   ```
   The skill will warn if file is missing and pause for user decision.

2. **PRD/Spec/Tickets location**: Full file path (user mentions directly)
   - Supported formats: .md, .pdf, .docx
   - Should contain test scenarios, requirements, and acceptance criteria

3. **Test media folder (optional)**: If tests require uploading files
   - User specifies folder path in skill invocation
   - Skill will reference media from this location

## Step-by-Step Execution

### Phase 1: Preparation and Validation

1. **Credentials Check**
   - Attempt to read @TEST-ACCOUNT.md from provided or inferred path
   - If file not found:
     - Display warning with file path attempted
     - Pause and ask user: "Generate test credentials automatically? (yes/no)"
     - If yes: Create @TEST-ACCOUNT.md with auto-generated credentials
     - If no: Exit with error

2. **Server Connectivity**
   - Attempt to ping http://localhost:8000
   - If unreachable:
     - Check if port 8000 is open (netstat/lsof)
     - Suggest troubleshooting: "Server not running. Start it and retry."
     - Exit if cannot connect

3. **Load Specification**
   - Read provided PRD/Spec file
   - Extract test scenarios (numbered, with descriptions)
   - Identify key workflows to test

### Phase 2: Visual Testing

1. **Initialize Playwright**
   - Create browser context with test credentials
   - Note: Use Playwright browser tools ONLY (not playwright_browser_run_code_unsafe)
   - Navigate to http://localhost:8000
   
2. **Execute Test Scenarios**
   - For each scenario in spec:
     - Navigate to relevant page/feature
     - Perform user actions (click, type, submit)
     - Take screenshot after each meaningful step
     - Store as: `01-scenario-name.png`, `02-next-step.png`, etc.
     - Folder structure: `.playwright-screenshots/<##-scenario-topic>/`
     - Create folder if it does not exist
   
3. **Failure Handling**
   - If ANY visual test fails:
     - STOP immediately
     - Report which test failed and why
     - Show screenshot evidence
     - Do NOT continue to remaining tests
     - Provide error output for debugging

4. **Success Criteria**
   - All steps execute without errors
   - Screenshots captured successfully
   - Visual elements match expected layout

### Phase 3: PDF Documentation Generation

Use the **pdf skill** to generate comprehensive documentation:

1. **Document Structure**
   - Cover page: Title + brief description
   - Table of contents (auto-generated)
   - Functionality overview (from spec)
   - Step-by-step instructions section:
     - Each step numbered (Step 1, Step 2, etc.)
     - Description of action
     - Associated screenshot embedded inline
     - Expected outcome noted

2. **Screenshot Integration**
   - Include screenshots in sequential order
   - Place directly after corresponding step instruction
   - Add captions: "Screenshot: [Description of what to do]"
   - Maintain readable layout (fit 1 screenshot per step or page)

3. **File Organization**
   - Create folder if needed: `docs/pdf/`
   - Save as: `docs/pdf/<feature-name>-visual-test-guide.pdf`
   - Ensure path is absolute or relative to project root



## Constraints and Allowed Actions

### Allowed Actions
- Create @TEST-ACCOUNT.md if user approves
- Create .playwright-screenshots/ folder structure
- Create docs/pdf/ folder structure
- Write and upload test content inside the app
- Edit code if behavior doesn't align with tickets (only if necessary for tests)
- Database migration (if required for test setup)
- Navigate purely by browser snapshots (Playwright visual testing)
- Account creation from test credentials (if not exists)

### Prohibited Actions
- Start another dev server or database server
- Use tool "playwright_browser_run_code_unsafe"
- Continue testing after first failure
- Make assumptions about missing spec details
- Overwrite existing test credentials without approval

## Error Handling and Exit Scenarios

| Condition | Action |
|-----------|--------|
| @TEST-ACCOUNT.md not found | Warn user, pause, offer auto-gen |
| Server not running at localhost:8000 | Attempt diagnostic, suggest restart, exit |
| Connection still fails after diagnostic | Exit with clear error message |
| Visual test fails | Stop immediately, report failure, show evidence |
| PDF generation fails | Report error, provide screenshot folder anyway |
| Spec file not readable | Ask user for alternative path or content |

## Output Deliverables

After successful execution, provide:

1. **PDF Document**
   - Location: `docs/pdf/<feature-name>-visual-test-guide.pdf`
   - Contains: Title, TOC, functionality overview, step-by-step instructions with inline screenshots

2. **Screenshot Folder**
   - Location: `.playwright-screenshots/<##-scenario-topic>/`
   - Contents: Sequential PNG files (01-action.png, 02-result.png, etc.)

3. **Test Summary**
   - Total scenarios tested
   - Pass/fail status
   - Any errors encountered
   - Links to generated artifacts

## Example Skill Invocation

```
Test the login flow using Playwright:
- Spec: /projects/myapp/REQUIREMENTS.md
- Credentials: /projects/myapp/@TEST-ACCOUNT.md
- Media folder: /projects/myapp/test-media/
- Test scenarios: Login, Dashboard access, Logout
```

Expected outcome: Screenshots in `.playwright-screenshots/01-login/`, PDF in `docs/pdf/login-flow-visual-test-guide.pdf`, manual verification steps displayed.

## Notes for Implementation

- Always read the spec BEFORE executing tests to understand expected behavior
- Pause before auto-generating credentials to ask user for approval
- Number screenshots sequentially across ALL test scenarios (01, 02, 03... not restarting per scenario)
- Embed screenshots directly in PDF, not as appendix
- Provide clear, actionable error messages if any step fails
- Test accessibility and responsiveness if spec requires it
