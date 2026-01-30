# Lovable LLM Framework — Guardrails for Clean Architecture

_Version: 1.0.0 — Created: 2026-01-30_

## 📌 TL;DR — The 5 Commandments

1. **🔍 SEARCH FIRST** — Check `src/lib/` for existing functions before writing new ones
2. **🏛️ 3-LAYER RULE** — Components render, Hooks manage state, `lib/` contains logic
3. **✅ VALIDATE EVERYTHING** — All inputs through `src/lib/validation.ts`, all outputs sanitized
4. **🔒 SINGLE OWNER** — Every piece of state has exactly one owner (context/hook)
5. **📏 BOUND EVERYTHING** — All queries have LIMIT, all loops have max iterations, all API calls have timeouts

**If you violate these, you create technical debt. Period.**

---

## 🎯 Purpose

This document provides **mandatory guardrails** for Lovable LLM to prevent spaghetti code and ensure architectural consistency from day one. Lovable is known to create "hornets nest" code when not constrained. **Follow these rules strictly.**

---

## ⚡ Quick Start — Read This First

**Before you write ANY code, read these 3 rules:**

1. **🔍 Search First, Code Second**
   - ALWAYS search `src/lib/` for existing functions before creating new ones
   - Use existing hooks in `src/hooks/` before creating new state management
   - Check `src/types/` for existing DTOs before defining new interfaces

2. **🏛️ Architecture in 3 Layers**
   - **Layer 1 (Components):** UI only — no business logic, no database calls
   - **Layer 2 (Hooks/Contexts):** State management — read/write through Layer 3
   - **Layer 3 (lib/):** Business logic — validation, data access, calculations

3. **✅ The Golden Rule**
   - If you're about to copy-paste code → Extract to `src/lib/` instead
   - If you're about to add business logic to a component → Move to `src/lib/` instead
   - If you're about to make a database call in a component → Use a hook instead

**🚨 STOP if you see:**
- Same code in 2+ places
- Business logic in a component
- Direct Supabase calls outside `src/lib/database.ts`
- Unvalidated user input
- Unbounded database queries

---

## 🚨 Critical Rules — Non-Negotiable

### 1. **NO Duplicate Logic — DRY is Law**

- **Before writing ANY function**, search the codebase for existing implementations
- **Reuse existing helpers** in `src/lib/` before creating new ones
- **Centralize all business logic** in `src/lib/` modules, NOT in components or pages
- **One function, one location** — if you need the same logic twice, extract it to a shared helper

**Existing Centralized Modules (USE THESE):**
- `src/lib/database.ts` — All Supabase data access
- `src/lib/beaver-builder.ts` — Beaver Builder JSON generation
- `src/lib/validation.ts` — Input validation and sanitization
- `src/lib/layout-fsm.ts` — Layout state transitions
- `src/lib/navigation.ts` — Navigation/footer data contracts
- `src/lib/fetch-utils.ts` — Network calls with timeouts
- `src/lib/errors.ts` — Error handling patterns
- `src/lib/query-limits.ts` — Pagination and query bounds

**Anti-Pattern Example (NEVER DO THIS):**
```tsx
// ❌ BAD: Inline validation in component
const handleSubmit = (data) => {
  if (!data.name || data.name.length > 100) return;
  // ... more validation
}
```

**Correct Pattern:**
```tsx
// ✅ GOOD: Use centralized validation
import { validateLayoutName } from '@/lib/validation';

const handleSubmit = (data) => {
  const validated = validateLayoutName(data.name);
  // ...
}
```

---

### 2. **State Management — Single Owner Rule**

- **Every piece of state has EXACTLY ONE owner**
- **Use existing contexts** before creating new ones
- **Never mutate state from multiple locations**
- **Use FSM for status transitions** (see `src/lib/layout-fsm.ts`)

