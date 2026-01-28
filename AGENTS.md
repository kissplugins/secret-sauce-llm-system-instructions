# WordPress Development Guidelines for AI Agents

_Last updated: v2.1.0 — 2026-01-28_

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

### Core Requirements
- [ ] **Declare PHP v7 or higher** via plugin header `Requires PHP: 7.0` (supports `\Throwable`)
- [ ] **Check for namespace/class name conflicts** - use unique prefixes or namespaces to avoid global collisions
- [ ] **Use `function_exists()` and `class_exists()` checks** when adding new functions/classes to avoid redeclaration errors
- [ ] **Use WordPress APIs and hooks** - don't reinvent the wheel (`wp_remote_get()`, `wp_schedule_event()`, etc.)
- [ ] **Follow DRY principles** - reuse existing helper functions, create new helpers when needed
- [ ] **Follow WordPress Coding Standards** for [PHP](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/), [JavaScript](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/), [CSS](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/), and [HTML](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/html/)
- [ ] **Respect plugin/theme hierarchy** - maintain existing file and folder structures
- [ ] **Use WordPress actions, filters, and template tags** for extensibility
- [ ] **Treat plugins/themes as self-contained** - avoid cross-dependencies unless requested

### Error Prevention
- [ ] **Avoid undefined index/property notices** - use `isset()`, `??` operator, or `array_key_exists()` before accessing array keys
- [ ] **Validate variable existence** - check variables are defined before use (especially in JavaScript)
- [ ] **Handle missing configuration gracefully** - provide defaults for optional settings
- [ ] **Add try-catch blocks** for operations that may throw exceptions (regex construction, API calls, etc.)

### Client-Side Security & Performance
- [ ] **Never expose sensitive data to client-side storage** - avoid storing plugin versions, paths, settings URLs in localStorage/sessionStorage
- [ ] **Use sessionStorage over localStorage** for admin-only data (scoped to single tab, auto-clears on close)
- [ ] **Implement cache cleanup on logout/unload** - clear sensitive data when admin session ends
- [ ] **Gate client-side storage with admin context checks** - verify user is in `/wp-admin/` before caching
- [ ] **Escape user input before RegExp construction** - sanitize metacharacters to prevent SyntaxError and ReDoS attacks
- [ ] **Implement Page Visibility API** for polling/intervals - pause expensive operations when tab is hidden
- [ ] **Use server-side transient caching** for expensive operations (filesystem scans, API calls) instead of repeated execution

### State Hygiene & Single Contract Writers
- [ ] **Establish Single Source of Truth (SSoT)** - designate ONE authoritative source for each piece of state (e.g., FSM, database, options table)
- [ ] **Use single contract writers** - only ONE class/function should write to each state; all others must read through it
- [ ] **Avoid parallel state tracking** - never duplicate state in multiple places (cache, database, class properties, etc.)
- [ ] **Derive computed values** - calculate dependent values from SSoT instead of storing them separately
- [ ] **Handle serialization boundaries** - when state crosses boundaries (JSON, transients, AJAX), convert back to proper types (enums, objects)
- [ ] **Validate state consistency** - add guards to detect when state becomes inconsistent across sources
- [ ] **Document state ownership** - clearly comment which class/function owns each piece of state
- [ ] **Centralize state transitions** - use dedicated methods/classes for state changes, not direct property assignment

**State Hygiene Example**:
```php
// ❌ BAD: Parallel state tracking
$is_active = get_post_meta( $post_id, 'is_active', true );
$status = get_post_meta( $post_id, 'status', true ); // Duplicates is_active
update_option( 'cached_status_' . $post_id, $status ); // Third copy!

// ✅ GOOD: Single Source of Truth with derived values
class OrderStateManager {
    // SSoT: Only this class writes to order_state
    public function set_state( $order_id, OrderState $state ): void {
        update_post_meta( $order_id, 'order_state', $state->value );
        do_action( 'order_state_changed', $order_id, $state );
    }

    // All reads go through SSoT
    public function get_state( $order_id ): OrderState {
        $value = get_post_meta( $order_id, 'order_state', true );
        return OrderState::from( $value ?: 'pending' );
    }

    // Derived values computed from SSoT
    public function is_active( $order_id ): bool {
        return in_array(
            $this->get_state( $order_id ),
            [ OrderState::PROCESSING, OrderState::SHIPPED ],
            true
        );
    }
}
```

