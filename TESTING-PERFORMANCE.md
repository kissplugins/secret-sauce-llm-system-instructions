There is not (yet) a well-known, off‑the‑shelf GitHub Action that statically detects unbounded `WP_Query` usage and N+1 patterns in WordPress plugins specifically, but you can get close by combining existing actions with PHP static analysis and performance tooling.[1][2]

## Existing GitHub actions

- WP Performance Tests – measures runtime performance of a WordPress site (TTFB, page load, etc.) from CI, and posts results into PRs, but it does not currently inspect PHP for unbounded or N+1 queries.[1]

## Static analysis for anti‑patterns

To catch bad query patterns at code level, wire PHP analyzers into GitHub Actions and add custom rules:

- PHPStan or Psalm – both can be run as CI steps and extended with plugins or custom rulesets to flag patterns like `posts_per_page => -1`, `no_found_rows => false`, or repeated `get_post_meta()` in loops that often indicate N+1 behavior.[2][3]
- Snyk for PHP – can also run in CI to detect some performance‑related and security‑related query issues, though its focus is broader than WordPress.[2]

Typical approach:

- Add a job that installs dependencies, then runs `phpstan analyse` (or `psalm`) over your plugin.[2]
- Add custom rules or a WordPress‑aware extension to flag:
  - `WP_Query` or `wc_get_products()` without a sane `posts_per_page`/`numberposts` limit  
  - Loops that contain `get_post_meta()`, `get_term_meta()`, or direct `$wpdb->get_results()` calls with interpolated IDs.[4][5]

## Runtime N+1 / unbounded detection

Static analysis alone will miss some N+1 cases, so a second CI job can run WordPress under load and inspect queries:

- Use a small WP install in CI, activate your plugin, hit key pages with `wp-env`/WP‑CLI or curl, and enable `SAVEQUERIES` to log SQL.[6]
- Parse `$wpdb->queries` in a test script to:
  - Fail the build if total queries exceed a threshold for a given route.
  - Fail if the same `SELECT * FROM wp_postmeta WHERE post_id = ?` pattern repeats N times per request (classic N+1).[4][6]

## WordPress‑specific performance rule sets

There are emerging WordPress‑focused rule packs that encode many of these checks:

- claude-wordpress-skills – a public repo that defines a large set of WordPress performance heuristics (unbounded queries, missing limits, N+1 loops, expensive hooks, etc.), which you can adapt into lint rules or checks that run in CI.[7][4]

So today the practical solution is to assemble a workflow: PHPStan/Psalm (plus custom WordPress rules) for static patterns, plus a small runtime harness that inspects `$wpdb->queries` for N+1 and unbounded behavior, rather than a single turnkey “WordPress N+1 GitHub Action.”

[1](https://github.com/marketplace/actions/wp-performance-tests)
[2](https://snyk.io/blog/getting-started-php-static-analysis-2024/)
[3](https://deliciousbrains.com/php-static-code-analysis/)
[4](https://dev.to/n3rdh4ck3r/claude-code-skill-for-wordpress-performance-reviews-1560)
[5](https://github.com/WordPress/developer-blog-content/issues/128)
[6](https://instawp.com/what-is-the-n1-problem-in-wordpress-how-to-fix-it/)
[7](https://github.com/elvismdev/claude-wordpress-skills)
[8](https://github.com/woocommerce/woocommerce/issues/62473)
[9](https://github.com/WordPress/performance/issues/1303)
[10](https://www.reddit.com/r/ClaudeAI/comments/1p746gy/built_a_claude_code_skill_for_wordpress/)
[11](https://github.com/Nxploited/CVE-2025-5287)
[12](https://github.com/woocommerce/action-scheduler/issues/978)
[13](https://github.com/advisories/GHSA-59mj-ghvm-7pcm)
[14](https://laravel-news.com/how-to-detect-n1-queries-in-php)
[15](https://github.com/topics/wordpress-performance)
[16](https://github.com/advisories/GHSA-mp34-36vv-8364)
[17](https://digma.ai/n1-query-problem-and-how-to-detect-it/)
[18](https://github.com/advisories/GHSA-xxw7-22gg-jp7x)
[19](https://joost.blog/github-actions-wordpress/)
[20](https://github.com/advisories/ghsa-9wx6-ggqr-p55x)
[21](https://www.lambdatest.com/blog/performance-testing-tools/)
[22](https://phpstan.org/)
[23](https://www.psalm.dev)
[24](https://snyk.io/articles/devsecops/)
[25](https://mcpmarket.com/tools/skills/wordpress-remote-docker-manager)

===

Claude Opus 4.5 Analysis

The core recommendation is correct—there's no turnkey solution, so you assemble PHPStan/Psalm plus runtime query inspection. But the analysis has some issues worth noting:

**Source quality is mixed.** Several of those citations are GitHub security advisories (GHSA-*) that have nothing to do with query detection—looks like the search pulled in noise. The claude-wordpress-skills repo reference is legitimate and potentially useful, though.

**The static analysis approach is undersold.** You don't need "emerging rule packs"—you can write targeted PHPStan rules fairly quickly. The patterns you care about are finite:

```php
// These are greppable without sophisticated analysis:
'posts_per_page' => -1
'numberposts' => -1
'nopaging' => true
->get_results( without LIMIT
```

A simple bash script in CI catches 80% of unbounded queries:

```bash
grep -rn "posts_per_page.*=>.*-1\|numberposts.*=>.*-1\|nopaging.*=>.*true" \
  --include="*.php" src/ && exit 1
```

**The N+1 detection is harder than presented.** Static analysis genuinely struggles here because the pattern is contextual—`get_post_meta()` inside a loop isn't always N+1 if the meta was primed earlier. The runtime `SAVEQUERIES` approach is more reliable but requires representative test coverage to actually trigger the code paths.

**What I'd actually do for your plugins:**

1. Simple grep-based pre-commit hook for obvious unbounded patterns
2. PHPStan baseline with WordPress stubs, add rules incrementally  
3. A lightweight integration test that hits key endpoints and asserts `count($wpdb->queries) < N`

The Perplexity answer points in the right direction but oversells the tooling maturity. For a plugin like your Hourly Monitor, manual code review with your SMSPD checklist is probably more effective than trying to automate this today.