**Existing State Owners (USE THESE):**
- `src/contexts/NavigationContext.tsx` — Nav/footer/CMS data
- `src/contexts/ColorThemeContext.tsx` — Theme state
- `src/hooks/useAuth.tsx` — Authentication state
- `src/hooks/useLayouts.tsx` — Layout CRUD operations
- `src/hooks/useColorPalettes.tsx` — Palette CRUD operations
- `src/hooks/useApiTokens.tsx` — API token management

**Rules:**
- Components **read** from contexts/hooks
- Components **dispatch actions** to contexts/hooks
- Components **NEVER** directly mutate database or global state
- Use `layout-fsm.ts` for ALL layout status changes

---

### 3. **Security — Validate Everything**

- **ALL user input MUST be validated** using `src/lib/validation.ts`
- **ALL AI-generated content MUST be sanitized** before rendering
- **ALL database queries MUST use parameterized queries** (Supabase handles this)
- **ALL URLs MUST be sanitized** (allow only `/`, `#`, `https://`)

**Mandatory Validation Points:**
1. Form inputs → validate before state update
2. AI responses → validate before persistence
3. Database reads → validate shape with Zod schemas
4. URL parameters → validate before use

**Example:**
```tsx
// ✅ ALWAYS validate AI responses
import { validateLayoutPayload } from '@/lib/validation';

const aiResponse = await generateLayout(prompt);
const validated = validateLayoutPayload(aiResponse); // Throws if invalid
await saveLayout(validated);
```

---

### 4. **Performance — Bound Everything**

- **ALL database queries MUST have LIMIT clauses** (use `src/lib/query-limits.ts`)
- **ALL loops MUST have maximum iteration counts**
- **ALL API calls MUST have timeouts** (use `src/lib/fetch-utils.ts`)
- **ALL arrays from user/AI MUST be clamped** (max 10 items unless specified)

**Use Existing Limits:**
```tsx
import { QUERY_LIMITS } from '@/lib/query-limits';

// ✅ GOOD: Bounded query
const layouts = await supabase
  .from('layouts')
  .select('*')
  .limit(QUERY_LIMITS.LAYOUTS_LIST);
```

---

### 5. **Component Architecture — Separation of Concerns**

**Components MUST:**
- Be **presentational only** (render UI, handle user events)
- **Delegate business logic** to `src/lib/` modules
- **Read state** from contexts/hooks
- **Dispatch actions** to contexts/hooks
- **Stay under 300 lines** (split if larger)

**Components MUST NOT:**
- Contain business logic (calculations, validation, formatting)
- Make direct database calls
- Contain duplicate code
- Mix data fetching with rendering

**File Structure Rules:**
```
src/
├── components/        # Presentational components ONLY
├── pages/            # Page-level components (routing)
├── hooks/            # State management hooks
├── contexts/         # Global state providers
├── lib/              # ALL business logic goes here
├── types/            # TypeScript interfaces/types
└── integrations/     # External service configs
```

---

### 6. **Data Transfer Objects (DTOs) — Strict Shapes**

- **ALL data crossing boundaries MUST use defined DTOs**
- **DTOs live in `src/types/`**
- **Validate DTOs with Zod schemas** in `src/lib/validation.ts`

**Existing DTOs (USE THESE):**
```tsx
// src/types/navigation.ts
export interface NavItem { id, label, href, visibleWhen, order }
export interface FooterSection { id, title, order, links }
export interface SiteMeta { siteName, logoUrl, supportEmail, copyright }

// src/types/index.ts
export interface Layout { id, user_id, name, type, status, content, ... }
export interface ColorPalette { id, name, colors, is_default, ... }
```

**Rule:** If data shape is used in 2+ places, it MUST be a DTO in `src/types/`.



---

### 7. **Error Handling — Fail Gracefully**

- **ALL async operations MUST have try/catch**
- **ALL errors MUST be logged** with context
- **ALL user-facing errors MUST be friendly messages**
- **Use existing error utilities** in `src/lib/errors.ts`

**Pattern:**
```tsx
import { handleError } from '@/lib/errors';

try {
  await riskyOperation();
} catch (error) {
  handleError(error, { context: 'LayoutCreation', userId });
  toast.error('Failed to create layout. Please try again.');
}
```

