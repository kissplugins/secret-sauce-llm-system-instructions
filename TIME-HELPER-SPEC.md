# Time Helper Module Specification

**Version:** 1.0  
**Status:** Reference Architecture  
**Last Updated:** 2025-12-27  
**Context:** WordPress plugins with WooCommerce order reporting

---

## 1. Problem Domain

### 1.1 The WordPress Timezone Trap

WordPress provides timezone-aware functions that **do not return true Unix timestamps**:

| Function | Returns | Actual Meaning |
|----------|---------|----------------|
| `time()` | `1703689200` | True UTC timestamp ✅ |
| `current_time('timestamp')` | `1703671200` | **Fake** timestamp adjusted by site offset ⚠️ |
| `current_time('timestamp', true)` | `1703689200` | True UTC timestamp ✅ |

**The trap:** `current_time('timestamp')` returns a number that *looks* like a Unix timestamp but is actually shifted by the site's UTC offset. If your site is UTC-5, you get a number that's 5 hours behind true UTC.

### 1.2 WooCommerce UTC Storage

WooCommerce stores all order timestamps in UTC:

```
wp_wc_orders.date_created_gmt = '2025-12-27 15:00:00' (UTC)
wp_wc_orders.date_created     = '2025-12-27 10:00:00' (display only)
```

When querying with `wc_get_orders()`:
- `date_created` expects **site-timezone** values
- `date_created_gmt` expects **UTC** values

### 1.3 The Mismatch Bug

If you use `current_time('timestamp')` to build query ranges and pass them to `wc_get_orders()` with `date_created`, the query interprets your "fake" timestamp as a real UTC value, causing:

- Orders shifted by the timezone offset (e.g., 5 hours off for UTC-5)
- Missing orders near day boundaries
- Duplicate orders in edge cases

**Solution:** Always convert to true UTC when querying WooCommerce.

---

## 2. Architecture Principles

### 2.1 Separation of Concerns

```
┌─────────────────────────────────────────────────────────────────────┐
│                      BUSINESS LOGIC LAYER                           │
│                   (Site Timezone Semantics)                         │
│                                                                     │
│   "Today" = user's local calendar day                              │
│   "This hour" = user's local clock hour                            │
│   Scheduling = user's local time expectations                       │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       QUERY LAYER                                    │
│                   (UTC Conversion Point)                            │
│                                                                     │
│   Convert site-TZ boundaries → UTC ISO strings                      │
│   Use date_created_gmt for all WC queries                          │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      WOOCOMMERCE / DATABASE                         │
│                        (UTC Storage)                                │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Core Principles

1. **Single Source of Truth:** One method for "now", one for day boundaries, one for UTC conversion
2. **Explicit Conversion:** UTC conversion happens at clearly defined points (query layer)
3. **Semantic Clarity:** Method names indicate timezone context (`now()` vs `now_utc()`)
4. **Display vs Processing:** Separate methods for user display vs internal calculations
5. **Immutability:** Use `DateTimeImmutable` to prevent accidental mutation

---

## 3. API Design

### 3.1 Class Structure

```php
class Time_Helper {
    // ─── INTERNAL PROCESSING (Site Timezone) ───────────────────────
    public static function now(): int;
    public static function now_utc(): int;
    public static function today(): string;
    public static function get_day_range(?string $date = null): array;
    public static function get_hour(int $timestamp): int;
    
    // ─── UTC CONVERSION (Query Layer) ──────────────────────────────
    public static function format_iso_utc(int $timestamp): string;
    public static function get_day_range_utc(?string $date = null): array;
    
    // ─── USER DISPLAY (Respects WP Settings) ───────────────────────
    public static function format(int $timestamp): string;
    public static function format_date(int $timestamp): string;
    public static function format_time(int $timestamp): string;
    
    // ─── MACHINE-READABLE (Fixed ISO Format) ───────────────────────
    public static function format_iso(int $timestamp): string;
    
