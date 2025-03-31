GENERAL WORDPRESS PLUGIN DEVELOPMENT GUIDELINES

 DATA PROCESSING & CONVERSION
* Manual Initiation: Do not automatically begin intensive data processing or conversions upon plugin activation. Instead, require manual initiation via a user-clicked button, unless immediate processing is explicitly requested by the user.
* Progress Feedback: Include a simple progress meter or status indicator for plugins performing data processing or conversions.
* Rate Limiting: Implement a default 2–3 second delay between processing each Post unless the user explicitly requests no rate limiting.


 WORDPRESS BEST PRACTICES
* Do not refactor code not related to the immediate requested task unless the user has asked for broad refactoring of the code base.
* Use core WordPress functions and APIs whenever possible instead of creating new functions.
* Follow WordPress coding standards for PHP, JavaScript, and CSS for readability and maintainability.
* Sanitize and validate inputs to enhance security.
* Implement nonces for securing form submissions and AJAX requests against CSRF.
* Localize and internationalize your plugins using WordPress i18n functions.
* Use WordPress database functions ($wpdb) instead of raw SQL queries.
* Minimize queries, optimize performance, and aggressively use object caching.
* Avoid direct file system access; utilize WordPress filesystem API (WP_Filesystem).
* Conditionally load scripts and styles to separate admin and frontend functionality clearly.
* Utilize actions and filters to extend functionality and allow easier integration by other developers.
* Provide informative error handling, admin notices, and logging for improved debugging and user troubleshooting.
* Round frontend displayed numbers to a maximum of two decimal places and include a thousand separators.
* Provide a direct link to the plugin's settings or status page on the Plugins listing page.
* Increment plugin version numbers with each release.
* Include changelogs in both the PHP file and README file for clarity.
* Double-check all code and documentation before final output.  

SPECIFICALLY FOR INITIAL PLUGIN (Early 1.x series) DEVELOPMENT 
* Keep PHP, HTML, JS, and CSS inline within a single PHP file unless absolutely necessary.
* Do not combine into single file or split up into multiple files unless explicitly requested by the user.
* If files are requested to be split up into multiple files please make sure to add the proper references to call them in the main plugin file
