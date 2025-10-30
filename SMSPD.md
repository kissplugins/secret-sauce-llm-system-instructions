# SMSPD Development Philosophy

**SMS** (text messaging) + **PD** (police department) = **SMSPD**

A WordPress development checklist focusing on quality attributes that matter.

---

## 🔒 **S** - Secure

Security-first development to protect users and data.

### Checklist
- [ ] All form submissions use WordPress nonces (`wp_nonce_field()` / `wp_verify_nonce()`)
- [ ] User input is sanitized appropriately (`sanitize_text_field()`, `sanitize_email()`, `esc_url()`, etc.)
- [ ] Output is escaped based on context (`esc_html()`, `esc_attr()`, `esc_url()`, `wp_kses()`)
- [ ] Database queries use prepared statements (`$wpdb->prepare()`)
- [ ] User capabilities checked before privileged operations (`current_user_can()`)
- [ ] AJAX requests validate nonces and permissions
- [ ] File uploads restricted by type and validated
- [ ] API endpoints authenticate and authorize requests
- [ ] Sensitive data (API keys, credentials) stored securely (not in code)
- [ ] SQL injection vectors eliminated
- [ ] XSS (Cross-Site Scripting) vulnerabilities prevented
- [ ] CSRF (Cross-Site Request Forgery) protections in place

**Ask yourself:** "Could a malicious user exploit this feature?"

---

## 🔧 **M** - Maintainable

Reduce technical debt by doing things the right way from the start.

### Checklist
- [ ] Code follows WordPress Coding Standards (WPCS)
- [ ] Functions are single-purpose and reasonably sized (<50 lines ideally)
- [ ] Clear, descriptive naming conventions used (no `$temp`, `$data1`, `function1()`)
- [ ] DRY principle applied (Don't Repeat Yourself)
- [ ] Magic numbers replaced with named constants
- [ ] Dependencies are minimal and well-justified
- [ ] Error handling is comprehensive (try/catch, validation checks)
- [ ] Code is modular and loosely coupled
- [ ] Third-party libraries kept up to date
- [ ] Technical debt is tracked and addressed regularly
- [ ] No commented-out code blocks left in production
- [ ] Consistent file and folder structure

**Ask yourself:** "Could another developer understand and modify this code in 6 months?"

---

## 📈 **S** - Scalable

Architecture supports growth without requiring rewrites.

### Checklist
- [ ] Database schema supports future feature expansion
- [ ] Code architecture isn't locked into specific implementation details
- [ ] Caching strategy implemented where appropriate (transients, object cache)
- [ ] Pagination used for large data sets
- [ ] Background processing for heavy tasks (WP Cron or better alternatives)
- [ ] Database indexes optimized for common queries
- [ ] Asset loading optimized (only load what's needed, when needed)
- [ ] No architectural decisions that would require significant refactoring at scale
- [ ] API rate limiting considered for public endpoints
- [ ] Image optimization and lazy loading implemented
- [ ] Consider multi-site compatibility if relevant

**Ask yourself:** "Will this work with 1,000 users? 10,000 users? 100,000 records?"

---

## ⚡ **P** - Performant

Fast, efficient code that respects user time and server resources.

### Checklist
- [ ] **No unbounded queries** - all queries have limits (`posts_per_page`, `number`, etc.)
- [ ] Queries only select needed fields (avoid `SELECT *` when possible)
- [ ] N+1 query problems avoided (use `WP_Query` tax_query, meta_query wisely)
- [ ] Expensive operations are cached appropriately
- [ ] Database queries optimized and indexed
- [ ] Transients used for data that doesn't change frequently
- [ ] External API calls are cached or asynchronous
- [ ] Scripts and styles minified and concatenated for production
- [ ] Critical CSS inlined, non-critical deferred
- [ ] Images properly sized and optimized
- [ ] Unnecessary database queries eliminated in loops
- [ ] Profiling done to identify bottlenecks

**Ask yourself:** "Is this the most efficient way to accomplish this task?"

---

## 📝 **D** - Documented

Clear documentation for present and future developers.

### Checklist
- [ ] All functions have PHPDoc comments (description, `@param`, `@return`, `@throws`)
- [ ] All JavaScript functions have JSDoc comments
- [ ] Complex logic has inline comments explaining "why", not just "what"
- [ ] README.md exists with setup instructions
- [ ] Project plan document exists and is maintained
- [ ] Specific feature/module plans documented before implementation
- [ ] CHANGELOG.md actively maintained with:
  - What was built
  - Why decisions were made
  - Key learnings
  - Breaking changes
- [ ] Environment requirements documented
- [ ] API endpoints documented (parameters, responses, authentication)
- [ ] Database schema changes documented
- [ ] Deployment process documented
- [ ] Known issues/limitations documented

**Ask yourself:** "If I left tomorrow, could someone else continue this work?"

---

## Implementation Guidelines

### Before Starting Development
1. Create/update project plan document
2. Review SMSPD checklist for relevant items
3. Identify potential security, scalability, or performance concerns
4. Document architectural decisions

### During Development
1. Write documentation as you code (not after)
2. Test with realistic data volumes
3. Review your own code against SMSPD before submitting
4. Update CHANGELOG.md with decisions and learnings

### Before Marking Complete
1. Full SMSPD checklist review
2. Security audit (especially nonces, sanitization, escaping)
3. Performance testing (check for unbounded queries)
4. Documentation completeness check
5. Update all relevant documentation

---

## Quick Reference: WordPress Security Functions

```php
// Nonces
wp_nonce_field('action_name', 'nonce_field_name');
wp_verify_nonce($_POST['nonce_field_name'], 'action_name');

// Sanitization (Input)
sanitize_text_field()    // General text
sanitize_email()         // Email addresses
sanitize_url()           // URLs
absint()                 // Positive integers
sanitize_textarea_field() // Textarea content

// Escaping (Output)
esc_html()              // HTML content
esc_attr()              // HTML attributes
esc_url()               // URLs
esc_js()                // JavaScript strings
wp_kses()               // Allow specific HTML tags

// Database
$wpdb->prepare("SELECT * FROM {$wpdb->posts} WHERE ID = %d", $id);

// Permissions
current_user_can('edit_posts')
```

---

## Quick Reference: Performance Patterns

```php
// ✅ GOOD - Bounded query
$args = array(
    'post_type' => 'post',
    'posts_per_page' => 20,  // Always set a limit
);

// ❌ BAD - Unbounded query
$args = array(
    'post_type' => 'post',
    'posts_per_page' => -1,  // Gets ALL posts - dangerous at scale
);

// ✅ GOOD - Cache expensive operations
$results = get_transient('my_expensive_query');
if (false === $results) {
    $results = expensive_database_operation();
    set_transient('my_expensive_query', $results, HOUR_IN_SECONDS);
}

// ✅ GOOD - Only load assets when needed
if (is_singular('custom_post_type')) {
    wp_enqueue_script('custom-feature-js');
}
```

---

## Version History

**Version 1.0** - Initial SMSPD Framework
- Established core principles: Secure, Maintainable, Scalable, Performant, Documented
- Created comprehensive checklist format
- Added WordPress-specific examples and quick reference

---

*Remember: SMSPD isn't about perfection—it's about consistently making better decisions that prevent future problems.*