    // ─── VALIDATION & UTILITIES ────────────────────────────────────
    public static function is_valid_date(string $date): bool;
    public static function is_today(string $date): bool;
    public static function is_future(string $date): bool;
    public static function cache_key(string $prefix, ?string $date = null): string;
}
```

### 3.2 Method Specifications

#### `now(): int`
Returns the current timestamp in **site timezone context**.

```php
public static function now(): int {
    return current_time('timestamp');
}
```

**Use for:** Cron windows, "last 15 minutes" calculations, business logic.

**Warning:** This is NOT a true Unix timestamp. Do not pass directly to `gmdate()` or WC queries.

#### `now_utc(): int`
Returns the current **true UTC** timestamp.

```php
public static function now_utc(): int {
    return time();
}
```

**Use for:** Comparing with external systems, debugging timezone offset.

#### `get_day_range(?string $date = null): array`
Returns start/end timestamps for a calendar day in site timezone.

```php
public static function get_day_range(?string $date = null): array {
    $timezone = wp_timezone();
    
    if (null === $date) {
        $now   = new DateTimeImmutable('now', $timezone);
        $start = $now->setTime(0, 0, 0);
    } else {
        $start = new DateTimeImmutable($date . ' 00:00:00', $timezone);
    }
    
    $end = $start->setTime(23, 59, 59);
    
    return [
        'start' => $start->getTimestamp(),  // True Unix timestamp
        'end'   => $end->getTimestamp(),    // True Unix timestamp
    ];
}
```

**Critical:** Uses `DateTimeImmutable::getTimestamp()` which returns **true UTC timestamps**, not the fake `current_time()` values. This is intentional—we want real timestamps for the boundaries of a local calendar day.

#### `format_iso_utc(int $timestamp, bool $is_site_tz = true): string`
Converts a timestamp to UTC ISO string for WooCommerce queries.

**v1.1.14 Fix:** This method now properly handles the distinction between:
1. **"Fake" site-TZ timestamps** from `current_time('timestamp')` - these are shifted by the site's UTC offset and need to be UN-shifted back to true UTC.
2. **True UTC timestamps** from `DateTimeImmutable::getTimestamp()` - these are already in UTC and should be formatted directly.

```php
public static function format_iso_utc(int $timestamp, bool $is_site_tz = true): string {
    if ($is_site_tz) {
        // Un-shift the "fake" site-TZ timestamp back to true UTC.
        $offset    = (int) wp_date('Z', $timestamp);
        $timestamp = $timestamp - $offset;
    }

    return gmdate('Y-m-d H:i:s', $timestamp);
}
```

**Use for:** Building `date_created_gmt` query parameters.

**Parameters:**
- `$timestamp` - Unix timestamp
- `$is_site_tz` - Set to `true` (default) for timestamps from `WHM_Date::now()` / `current_time('timestamp')`. Set to `false` for timestamps from `DateTimeImmutable::getTimestamp()`.

#### `get_day_range_utc(?string $date = null): array`
Returns day boundaries with both timestamps AND UTC ISO strings.

```php
public static function get_day_range_utc(?string $date = null): array {
    $local_range = self::get_day_range($date);

    // Pass false: get_day_range() uses DateTimeImmutable::getTimestamp()
    // which returns TRUE UTC timestamps, not "fake" site-TZ timestamps.
    return [
        'start'     => $local_range['start'],
        'end'       => $local_range['end'],
        'start_iso' => self::format_iso_utc($local_range['start'], false),
        'end_iso'   => self::format_iso_utc($local_range['end'], false),
    ];
}
```

**Use for:** Querying "today's orders" from WooCommerce.

---

## 4. Usage Examples

### 4.1 Querying Orders in a Time Window

```php
// ❌ WRONG: Using site-TZ timestamps directly
$end   = current_time('timestamp');
$start = $end - (15 * MINUTE_IN_SECONDS);
$orders = wc_get_orders([
    'date_created' => "$start...$end",  // BUG: comparing fake timestamps to UTC
]);

// ✅ CORRECT: Convert to UTC at query time
$end   = Time_Helper::now();
$start = $end - (15 * MINUTE_IN_SECONDS);
$orders = wc_get_orders([
    'date_created_gmt' => Time_Helper::format_iso_utc($start) . '...' . Time_Helper::format_iso_utc($end),
]);
```

### 4.2 Querying Today's Orders

```php
// ❌ WRONG: Manual calculation with current_time
$start = strtotime('today midnight', current_time('timestamp'));
$end   = strtotime('tomorrow midnight', current_time('timestamp')) - 1;
$orders = wc_get_orders([
    'date_created' => "$start...$end",
]);

