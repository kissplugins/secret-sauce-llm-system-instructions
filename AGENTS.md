# WordPress Development Guidelines for AI Agents

_Last updated: v2.0.0 — 2025-10-09_

## 👩‍💻 Purpose

This document defines the principles, constraints, and best practices that AI agents must follow when working with WordPress code repositories. The goal is to ensure safe, consistent, and maintainable contributions across security, functionality, and documentation.

---

## 🔐 Security

- [ ] **Sanitize all inputs** using WordPress functions (`sanitize_text_field()`, `sanitize_email()`, `absint()`, etc.)
- [ ] **Escape all outputs** using appropriate functions (`esc_html()`, `esc_attr()`, `esc_url()`, `wp_kses_post()`)
- [ ] **Verify nonces** for all form submissions and AJAX requests
- [ ] **Check capabilities** using `current_user_can()` before allowing actions
- [ ] **Validate and authenticate** everything - never trust user input
- [ ] **Use `$wpdb->prepare()`** for all database queries to prevent SQL injection
- [ ] **Never expose sensitive data** (passwords, tokens, API keys) in logs, comments, or commits
- [ ] **Avoid custom security logic** when WordPress native APIs exist

---

## ⚡ Performance

- [ ] **No unbound queries** - always use LIMIT clauses and pagination
- [ ] **Cache expensive operations** using WordPress Transients API
- [ ] **Minimize HTTP requests** - batch operations when possible
- [ ] **Minimize database calls** - use `WP_Query` efficiently, avoid queries in loops
- [ ] **Optimize only when requested** - don't prematurely optimize code

---

## 🏗️ The WordPress Way

- [ ] **Use WordPress APIs and hooks** - don't reinvent the wheel (`wp_remote_get()`, `wp_schedule_event()`, etc.)
- [ ] **Follow DRY principles** - reuse existing helper functions, create new helpers when needed
- [ ] **Follow WordPress Coding Standards** for [PHP](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/), [JavaScript](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/), [CSS](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/), and [HTML](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/html/)
- [ ] **Respect plugin/theme hierarchy** - maintain existing file and folder structures
- [ ] **Use WordPress actions, filters, and template tags** for extensibility
- [ ] **Treat plugins/themes as self-contained** - avoid cross-dependencies unless requested
- [ ] **Use `function_exists()` checks** when adding new functions to avoid redeclaration errors

---

## 🔧 Scope & Change Control

- [ ] **Stay within task scope** - only perform explicitly requested tasks
- [ ] **No refactoring** unless explicitly requested
- [ ] **No renaming** functions, variables, classes, or files unless instructed
- [ ] **No label changes** (taxonomy labels, admin menu labels) without explicit guidance
- [ ] **No speculative improvements** or architectural changes
- [ ] **Preserve existing data structures** (arrays, objects, database schema) unless absolutely necessary
- [ ] **Maintain naming conventions** consistent with the existing project
- [ ] **Prioritize preservation over optimization** when in doubt

---

## 📝 Documentation & Versioning

- [ ] **Use PHPDoc standards** for all functions and classes
- [ ] **Add inline documentation** for complex logic
- [ ] **Increment version numbers** in plugin/theme headers when making changes
- [ ] **Update CHANGELOG.md** with version number, date, and medium-level details of changes
- [ ] **Update README.md** when adding major features or changing usage
- [ ] **Maintain Table of Contents** if present in documentation
- [ ] **Document data structure changes** with clear justification

**PHPDoc Example**:
```php
/**
 * Get the user's display name.
 *
 * @since 1.0.0
 * @param int $user_id The ID of the user.
 * @return string The display name.
 */
function get_user_display_name( $user_id ) {
    // Implementation
}
```

---

## 🧪 Testing & Validation

- [ ] **Preserve existing functionality** - avoid breaking changes
- [ ] **Test all changes** before considering complete
- [ ] **Add self-tests** for new features when appropriate
- [ ] **Validate security implementations** (nonces, capabilities, sanitization)
- [ ] **Ensure backward compatibility** unless explicitly breaking changes are requested

---

## ✅ Pre-Commit Checklist

Before completing any task, verify:

- [ ] Stayed strictly within the scope of the task
- [ ] Did not rename or relabel code unintentionally
- [ ] Applied WordPress security best practices (sanitize, escape, nonce, capabilities)
- [ ] Preserved existing data structures unless necessary
- [ ] Reused existing functionality when possible (DRY principle)
- [ ] Used PHPDoc-style comments for any new or changed code
- [ ] Updated the version number in plugin/theme header
- [ ] Updated CHANGELOG.md with version, date, and details
- [ ] No unbound queries or performance issues introduced
- [ ] Followed WordPress Coding Standards
- [ ] Used WordPress APIs instead of custom implementations

---

## 📋 Quick Reference

### Security Functions
- **Input**: `sanitize_text_field()`, `sanitize_email()`, `sanitize_url()`, `absint()`, `wp_unslash()`
- **Output**: `esc_html()`, `esc_attr()`, `esc_url()`, `esc_js()`, `wp_kses_post()`
- **Nonces**: `wp_nonce_field()`, `wp_create_nonce()`, `check_admin_referer()`, `wp_verify_nonce()`
- **Capabilities**: `current_user_can()`, `user_can()`
- **Database**: `$wpdb->prepare()`, `$wpdb->get_results()`, `$wpdb->insert()`

### Performance Functions
- **Caching**: `get_transient()`, `set_transient()`, `delete_transient()`
- **HTTP**: `wp_remote_get()`, `wp_remote_post()`, `wp_safe_remote_get()`
- **Queries**: `WP_Query`, `get_posts()`, `wp_cache_get()`, `wp_cache_set()`

### WordPress APIs
- **Options**: `get_option()`, `update_option()`, `delete_option()`, `add_option()`
- **Hooks**: `add_action()`, `add_filter()`, `do_action()`, `apply_filters()`
- **AJAX**: `wp_ajax_{action}`, `wp_ajax_nopriv_{action}`, `wp_send_json_success()`, `wp_send_json_error()`
- **Scheduling**: `wp_schedule_event()`, `wp_schedule_single_event()`, `wp_clear_scheduled_hook()`

---

_This document consolidates all WordPress development guidelines for AI agents. Follow these principles to ensure safe, maintainable, and WordPress-compliant code._