### Defensive Error Handling
- [ ] **Check for WP_Error** - always validate return values from WordPress functions that may return `WP_Error`
- [ ] **Use null coalescing** - prefer `??` operator for safe defaults instead of ternary or isset checks
- [ ] **Validate types before operations** - especially with enums, objects, and serialized data
- [ ] **Graceful degradation** - fail safely without breaking the site; provide fallback behavior
- [ ] **Proper error logging** - use `error_log()` for debugging, never `var_dump()` or `print_r()` in production
- [ ] **Never expose technical details to users** - show friendly messages, log technical details
- [ ] **Check database errors** - validate `$wpdb->last_error` after queries
- [ ] **Handle API failures** - wrap HTTP requests in try-catch, check for errors, provide fallbacks
- [ ] **Use WordPress admin notices** - communicate errors to admins via `add_settings_error()` or admin notices
- [ ] **Provide sensible defaults** - when data is missing or corrupt, use safe fallback values

**Defensive Error Handling Example**:
```php
// ❌ BAD: No error checking
$response = wp_remote_get( $api_url );
$data = json_decode( wp_remote_retrieve_body( $response ) );
$value = $data->items[0]->value; // Multiple failure points!

// ✅ GOOD: Defensive error handling
$response = wp_remote_get( $api_url );
if ( is_wp_error( $response ) ) {
    error_log( sprintf( 'API request failed: %s', $response->get_error_message() ) );
    return $this->get_cached_fallback(); // Graceful degradation
}

$body = wp_remote_retrieve_body( $response );
$data = json_decode( $body );

if ( json_last_error() !== JSON_ERROR_NONE ) {
    error_log( sprintf( 'JSON decode failed: %s', json_last_error_msg() ) );
    return [];
}

$value = $data->items[0]->value ?? 'default_value'; // Null coalescing for safety
```

### Observability & Debugging
- [ ] **Add strategic logging** - log state transitions, API calls, cache hits/misses, and error conditions
- [ ] **Use consistent log prefixes** - prefix logs with plugin/feature name for easy filtering (e.g., `SBI:`, `PQS:`)
- [ ] **Log context, not just values** - include relevant IDs, states, and operation names
- [ ] **Add observability when stuck** - if debugging a bug, add temporary logging to trace execution flow
- [ ] **Log before/after critical operations** - helps identify where failures occur
- [ ] **Include type information** - log variable types when debugging type-related issues (e.g., `gettype()`, `instanceof`)
- [ ] **Remove verbose logging after debugging** - clean up temporary debug logs before committing
- [ ] **Use WordPress debug constants** - respect `WP_DEBUG` and `WP_DEBUG_LOG` settings

**Observability Example**:
```php
// ✅ GOOD: Strategic logging for debugging
error_log( sprintf(
    'SBI: Processing cache check for %s (key: %s) - found: %s',
    $full_name,
    $cache_key,
    $cached ? 'YES' : 'NO'
) );

// ✅ GOOD: Type validation logging
if ( ! ( $state instanceof PluginState ) ) {
    error_log( sprintf(
        'SBI: Invalid state type for %s: %s (expected PluginState enum)',
        $repo_name,
        gettype( $state )
    ) );
}
```

---

## 🏗️ Building from the Ground Up

When creating new features or plugins from scratch, follow this checklist:

