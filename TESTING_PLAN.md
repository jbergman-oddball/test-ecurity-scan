# Testing Plan: Security Scan Workflow & Package Change Detection

## Context

The security-scan.yml workflow and detect_package_changes.py script need to be tested before moving to production. The current infrastructure doesn't have robust testing patterns, so we need a practical, isolated testing approach that:

- Tests the Python script locally first
- Uses a separate test repository to validate the full workflow
- Avoids breaking production workflows
- Provides clear validation steps
- Add a new line
- Devops 4 life

## Approach

### Phase 1: Local Python Script Testing (No GitHub Actions Required)

**Goal**: Validate the detect_package_changes.py script works correctly with different scenarios.

**Steps**:

1. Create a local test script `test_detect_changes.sh` in `.github/scripts/` that:
   - Creates test commits with known package.json changes
   - Runs the Python script against those commits
   - Validates the output matches expected results

2. Test scenarios to cover:
   - Adding a new dependency
   - Removing a dependency
   - Changing a version
   - Modifying non-dependency fields (scripts, etc.)
   - No changes at all

3. The test script will:
   ```bash
   # Create a test branch
   # Make controlled changes to package.json
   # Run: python3 detect_package_changes.py <base_sha> <head_sha>
   # Verify output
   ```

**Validation**: Script outputs correct `any_changed=true/false` and alert_lines match expected changes.

---

### Phase 2: Test Repository Setup

**Goal**: Create a minimal test repository to validate the full GitHub Actions workflow.

**Repository Structure**:

```
test-security-scan-repo/
├── package.json           # Simple Node.js package
├── .github/
│   ├── workflows/
│   │   └── security-scan.yml          # Copy from main repo
│   │   └── start_slack_thread.yml     # Simplified version (no actual Slack)
│   └── scripts/
│       └── detect_package_changes.py  # Copy from main repo
└── README.md             # Documentation
```

**Package.json Content** (minimal):

```json
{
  "name": "test-security-scan",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

**Workflow Modifications for Testing**:

- Comment out or mock the `start_slack_thread` notification step
- Set workflow to trigger on any branch (not just develop/main)
- Keep the bypass-package-lock environment optional or remove for testing

---

### Phase 3: Test Workflow Execution

**Goal**: Run the workflow through different test scenarios.

**Test Cases**:

1. **Test Case 1: Add Dependency**
   - Create branch `test/add-dependency`
   - Add a new dependency: `"lodash": "^4.17.21"`
   - Push and observe workflow behavior
   - Expected: `any_changed=true`, approval required

2. **Test Case 2: Change Version**
   - Create branch `test/change-version`
   - Update existing dependency version
   - Push and observe workflow behavior
   - Expected: `any_changed=true`, changes detected in alert_lines

3. **Test Case 3: Non-Dependency Change**
   - Create branch `test/script-change`
   - Only modify the "scripts" section
   - Push and observe workflow behavior
   - Expected: `any_changed=false`, workflow skips approval

4. **Test Case 4: Remove Dependency**
   - Create branch `test/remove-dependency`
   - Remove a dependency
   - Push and observe workflow behavior
   - Expected: `any_changed=true`, removal detected

**Validation**: Each scenario triggers the correct workflow behavior and outputs accurate change detection.

---

### Phase 4: Integration Back to Main Repo

**Goal**: Once validated, safely integrate back to the production repository.

**Steps**:

1. Keep the test repository for future regression testing
2. If changes are needed to the scripts/workflow, apply them to main repo
3. Test on a feature branch in main repo (not develop/main) first
4. Monitor the first production run closely

---

## Critical Files

### To Create (for testing):

- `.github/scripts/test_detect_changes.sh` - Local testing script
- Test repository (external, new repo)

### To Copy to Test Repo:

- `.github/workflows/security-scan.yml`
- `.github/scripts/detect_package_changes.py`
- `.github/workflows/start_slack_thread.yml` (simplified)

### To Monitor:

- `.github/workflows/security-scan.yml:111-112` - The approval condition
- `.github/scripts/detect_package_changes.py:214-221` - Output format

---

## Verification Steps

### Local Testing:

```bash
cd .github/scripts
chmod +x test_detect_changes.sh
./test_detect_changes.sh
# Should output: All tests passed ✓
```

### Test Repository:

1. Create test repo on GitHub
2. Copy files
3. Run each test case (4 test branches)
4. Verify GitHub Actions runs and outputs
5. Check approval flow works correctly

### Production Validation:

1. Test on non-production branch in main repo
2. Monitor first real run
3. Verify Slack notifications work
4. Confirm approval gate functions

---

## Risk Mitigation

- **No production impact**: All testing happens in isolated environments
- **Rollback plan**: Keep current workflow, test in parallel on test branches
- **Clear validation**: Each phase has clear pass/fail criteria
- **Incremental**: Can stop at any phase if issues found
