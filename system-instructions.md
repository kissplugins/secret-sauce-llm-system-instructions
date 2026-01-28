PLEASE FOLLOW GENERAL WORDPRESS PLUGIN DEVELOPMENT GUIDELINES

ARCHITECTURE & AUDIT SOURCE OF TRUTH
* Architecture: Follow ARCHITECTURE.md as the authoritative architecture standard.
* Auditing: When asked to audit code, follow AUDIT-INSTRUCTIONS.md as the authoritative audit rubric.
* If any instruction in this file conflicts with those documents, defer to the authoritative document.

DATA PROCESSING & CONVERSION
* Manual Initiation: Do not automatically begin intensive data processing or conversions upon plugin activation. Instead, require manual initiation via a user-clicked button, unless immediate processing is explicitly requested by the user.
* Progress Feedback: Include a simple progress meter or status indicator for plugins performing data processing or conversions.
* Rate Limiting: Implement a default 2–3 second delay between processing each Post unless the user explicitly requests no rate limiting.


WORDPRESS BEST PRACTICES
* Use core WordPress functions and APIs whenever possible instead of creating new functions.
* Follow WordPress coding standards for PHP, JavaScript, and CSS for readability and maintainability.
* Sanitize and validate inputs to enhance security.
* Implement nonces for securing form submissions and AJAX requests against CSRF.
* Localize and internationalize your plugins using WordPress i18n functions.
* Use WordPress database functions ($wpdb) instead of raw SQL queries.
* Minimize queries, optimize performance, and use caching strategically (measure first; design invalidation up front).
* Avoid direct file system access; utilize WordPress filesystem API (WP_Filesystem).
* Conditionally load scripts and styles to separate admin and frontend functionality clearly.
* Utilize actions and filters to extend functionality and allow easier integration by other developers.
* Provide informative error handling, admin notices, and logging for improved debugging and user troubleshooting.
* Time & dates: Store timestamps in UTC; convert to site timezone only for display.
* Round frontend displayed numbers to a maximum of two decimal places and include a thousand separators.
* Provide a direct link to the plugin's settings or status page on the Plugins listing page.
* Increment plugin version numbers with each release.
* Maintain CHANGELOG.md (and README.md if present) with version/date and medium-level details.
* Double-check all code and documentation before final output.
* All newly created WP admin pages for the plugin should be inserted into a new "Plugin Name" Settings page at the top WP admin level unless the user has asked for it to be placed in "Tools". All the pages related to the plugin should be grouped within the same sub-menu and not spread amongst different WP admin menu sub-menus.


FILE STRUCTURE & CHANGE CONTROL
* Follow the existing project structure and patterns.
* Do not refactor existing code outside the current & immediate task/request.
* Avoid reorganizing files or creating new folders unless it is necessary to meet the request or the repo already follows that structure.
* If you add new files/classes, wire them up cleanly (autoloading/`require` strategy consistent with the project) and keep responsibilities modular (per ARCHITECTURE.md).

OUTPUT FILES IN THEIR ENTIREITY
Unless the user specifically asks for all files to be outputted at once (which usally causes rate limting or time outs,) please generate just the first file that needs modifications in its entireity so the user can just copy/paste the whole thing. I'll ask you to output one complete modified file at a time so please wait for my prompt.
