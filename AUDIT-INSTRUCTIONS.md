Audit this WordPress plugin/theme against these criteria in priority order:

1. **Security** (Critical)
   - Missing sanitization (`sanitize_*`, `wp_kses*`, `absint`, `esc_*`)
   - Missing output escaping in HTML/JS/URL/attribute contexts
   - Missing/improper nonce verification on forms and AJAX
   - Missing capability checks (`current_user_can`)
   - SQL injection risks (direct `$wpdb` without `prepare()`)
   - AJAX handlers: `wp_ajax_nopriv_*` without proper authorization
   - REST endpoints missing `permission_callback` (never `__return_true` without reason)
   - File uploads without MIME/extension validation
   - `unserialize()`/`maybe_unserialize()` on untrusted input
   - User-controlled URLs in `wp_remote_*` (SSRF)
   - Options that could enable privilege escalation

2. **Performance** (High)
   - Unbounded queries (missing LIMIT, posts_per_page = -1)
   - Missing pagination on large datasets
   - N+1 query patterns (queries inside loops)
   - Queries on frontend that should be cached/transient
   - Missing object caching consideration for repeated lookups
   - Autoloaded options that shouldn't be (`autoload = yes` on large data)

3. **WordPress API Compliance** (Medium-High)
   - Direct SQL when WP_Query/WP_User_Query/get_posts would work
   - Custom implementations of core functionality
   - Scripts/styles not using wp_enqueue_* system
   - Missing hook prefixes (namespace collisions)
   - Direct file operations instead of WP_Filesystem
   - Multisite: `switch_to_blog` without `restore_current_blog`

4. **Error Handling & Data Integrity** (Medium)
   - External API calls without try/catch or timeout handling
   - Silent failures that could cause data loss
   - Missing WP_Error checks on fallible operations
   - Debug output left in code (var_dump, print_r, error_log)
   - Missing database transactions for multi-step operations

5. **Code Quality** (Low-Medium)
   - DRY violations: repeated code > 3 lines appearing 2+ times
   - Hardcoded strings that should use __() or _e()
   - Missing/inadequate PHPDoc on public functions
   - Deprecated function usage
   - Improper activation/deactivation/uninstall hooks

For each issue found, note:
- File and line number
- Severity: Critical (security/data loss) / High (performance/compliance) / Medium / Low
- Specific violation with code snippet
- Recommended fix with code example

Output format: Create/append to AUDIT.md with:
1. Summary stats (issues by severity)
2. Findings organized by criteria
3. Actionable TODO checklist prioritized by severity
