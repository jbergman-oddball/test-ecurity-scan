# Test Security Scan Repository

This repository is used to test and validate the security scan workflow and package change detection system before deploying to production.

## Purpose

- Test the `detect_package_changes.py` script
- Validate the `security-scan.yml` GitHub Actions workflow
- Ensure package.json change detection works correctly
- Verify approval gates function as expected

## Structure

```
test-security-scan/
├── package.json                           # Minimal Node.js package for testing
├── TESTING_PLAN.md                        # Comprehensive testing plan
├── .github/
│   ├── workflows/
│   │   ├── security-scan.yml              # Main security scan workflow
│   │   └── start_slack_thread.yml         # Simplified notification workflow
│   └── scripts/
│       └── detect_package_changes.py      # Package change detection script
└── README.md                              # This file
```

## Test Cases

This repository will be used to test the following scenarios:

1. **Add Dependency** - Adding a new package dependency
2. **Change Version** - Updating an existing dependency version
3. **Non-Dependency Change** - Modifying scripts or other non-dependency fields
4. **Remove Dependency** - Removing an existing dependency

## Usage

1. Create test branches for each scenario (e.g., `test/add-dependency`)
2. Make the relevant package.json changes
3. Push and observe the GitHub Actions workflow behavior
4. Verify that change detection and approval gates work correctly

## Expected Outcomes

- `any_changed=true` when dependencies/versions change
- `any_changed=false` when only non-dependency fields change
- Accurate change detection in alert_lines output
- Proper approval gate triggering

## Related Documentation

See [TESTING_PLAN.md](./TESTING_PLAN.md) for the complete testing strategy.