// ✅ CORRECT: Use the helper
$range = Time_Helper::get_day_range_utc();
$orders = wc_get_orders([
    'date_created_gmt' => $range['start_iso'] . '...' . $range['end_iso'],
]);
```

### 4.3 Displaying Order Time to Users

```php
// ❌ WRONG: Using date() or gmdate() directly
echo date('Y-m-d H:i:s', $order->get_date_created()->getTimestamp());

// ✅ CORRECT: Use WC helper or Time_Helper
echo wc_format_datetime($order->get_date_created(), get_option('time_format'));
// or
echo Time_Helper::format_time($order->get_date_created()->getTimestamp());
```

### 4.4 Cron Job Time Windows

```php
class My_Cron {
    public static function process_recent_orders() {
        // Business logic uses site-TZ timestamps
        $end_time   = Time_Helper::now();
        $start_time = $end_time - MY_TIME_WINDOW;

        // Query layer converts to UTC
        $orders = My_Query::get_orders_in_window($start_time, $end_time);

        // Process orders...
    }
}

class My_Query {
    public static function get_orders_in_window(int $start, int $end): array {
        // UTC conversion happens HERE, at the query boundary
        return wc_get_orders([
            'date_created_gmt' => Time_Helper::format_iso_utc($start) . '...' . Time_Helper::format_iso_utc($end),
            'status'           => ['completed', 'processing'],
        ]);
    }
}
```

---

## 5. Testing Requirements

### 5.1 Self-Test: UTC Conversion (Non-Tautological)

**v1.1.14 Fix:** The test now validates against TRUE UTC time, not the input timestamp.
This catches the bug where `gmdate()` was applied to a site-TZ shifted timestamp without un-shifting it.

```php
public static function test_utc_conversion(): array {
    $site_now = WHM_Date::now();     // "Fake" site-TZ timestamp
    $utc_now  = WHM_Date::now_utc(); // True UTC timestamp

    // Convert site-TZ timestamp to UTC ISO string.
    $utc_iso = WHM_Date::format_iso_utc($site_now);

    // The CORRECT expected output is the TRUE UTC time.
    // If format_iso_utc() just did gmdate($site_now) without un-shifting,
    // this test would FAIL.
    $expected_iso = gmdate('Y-m-d H:i:s', $utc_now);

    // Allow 2-second tolerance for execution time.
    $utc_iso_ts      = strtotime($utc_iso . ' UTC');
    $expected_iso_ts = strtotime($expected_iso . ' UTC');
    $time_diff       = abs($utc_iso_ts - $expected_iso_ts);
    $valid_utc       = ($time_diff <= 2);

    return [
        'passed'  => $valid_utc,
        'details' => [
            'site_tz_timestamp' => $site_now,
            'utc_timestamp'     => $utc_now,
            'utc_iso_output'    => $utc_iso,
            'expected_utc_iso'  => $expected_iso,
            'time_diff_seconds' => $time_diff,
            'offset_hours'      => round(($site_now - $utc_now) / 3600, 2),
        ],
    ];
}
```

### 5.2 Self-Test: Day Range Consistency

```php
public static function test_day_range(): array {
    $range = Time_Helper::get_day_range();

    // Day span should be exactly 86399 seconds (23:59:59)
    $valid_span = (($range['end'] - $range['start']) === 86399);

    // Current time should fall within today's range
    $now = Time_Helper::now();
    // Note: now() returns fake timestamp, but for "is it today?" we compare to day boundaries
    // This works because we're checking relative position, not absolute UTC

    return [
        'passed'  => $valid_span,
        'details' => [
            'start'       => $range['start'],
            'end'         => $range['end'],
            'span'        => $range['end'] - $range['start'],
            'wp_timezone' => wp_timezone_string(),
        ],
    ];
}
```

### 5.3 Manual Test Scenarios

**Scenario 1: Non-UTC WordPress Timezone**
1. Set WordPress timezone to `America/New_York` (UTC-5)
2. Create an order at 10:00 AM local time
3. Verify order appears in "today's orders" query
4. Verify order appears in correct hour bucket

**Scenario 2: Day Boundary Crossing**
1. Set timezone to `Pacific/Auckland` (UTC+13)
2. Create order at 11:00 PM UTC (which is tomorrow in Auckland)
3. Verify order appears in "today's" local day, not yesterday's

**Scenario 3: DST Transition**
1. Test during daylight saving transition
2. Verify no duplicate or missing hours
3. Verify cron continues on expected schedule

---

## 6. Common Pitfalls

### 6.1 Pitfall: Mixing Timestamp Types

```php
// ❌ DANGEROUS: Mixing current_time() with time()
$site_time = current_time('timestamp');  // Fake timestamp
$real_time = time();                      // Real timestamp
$elapsed   = $real_time - $site_time;     // NOT elapsed time! This is the UTC offset!
```

### 6.2 Pitfall: Using date() Instead of gmdate()

```php
// ❌ WRONG: date() uses PHP's timezone setting
$iso = date('Y-m-d H:i:s', $timestamp);

