# Test-Stub

This directory contains test framework and basic test cases for the DC Power driver.

## Purpose

This stub provides:
- xUnit test framework configuration
- Basic test structure examples
- Test organization patterns
- Placeholder test cases

## Synchronization

**Important:** This directory is synchronized from InstrumentFoundation via GitHub SubTree.

When InstrumentFoundation updates shared test utilities or frameworks, use:
```bash
git subtree pull --prefix=Test-Stub <InstrumentFoundation-repo> <branch> --squash
```

## When Creating a Concrete Driver

1. Rename to match your driver tests (e.g., `N6705C-Tests`)
2. Implement actual test cases with real instrument interactions
3. Add integration tests, performance tests, etc.
4. Configure test fixtures and mocking as needed

## Template Characteristics

- ✅ Tests compile and run
- ✅ Framework properly configured (xUnit)
- ✅ Demonstrates test patterns
- ⚠️ Tests are placeholders (no real validation)
- ✅ Synced with InstrumentFoundation test framework