---

### 8. **FSM for State Transitions — No Direct Status Changes**

- **ALL layout status changes MUST go through `layout-fsm.ts`**
- **NEVER set status directly** (e.g., `layout.status = 'published'`)
- **Use transition functions** that validate allowed state changes

**Existing FSM (USE THIS):**
```tsx
import { transitionLayoutStatus } from '@/lib/layout-fsm';

// ✅ GOOD: Use FSM
const newStatus = transitionLayoutStatus(currentStatus, 'publish');

// ❌ BAD: Direct mutation
layout.status = 'published'; // NEVER DO THIS
```

**FSM Scoring (from AGENTS.md):**
- Layout entity has 3+ states (draft, active, archived) → **Use FSM**
- Transitions have rules (not all states can reach all others) → **Use FSM**
- Status changes are auditable → **Use FSM**

---

## 📋 Pre-Code Checklist

Before writing ANY code, answer these questions:

1. ✅ **Does this logic already exist?** → Search `src/lib/` first
2. ✅ **Where does this state live?** → Use existing context/hook or justify new one
3. ✅ **Is this input validated?** → Use `src/lib/validation.ts`
4. ✅ **Is this query bounded?** → Use `src/lib/query-limits.ts`
5. ✅ **Is this a component or business logic?** → Components render, `lib/` thinks
6. ✅ **Does this need a DTO?** → If data crosses boundaries, yes
7. ✅ **Is error handling in place?** → All async ops need try/catch
8. ✅ **Does this change state?** → Use FSM if applicable

---

## 🚫 Forbidden Patterns

| ❌ NEVER DO THIS | ✅ DO THIS INSTEAD |
|------------------|-------------------|
| Inline validation in components | Use `src/lib/validation.ts` |
| Direct Supabase calls in components | Use `src/lib/database.ts` or hooks |
| Duplicate helper functions | Extract to `src/lib/` |
| Unbounded queries | Use `QUERY_LIMITS` |
| Magic strings for status | Use `layout-fsm.ts` enums |
| Business logic in components | Move to `src/lib/` |
| Multiple state owners | Single context per domain |
| Unvalidated AI responses | Validate with Zod before use |
| Direct status mutations | Use FSM transitions |
| Scattered error handling | Use `src/lib/errors.ts` |
| Hardcoded limits | Use `src/lib/query-limits.ts` |
| Inline fetch calls | Use `src/lib/fetch-utils.ts` |

---

## 🏗️ Architecture Patterns to Follow

**Visual Architecture:** See the Mermaid diagram "Layout Builder Architecture - Data Flow" for a visual representation of the 3-layer architecture.

### Pattern 1: Data Flow (Read Operations)
```
User Action → Component Event Handler → Hook/Context → lib/database.ts → Supabase → Validation → State Update → Re-render
```

### Pattern 2: Data Flow (Write Operations)
```
User Input → Component → Validation (lib/validation.ts) → Hook/Context → lib/database.ts → Supabase → State Update → UI Feedback
```

### Pattern 3: AI Integration
```
User Prompt → Component → Hook → Edge Function → AI Gateway → Validation (lib/validation.ts) → Persistence → State Update
```

### Pattern 4: State Transitions
```
Current State → User Action → FSM Validation (lib/layout-fsm.ts) → Allowed Transition → Database Update → State Update
```

### Key Principles:
- **Unidirectional data flow** — Data flows down, events flow up
- **Single source of truth** — State lives in one place
- **Validation at boundaries** — Validate on entry, trust internally
- **Fail fast in dev, graceful in prod** — Errors surface immediately during development

---

## 📦 Tech Stack Reference

- **Frontend:** React 18 + TypeScript + Vite
- **UI:** shadcn/ui + Radix UI + Tailwind CSS
- **State:** React Context + TanStack Query
- **Backend:** Supabase (PostgreSQL + Edge Functions)
- **Validation:** Zod
- **Testing:** Vitest + Testing Library
- **Routing:** React Router v6