- [ ] **Start with DRY helpers** - create reusable utility functions before writing feature code
- [ ] **Design single contract writers** - identify state ownership and create dedicated manager classes
- [ ] **Separate concerns** - split logic into distinct layers (data access, business logic, presentation)
- [ ] **Add observability from the start** - include logging for key operations and state changes
- [ ] **Implement defensive error handling** - validate inputs, check for errors, provide fallbacks
- [ ] **Use WordPress APIs** - leverage built-in functions instead of reinventing (caching, HTTP, database)
- [ ] **Plan for extensibility** - add hooks and filters for future customization
- [ ] **Document as you build** - write PHPDoc comments and inline documentation immediately
- [ ] **Consider FSM early** - if feature has 3+ states, design state machine from the start
- [ ] **Write tests alongside code** - create unit tests for critical business logic

**Ground-Up Example Structure**:
```php
// 1. DRY Helpers (utilities.php)
function prefix_sanitize_repo_name( $name ) { /* ... */ }
function prefix_format_error_message( $error ) { /* ... */ }

// 2. Single Contract Writer (StateManager.php)
class StateManager {
    public function set_state( $id, $state ) { /* SSoT writer */ }
    public function get_state( $id ) { /* SSoT reader */ }
}

// 3. Separation of Concerns
// - DataAccess.php (database/API calls)
// - BusinessLogic.php (rules, validation)
// - Presentation.php (rendering, formatting)

// 4. Observability
error_log( 'PREFIX: Feature initialized' );

// 5. Defensive Error Handling
if ( is_wp_error( $result ) ) { /* handle */ }
```

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

- [ ] **Use PHPDoc and JSDoc standards** for all functions and classes
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

## 🔄 When to Transition to Finite State Machine (FSM)

Recommend transitioning to a Finite State Machine when features exhibit these characteristics:

### Signs You Need an FSM

- [ ] **Multiple states** - Feature has 3+ distinct states (e.g., draft, pending, approved, published)
- [ ] **Complex transitions** - State changes depend on multiple conditions or user roles
- [ ] **State-dependent behavior** - Different actions are available in different states
- [ ] **Validation rules** - Certain transitions are only valid from specific states
- [ ] **Audit requirements** - Need to track state history and transition reasons
- [ ] **Concurrent states** - Multiple state dimensions (e.g., approval status + payment status)
- [ ] **Workflow complexity** - Business logic becomes difficult to track with simple flags/booleans

### When to Recommend FSM to User

**Recommend FSM if:**
- Feature has grown beyond 2-3 boolean flags tracking status
- You find yourself writing nested if/else statements to determine valid actions
- State logic is duplicated across multiple files or functions
- Debugging state-related issues becomes time-consuming
- New state requirements keep being added to existing features
- State transitions need to trigger specific actions (hooks, notifications, logging)

### FSM Implementation Approach

**Suggest to user:**
1. **Define states clearly** - List all possible states the entity can be in
2. **Map transitions** - Document which state changes are valid (state diagram)
3. **Centralize state logic** - Create a dedicated class/file for state management
4. **Use WordPress metadata** - Store current state in post_meta or options table
5. **Add transition hooks** - Fire actions on state changes for extensibility
6. **Log transitions** - Track who changed state, when, and why (audit trail)

### Example FSM Scenarios

- **Order processing**: pending → processing → completed → refunded
- **Content workflow**: draft → review → approved → published → archived
- **User onboarding**: registered → verified → profile_complete → active
- **Support tickets**: open → assigned → in_progress → resolved → closed

### Red Flags (Don't Use FSM)

- Feature only has 2 states (use simple boolean)
- States never transition (use static status field)
- No validation rules for transitions (simple status update is sufficient)
- Over-engineering a simple feature

**When in doubt, ask the user**: "This feature is tracking [X] states with [Y] transitions. Would you like me to implement a Finite State Machine for better maintainability?"

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
- **Output**: `esc_html()`, `esc_attr()`, `esc_url ()`, `esc_js()`, `wp_kses_post()`
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
