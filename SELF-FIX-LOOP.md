```markdown
# Automated Self-Verification Pattern: Agent Instructions

## Your Objective

Create a closed-loop testing workflow where you generate test data, submit it to the application, verify the results programmatically, and iterate until success—WITHOUT human intervention.

## Core Workflow

1. **Generate Test Data**: Create JSON, SQL, or structured data for the test case
2. **Build Submission Script**: Write code to submit/import the data into the target system
3. **Execute Submission**: Run your script to apply changes
4. **Verify Results**: Use curl/CLI tools to programmatically check if changes worked
5. **Analyze & Iterate**: If verification fails, examine the output and adjust your approach
6. **Repeat**: Continue until verification passes

## Critical Rules

### Pause for Human Review
- **After 5 failed iterations**: STOP and report findings to the human
- **After 10 total iterations**: STOP regardless of status and request guidance
- **Before destructive operations**: Always confirm with human first (database drops, bulk deletes, production changes)

### Simplification Checkpoints
If you find yourself:
- Writing overly complex verification logic → **PAUSE**: Ask human if a simpler approach exists
- Creating multi-step workarounds → **PAUSE**: Suggest refactoring the underlying system
- Repeating the same verification pattern 3+ times → **PAUSE**: Propose extracting it into a reusable function
- Unable to verify programmatically → **PAUSE**: Ask human for manual verification or alternative approach

## Implementation Examples

### WordPress/WooCommerce Pattern
```bash
# 1. Generate test data
cat > test-product.json << 'EOF'
{"name": "Test Product", "price": "29.99", ...}
EOF

# 2. Submit to system
cat test-product.json | local-wp mysite eval-file import-product.php

# 3. Verify result
PRODUCT_URL=$(local-wp mysite post url $PRODUCT_ID)
curl -s "$PRODUCT_URL" | grep "29.99" && echo "✅ Pass" || echo "❌ Fail"

# 4. If fail → analyze HTML response, adjust JSON, repeat
```

### API Development Pattern
```bash
# 1. Generate test cases
node generate-api-tests.js > test-cases.json

# 2. Submit requests
node run-api-tests.js test-cases.json > results.json

# 3. Verify responses
node verify-results.js results.json
# Check: status codes, schema validation, error messages

# 4. If fail → examine results.json, fix endpoint code, repeat
```

### Database Migration Pattern
```bash
# 1. Write migration
cat > migrations/001_add_users_table.sql

# 2. Apply migration
psql -f migrations/001_add_users_table.sql

# 3. Verify schema
psql -c "\d users" | grep "id.*integer.*primary key" && echo "✅ Pass"

# 4. If fail → analyze schema, adjust SQL, rollback, repeat
```

## Your Iteration Template

For each test cycle, follow this structure:

```
ITERATION N:
1. What I'm testing: [describe goal]
2. Generated data: [show test data]
3. Submission command: [show exact command]
4. Expected result: [what should happen]
5. Actual result: [what happened]
6. Verification output: [show curl/CLI output]
7. Status: ✅ PASS / ❌ FAIL
8. Next action: [if fail, what to adjust]
```

## When to Stop and Ask for Help

**Stop immediately if:**
- You've tried 5 different approaches and all failed
- The verification is consistently returning unexpected results you can't explain
- You need to modify production data or systems
- The underlying application behavior seems broken (not just your test)
- You're creating increasingly complex workarounds instead of fixing the root issue

**Suggest simplification if:**
- Your verification script is >50 lines of code
- You're parsing HTML with regex instead of using proper tools
- You're chaining 4+ commands to verify one thing
- The same verification pattern appears in 3+ test cases

## Success Criteria

You've completed the task when:
1. ✅ Test data is generated programmatically
2. ✅ Submission script executes without errors
3. ✅ Verification confirms expected behavior
4. ✅ Process is repeatable (can run again with same results)
5. ✅ All verification commands are documented

## Remember

- **Iterate fast**: Don't overthink, try the simplest approach first
- **Verify everything**: Never assume it worked without checking
- **Document as you go**: Each iteration should be traceable
- **Know when to stop**: 5 failed iterations = time to ask for help
- **Simplify when stuck**: Complex solutions often indicate wrong approach

Now begin your automated verification workflow.
```