---

## 🎯 Common Tasks — Quick Reference

### Adding a New Feature
1. Define DTOs in `src/types/`
2. Create validation schemas in `src/lib/validation.ts`
3. Add business logic to `src/lib/` (new file or existing)
4. Create/update hook in `src/hooks/` for state management
5. Create presentational component in `src/components/`
6. Wire up in page component in `src/pages/`

### Adding a New Database Table
1. Create Supabase migration
2. Define DTO in `src/types/`
3. Add CRUD functions to `src/lib/database.ts`
4. Create hook in `src/hooks/` for state management
5. Add validation in `src/lib/validation.ts`
6. Add query limits in `src/lib/query-limits.ts`

### Adding a New API Integration
1. Add fetch wrapper in `src/lib/fetch-utils.ts`
2. Define response DTO in `src/types/`
3. Add validation in `src/lib/validation.ts`
4. Create hook for state management
5. Add error handling with `src/lib/errors.ts`

---

## 🎓 Learning Resources

- **Architecture Principles:** See `AGENTS.md` (v3.0.0)
- **Audit Findings:** See `PROJECT/2-WORKING/P1-AUDIT-CODEX.md`
- **Navigation Patterns:** See `PROJECT/2-WORKING/P1-CODEX-NAVIGATION.md`
- **FSM Scoring:** See AGENTS.md Section 2 (When to Use FSM)

---

## ⚠️ Red Flags — Stop and Refactor

If you see ANY of these, STOP and refactor immediately:

1. **Same code in 3+ places** → Extract to `src/lib/`
2. **Component >300 lines** → Split into smaller components
3. **Business logic in component** → Move to `src/lib/`
4. **Direct database calls in component** → Use hook
5. **Unbounded query** → Add LIMIT clause
6. **No error handling** → Add try/catch
7. **Unvalidated input** → Add validation
8. **Magic strings** → Use constants/enums
9. **Multiple state owners** → Consolidate to single context
10. **Direct status mutation** → Use FSM

---

## 🔒 Security Checklist

Before deploying ANY feature:

- [ ] All user inputs validated with Zod schemas
- [ ] All AI responses sanitized before rendering
- [ ] All URLs sanitized (only `/`, `#`, `https://` allowed)
- [ ] All database queries use parameterized queries (Supabase default)
- [ ] No sensitive data in logs or error messages
- [ ] RLS policies enabled on all Supabase tables
- [ ] Auth checks in place for protected routes
- [ ] CORS configured correctly for edge functions

---

## 📊 Complexity Thresholds (from AGENTS.md)

| Metric | Acceptable | Warning | Refactor Required |
|--------|------------|---------|-------------------|
| Lines per file | <300 | 300-500 | >500 |
| Functions per file | <10 | 10-15 | >15 |
| Parameters per function | <4 | 4-5 | >5 |
| Nesting depth | <3 | 3-4 | >4 |
| Duplicate code blocks | 0 | 1-2 | >2 |

---

## 🎬 Final Checklist Before Committing

- [ ] No duplicate logic (searched `src/lib/` first)
- [ ] State has single owner (used existing context/hook)
- [ ] All inputs validated (used `src/lib/validation.ts`)
- [ ] All queries bounded (used `QUERY_LIMITS`)
- [ ] Component is presentational only (no business logic)
- [ ] DTOs defined for data crossing boundaries
- [ ] Error handling in place (try/catch + logging)
- [ ] FSM used for state transitions (if applicable)
- [ ] No security vulnerabilities (validated inputs, sanitized outputs)
- [ ] Code is under complexity thresholds
- [ ] Tests added for new functionality (if applicable)

---

_This framework is mandatory for all Lovable LLM interactions. Violations will result in technical debt and refactoring overhead. When in doubt, refer to AGENTS.md for detailed architectural guidance._

**Version History:**
- v1.0.0 (2026-01-30): Initial framework based on AGENTS.md v3.0.0 and audit findings
