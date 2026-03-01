# 10.1 Git Bisect

## What is Bisect?

Binary search through commits to find which commit introduced a bug.

**Scenario:** Your tests passed 100 commits ago, but fail now. Which commit broke it?

## Basic Bisect Workflow

```bash
# Start bisecting
git bisect start

# Mark current commit as bad (bug exists)
git bisect bad

# Mark a known good commit (bug didn't exist)
git bisect good abc1234

# Git checks out middle commit
# Test it manually
npm test  # or whatever test

# If test passes:
git bisect good

# If test fails:
git bisect bad

# Git keeps halving until it finds the first bad commit

# When done:
git bisect reset
```

## Visual Example

```
Commits: A -- B -- C -- D -- E -- F -- G (bad)
                                     ↑
                              Known good

Bisect process:
1. Test D (middle): bad
   Range: A-D (bad exists here)

2. Test B (middle of A-D): good
   Range: B-D

3. Test C (middle of B-D): good
   Range: C-D

4. Test D: bad
   Found: D is the first bad commit!
```

## Automated Bisect

```bash
# Start bisect
git bisect start HEAD abc1234  # bad commit, good commit

# Run automated test
git bisect run npm test

# Git automatically:
# - Tests each commit
# - Marks good/bad based on exit code (0=good, 1+=bad)
# - Finds first bad commit
# - Shows result

# Reset when done
git bisect reset
```

<aside>
💡

**Pro tip:** Automated bisect with `git bisect run` is incredibly powerful. Any script that exits with 0 (success) or non-zero (failure) works.

</aside>

## Bisect with Script

```bash
# Create test script
cat > [test.sh](http://test.sh) << 'EOF'
#!/bin/bash
# Exit 0 if good, 1 if bad
npm test
exit $?
EOF

chmod +x [test.sh](http://test.sh)

# Run bisect with script
git bisect start HEAD v1.0.0
git bisect run ./[test.sh](http://test.sh)
```

## Advanced Bisect Commands

```bash
# Skip commits (e.g., won't compile)
git bisect skip

# Skip a range of commits
git bisect skip abc1234..def5678

# Visualize bisect progress
git bisect visualize
# OR
git bisect view

# Log bisect session
git bisect log

# Replay bisect session from log
git bisect replay <log-file>
```

## Bisect with Multiple Known States

```bash
# If you know multiple good/bad commits
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Mark additional known states
git bisect good abc1234
git bisect bad def5678
```

## Complete Example

```bash
# Bug appeared somewhere in last 50 commits
# Tests passed at commit tagged v1.0.0

# 1. Start bisect
git bisect start HEAD v1.0.0

# 2. Git checks out middle commit
# Checking out abc1234... (25 commits to test)

# 3. Test it
npm test
# Test failed!

# 4. Mark as bad
git bisect bad
# Bisecting: 12 commits left to test

# 5. Test again
npm test
# Test passed!

# 6. Mark as good
git bisect good
# Bisecting: 6 commits left to test

# ... continue until...
# def5678 is the first bad commit
# Author: John Doe
# Date: 2024-01-15
# Message: Refactor authentication

# 7. Review the bad commit
git show def5678

# 8. Reset
git bisect reset
```

## Bisect for Performance Regression

```bash
# Create performance test script
cat > [perf-test.sh](http://perf-test.sh) << 'EOF'
#!/bin/bash
# Test if performance is acceptable
time npm test
if [ $? -gt 5 ]; then
  exit 1  # Too slow (bad)
else
  exit 0  # Fast enough (good)
fi
EOF

chmod +x [perf-test.sh](http://perf-test.sh)
git bisect start HEAD v1.0.0
git bisect run ./[perf-test.sh](http://perf-test.sh)
```

## Bisect Exit Codes

**Script exit codes for `git bisect run`:**

- **0**: Good commit
- **1-127 (except 125)**: Bad commit
- **125**: Commit cannot be tested (skip)
- **126+**: Abort bisect

```bash
# Example with skip
cat > [test.sh](http://test.sh) << 'EOF'
#!/bin/bash
make
if [ $? -ne 0 ]; then
  exit 125  # Won't compile, skip
fi
make test
exit $?
EOF
```

## Bisect Best Practices

1. **Have a reliable test** - automated tests work best
2. **Use clean commits** - each commit should compile/run
3. **Tag known-good versions** - easier to bisect later
4. **Automate when possible** - `git bisect run` saves time
5. **Keep bisect logs** - save with `git bisect log > bisect.log`
6. **Test builds first** - if commits don't compile, fix that first

## Bisect Troubleshooting

```bash
# If you marked wrong
# Reset and start over
git bisect reset

# Or use reflog to undo
git reflog
git bisect good/bad <previous-state>

# If many commits won't build
git bisect skip
# Git will try nearby commits
```

## Finding When Feature Was Added

```bash
# Reverse: find when feature appeared
# Use "new" and "old" instead of "good" and "bad"

git bisect start --term-new=with-feature --term-old=without-feature
git bisect with-feature HEAD
git bisect without-feature v1.0.0

# Test for feature presence
git bisect run ./[has-feature.sh](http://has-feature.sh)
```

## Saving Bisect Session

```bash
# Save progress
git bisect log > my-bisect.log

# Later, resume
git bisect replay my-bisect.log
```

## Common Bisect Scenarios

### Scenario 1: Test Suite Failure

```bash
git bisect start HEAD v2.0.0
git bisect run npm test
# Finds commit that broke tests
```

### Scenario 2: Performance Regression

```bash
# Script checks if run time > 5 seconds
git bisect start HEAD v1.5.0
git bisect run ./[check-performance.sh](http://check-performance.sh)
```

### Scenario 3: Build Failure

```bash
# Some commits won't compile
git bisect start HEAD last-good-build
git bisect run make
# Skip non-compiling commits automatically
```

## Bisect with Complex Conditions

```bash
# Test multiple conditions
cat > [complex-test.sh](http://complex-test.sh) << 'EOF'
#!/bin/bash
# Must pass all tests AND be fast
make test || exit 1
time make run | grep "Time: [0-5]" || exit 1
exit 0
EOF

git bisect run ./[complex-test.sh](http://complex-test.sh)
```