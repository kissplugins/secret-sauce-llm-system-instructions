# agents.md

_Last updated: v1.0.0 — 2025-08-07_

## 👩‍💻 Purpose

This document defines the principles, constraints, and best practices that Large Language Model (LLM) agents must follow when interacting with WordPress-related code repositories. The goal is to ensure safe, consistent, and maintainable contributions across security, functionality, and documentation efforts.

---

## 🔐 WordPress Security Best Practices

- Always prioritize secure practices when writing or modifying code.
- Sanitize all user inputs using appropriate WordPress functions, such as `sanitize_text_field()` or `sanitize_email()`.
- Escape all outputs using functions like `esc_html()` and `wp_kses_post()`.
- Always verify user capabilities using functions like `current_user_can()`.
- Avoid introducing custom security logic when a native WordPress API exists.
- Never expose sensitive data (user emails, passwords, tokens) in logs, comments, or commits.

---

## 🔧 Scope & Change Control

- Only perform tasks that are explicitly described in the prompt or ticket.
- Do not refactor code or alter its structure unless explicitly requested.
- Avoid speculative improvements or architectural changes.
- Code should not be optimized unless the task is directly related to performance.

---

## 🏷️ Naming & Labeling

- Do not rename functions, variables, classes, or files unless directly instructed.
- Avoid changing labels (e.g., taxonomy labels, admin menu labels) without explicit guidance.
- Maintain consistency with existing naming conventions in the project.

---

## 🧱 Data Structures

- Do not modify data structures (arrays, objects, database schema, class properties) unless:
  - It is absolutely necessary for the task, and
  - You include clear justification and isolate the changes.
- Document any such changes and update related documentation.

---

## 🧩 Code Style & Reuse

- Follow the [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/) for PHP, JavaScript, CSS, and HTML.
- Follow the **DRY principle** (Don't Repeat Yourself):
  - Reuse existing helper functions or patterns instead of duplicating logic.
- Place newly created helpers in the appropriate location (e.g., `helpers.php`, `inc/`).

**Use PHPDoc-style comments** for any added or modified code. Examples include:

```php
/**
 * Get the user's display name.
 *
 * @param int $user_id The ID of the user.
 * @return string The display name.
 */
```

---

## 📝 Documentation & Versioning

- If a change is made, **increment the version number** and reflect it in any versioned metadata files.
- Maintain the Table of Contents (if present). Add new entries or update existing ones as needed.
- If a `CHANGELOG.md` or a changelog section exists in `README.md`:
  - Add a descriptive summary of the change.
  - Include the version number and current date.

---

## 🧪 Testing & Validation

- Ensure that all changes:
  - Preserve the functionality of existing templates and features.
  - Avoid introducing breaking changes.
- Use `function_exists()` or equivalent when adding new functions to avoid redeclaration errors.
- Prefer WordPress's native actions, filters, and template tags for extensibility and forward compatibility.

---

## 🛠️ Plugin & Theme Context

- Treat each plugin or theme as a **self-contained scope**.
- Avoid introducing dependencies between components unless explicitly requested.
- Respect existing file and folder structures.

---

## ⚠️ Final Reminders

- Use `$wpdb->prepare()` for database queries to prevent SQL injection.
- Prefer WordPress APIs for common tasks (e.g., `wp_remote_get()`, `wp_schedule_event()`).
- When adding new features, include inline comments and clear documentation blocks.
- When in doubt, prioritize **preservation over optimization**.

---

## ✅ Agent Checklist

- [ ] Stayed strictly within the scope of the task
- [ ] Did not rename or relabel code unintentionally
- [ ] Applied WordPress security best practices
- [ ] Preserved existing data structures unless necessary
- [ ] Reused existing functionality when possible
- [ ] Used PHPDoc-style comments for any new or changed code
- [ ] Updated the version number and changelog (if present)

---

Let me know if you’d like this turned into a downloadable `.md` file or inserted into a code repo structure!
````
