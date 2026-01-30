# Lovable LLM Quick Reference Cheat Sheet

_Companion to LOVABLE-STARTER.md — Keep this open while coding_

---

## 🚦 Before Writing ANY Code

```
1. Search src/lib/ for existing functions
2. Check src/hooks/ for existing state management
3. Verify src/types/ for existing DTOs
4. Read the relevant section in LOVABLE-STARTER.md
```

---

## 📂 File Organization — Where Does This Go?

| What You're Adding | Where It Goes | Example |
|-------------------|---------------|---------|
| Business logic | `src/lib/*.ts` | Validation, calculations, formatting |
| Data access | `src/lib/database.ts` | Supabase queries |
| State management | `src/hooks/*.tsx` | CRUD operations, data fetching |
| Global state | `src/contexts/*.tsx` | Auth, navigation, theme |
| UI component | `src/components/*.tsx` | Buttons, forms, cards |
| Page | `src/pages/*.tsx` | Route-level components |
| Type/Interface | `src/types/*.ts` | DTOs, interfaces |
| Validation schema | `src/lib/validation.ts` | Zod schemas |

---

## 🔧 Existing Modules — USE THESE

### Core Business Logic (`src/lib/`)
- `database.ts` — All Supabase data access
- `validation.ts` — Input validation, sanitization
- `layout-fsm.ts` — Layout state transitions
- `beaver-builder.ts` — BB JSON generation
- `navigation.ts` — Nav/footer data contracts
- `fetch-utils.ts` — Network calls with timeouts
- `errors.ts` — Error handling patterns
- `query-limits.ts` — Pagination constants

### State Management (`src/hooks/`)
- `useAuth.tsx` — Authentication state
- `useLayouts.tsx` — Layout CRUD
- `useColorPalettes.tsx` — Palette CRUD
- `useApiTokens.tsx` — API token management

### Global Contexts (`src/contexts/`)
- `NavigationContext.tsx` — Nav/footer/CMS
- `ColorThemeContext.tsx` — Theme state

---

## ✅ Quick Validation Checklist

```tsx
// ❌ BAD: Inline validation
if (!name || name.length > 100) return;

// ✅ GOOD: Use centralized validation
import { validateLayoutName } from '@/lib/validation';
const validated = validateLayoutName(name);
```

**Always validate:**
- [ ] User form inputs
- [ ] AI-generated responses
- [ ] Database query results
- [ ] URL parameters

---

## 🔒 Quick Security Checklist

- [ ] Input validated with Zod schema
- [ ] Output sanitized (HTML escaped)
- [ ] URLs sanitized (only `/`, `#`, `https://`)
- [ ] Database queries parameterized (Supabase default)
- [ ] No sensitive data in logs

---

## 📏 Quick Performance Checklist

```tsx
// ❌ BAD: Unbounded query
const layouts = await supabase.from('layouts').select('*');

// ✅ GOOD: Bounded query
import { QUERY_LIMITS } from '@/lib/query-limits';
const layouts = await supabase
  .from('layouts')
  .select('*')
  .limit(QUERY_LIMITS.LAYOUTS_LIST);
```

**Always bound:**
- [ ] Database queries (use `QUERY_LIMITS`)
- [ ] Loops (max iterations)
- [ ] API calls (timeouts via `fetch-utils.ts`)
- [ ] Arrays from user/AI (clamp to max 10)

---

## 🎯 Quick Component Checklist

**Components should:**
- [ ] Be under 300 lines
- [ ] Only render UI
- [ ] Read from hooks/contexts
- [ ] Dispatch actions to hooks/contexts
- [ ] Have NO business logic

**Components should NOT:**
- [ ] Make database calls
- [ ] Contain validation logic
- [ ] Contain calculations
- [ ] Duplicate code from other components

---

## 🔄 Quick FSM Checklist

```tsx
// ❌ BAD: Direct status mutation
layout.status = 'published';

// ✅ GOOD: Use FSM
import { transitionLayoutStatus } from '@/lib/layout-fsm';
const newStatus = transitionLayoutStatus(currentStatus, 'publish');
```

**Use FSM when:**
- [ ] Entity has 3+ states
- [ ] Transitions have rules
- [ ] Status changes need audit trail

---

## 🚨 Red Flags — STOP Immediately

1. Same code in 2+ places → Extract to `src/lib/`
2. Component >300 lines → Split it
3. Business logic in component → Move to `src/lib/`
4. Direct Supabase call in component → Use hook
5. Unbounded query → Add LIMIT
6. No error handling → Add try/catch
7. Unvalidated input → Add validation
8. Magic strings → Use constants/enums
9. Multiple state owners → Consolidate
10. Direct status mutation → Use FSM

---

## 📋 Copy-Paste Templates

### Template: New Feature
```tsx
// 1. Define DTO in src/types/
export interface MyFeature {
  id: string;
  name: string;
  // ...
}

// 2. Add validation in src/lib/validation.ts
export const validateMyFeature = (data: unknown): MyFeature => {
  return myFeatureSchema.parse(data);
};

// 3. Add business logic in src/lib/my-feature.ts
export const processMyFeature = (data: MyFeature) => {
  // Business logic here
};

// 4. Add hook in src/hooks/useMyFeature.tsx
export const useMyFeature = () => {
  // State management here
};

// 5. Add component in src/components/MyFeature.tsx
export const MyFeature = () => {
  const { data } = useMyFeature();
  return <div>{/* UI here */}</div>;
};
```

### Template: Error Handling
```tsx
import { handleError } from '@/lib/errors';
import { toast } from '@/hooks/use-toast';

try {
  await riskyOperation();
  toast({ title: 'Success!' });
} catch (error) {
  handleError(error, { context: 'FeatureName', userId });
  toast({ title: 'Error', description: 'Please try again', variant: 'destructive' });
}
```

---

## 🎓 When in Doubt

1. Read `LOVABLE-STARTER.md` for detailed guidance
2. Check `AGENTS.md` for architecture principles
3. Review `P1-AUDIT-CODEX.md` for common pitfalls
4. Search existing code for similar patterns

---

_Keep this cheat sheet open while working with Lovable LLM_

