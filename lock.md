# Git Error Resolution Guide

## Error Analysis

You encountered a Git lock file error and missing tag issue in your CI/CD database deployment pipeline. Here's what happened:

### 1. Git Lock File Error
```
error: cannot lock ref 'refs/remotes/origin/LHES-989-LRSC-DB-automation': Unable to create 'C:/runner/***/_work/***/***/vayu-sql-database/.git/refs/remotes/origin/LHES-989-LRSC-DB-automation.lock': File exists.
```

**Root Cause:** A previous Git operation didn't complete cleanly and left a lock file behind. This prevents any subsequent Git operations.

### 2. Missing Git Tags Error
```
fatal: ambiguous argument 'v4.6.131': unknown revision or path not in the working tree.
```

**Root Cause:** The tags `v4.6.131` and `v4.6.146` don't exist in the current Git repository working tree.

## Solution Implemented

The `ci_framework/db_scritps/db_git.sh` file has been updated with:

### 1. Git Lock File Cleanup
- Automatic detection and cleanup of Git lock files
- Retry mechanism with exponential backoff
- Safe cleanup of common lock files including:
  - `.git/index.lock`
  - `.git/refs/heads/*.lock`
  - `.git/refs/remotes/origin/*.lock`

### 2. Enhanced Tag Validation
- Pre-validation of tags before attempting Git diff
- Better error messages with specific tag information
- Graceful handling of missing tags

### 3. Improved Error Handling
- Multiple retry attempts for Git operations
- Specific exit codes for different error conditions:
  - Exit 50: Missing input tags
  - Exit 51: Git diff operation failed
  - Exit 53: Failed to fetch tags after retries
  - Exit 54: Old tag not found in repository
  - Exit 55: New tag not found in repository

## Manual Resolution Steps

If you encounter this error again, you can manually resolve it:

### Step 1: Clean Up Git Lock Files
```bash
# Navigate to the repository directory
cd /path/to/vayu-sql-database

# Remove lock files
find .git -name "*.lock" -type f -delete
rm -f .git/index.lock
rm -f .git/refs/heads/*.lock
rm -f .git/refs/remotes/origin/*.lock
```

### Step 2: Verify Tag Availability
```bash
# Fetch all tags from remote
git fetch --tags

# List all available tags
git tag -l

# Check if specific tags exist
git rev-parse --verify v4.6.131
git rev-parse --verify v4.6.146
```

### Step 3: Alternative Tag Resolution
If the tags are missing, you might need to:

1. **Check the correct repository**: Ensure you're working in the right repository
2. **Fetch from correct remote**: The tags might be in a different remote
3. **Use different tag format**: The tags might have a different naming convention
4. **Use commit hashes**: If tags don't exist, use commit hashes instead

## Prevention Strategies

### 1. Robust CI/CD Pipeline
- Add pre-flight checks for Git repository state
- Implement proper cleanup in CI/CD failure scenarios
- Use timeouts for Git operations

### 2. Tag Management
- Ensure tags are properly pushed to the remote repository
- Implement tag validation in your release process
- Use consistent tag naming conventions

### 3. Error Recovery
- Implement automatic retry mechanisms
- Add proper logging for debugging
- Use lock-free Git operations where possible

## Testing the Fix

You can test the improved script by running:

```bash
# Navigate to your framework directory
cd /Users/shivamkumar/Desktop/kimbal/devops-github-workflow/ci_framework

# Test the database workflow
./main_db.sh --workflow db_migration --workflow-type db_tag_release
```

The improved script will now:
- Automatically handle Git lock files
- Provide better error messages
- Retry operations when appropriate
- Validate tags before attempting diff operations

## Additional Recommendations

1. **Monitor Git Operations**: Add logging to track Git operation success/failure rates
2. **Repository Health Checks**: Implement periodic checks for Git repository integrity  
3. **Tag Synchronization**: Ensure your CI/CD pipeline properly synchronizes tags across environments
4. **Error Alerting**: Set up alerts for specific Git error patterns to catch issues early