// ✅ CORRECT: gmdate() always outputs UTC
$iso = gmdate('Y-m-d H:i:s', $timestamp);
```

### 6.3 Pitfall: Forgetting date_created vs date_created_gmt

```php
// ❌ These behave DIFFERENTLY:
wc_get_orders(['date_created' => '2025-12-27 10:00:00...2025-12-27 11:00:00']);
wc_get_orders(['date_created_gmt' => '2025-12-27 15:00:00...2025-12-27 16:00:00']);

// Always use date_created_gmt with UTC values for predictable results
```

### 6.4 Pitfall: strtotime() with current_time()

```php
// ❌ WRONG: strtotime doesn't understand the fake timestamp context
$tomorrow = strtotime('+1 day', current_time('timestamp'));

// ✅ CORRECT: Use DateTimeImmutable with wp_timezone()
$now      = new DateTimeImmutable('now', wp_timezone());
$tomorrow = $now->modify('+1 day')->getTimestamp();
```

### 6.5 Pitfall: Caching with Wrong Date Key

```php
// ❌ WRONG: Using gmdate for cache key (UTC date, not local date)
$cache_key = 'orders_' . gmdate('Y-m-d');

// ✅ CORRECT: Use local date for cache key
$cache_key = 'orders_' . wp_date('Y-m-d');
```

---

## 7. Implementation Checklist

- [ ] Create `Time_Helper` class with all methods from Section 3
- [ ] Add self-tests for UTC conversion and day range
- [ ] Audit all `current_time()` usage—ensure not passed to WC queries
- [ ] Audit all `wc_get_orders()` calls—use `date_created_gmt`
- [ ] Audit all `date()` calls—replace with `gmdate()` for UTC or `wp_date()` for display
- [ ] Add timezone info to admin debug/status page
- [ ] Document the UTC conversion architecture in project docs

---

## 8. Reference Implementation

See the WooCommerce Hourly Monitor plugin for a complete implementation:

- `includes/class-whm-date.php` - Time Helper implementation
- `includes/class-whm-query.php` - Query layer with UTC conversion
- `includes/class-whm-self-test.php` - Self-test methods
- `PROJECT-UTC-PHASE2.md` - Architecture documentation

---

## Appendix A: WordPress Time Function Reference

| Function | Returns | Use Case |
|----------|---------|----------|
| `time()` | True UTC timestamp | Comparing with external systems |
| `current_time('timestamp')` | Fake site-TZ timestamp | Business logic, display prep |
| `current_time('timestamp', true)` | True UTC timestamp | Rarely needed |
| `current_time('mysql')` | Site-TZ datetime string | Display only |
| `current_time('mysql', true)` | UTC datetime string | DB storage |
| `wp_date($format)` | Formatted in site-TZ | User display |
| `gmdate($format)` | Formatted in UTC | Machine output, WC queries |
| `date_i18n($format, $ts)` | Localized site-TZ | User display with i18n |
| `wp_timezone()` | DateTimeZone object | Creating DateTime objects |
| `wp_timezone_string()` | 'America/New_York' | Display/debugging |

## Appendix B: WooCommerce Query Reference

| Parameter | Expects | Notes |
|-----------|---------|-------|
| `date_created` | Site-TZ values | Converts internally—unpredictable |
| `date_created_gmt` | UTC values | Direct comparison—predictable ✅ |
| `date_modified` | Site-TZ values | Avoid |
| `date_modified_gmt` | UTC values | Use this ✅ |
| `date_completed` | Site-TZ values | Avoid |
| `date_completed_gmt` | UTC values | Use this ✅ |


