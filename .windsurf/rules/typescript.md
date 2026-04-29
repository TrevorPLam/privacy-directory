---
trigger: glob
globs: **/*.ts,**/*.tsx
---

<typescript_guidelines>

# TypeScript Guidelines

When writing or modifying TypeScript code:

## TypeScript 5.9 Compatibility

This project uses TypeScript 5.9 (released August 2025). Leverage new features when appropriate.

## Strict Mode Compliance

This project uses TypeScript strict mode. Follow these principles:

### Type Safety

- Never use `any` type - use `unknown` when type is truly unknown
- Explicitly handle null and undefined with type guards
- Use type narrowing patterns (if checks, type predicates)
- Define interfaces for all props and state objects

### Type Annotations

- Add type annotations to function parameters and return types
- Use `interface` for object shapes, `type` for unions/intersections
- Use generic types when appropriate for reusability
- Avoid type assertions (`as`) - use type guards instead

### Null Handling

```typescript
// Bad - unsafe
function getName(user: User): string {
  return user.name; // Could be null
}

// Good - safe
function getName(user: User): string | null {
  if (!user) return null;
  return user.name;
}

// Better - with type guard
function getName(user: User | null): string {
  if (!user) throw new Error("User not found");
  return user.name;
}
```

## TypeScript 5.9 New Features

### Prescriptive tsc --init Defaults

TypeScript 5.9 generates more prescriptive tsconfig.json by default:

- `moduleDetection: force` - treats all files as modules
- `target: esnext` - uses latest ECMAScript features
- `types: []` - limits automatic type loading
- `verbatimModuleSyntax: true` - strict module syntax
- `noUncheckedSideEffectImports: true` - validates side-effect imports
- `noUncheckedIndexedAccess: true` - safer array access
- `exactOptionalPropertyTypes: true` - stricter optional types

### import defer Support

- Use `import defer` for deferred module loading
- Module loads but execution deferred until first use
- Useful for code splitting and lazy loading

```typescript
import { heavyModule } from "./heavy";
// heavyModule loads but doesn't execute until used
```

### --module node20 Support

- Use `--module node20` for Node.js 20+ compatibility
- Better ES module support in Node.js
- Updated package.json exports resolution

### Performance Improvements

- Cache instantiations on mappers
- Avoid closure creation in file operations
- Faster type checking for large projects

## Path Aliases

Always use configured path aliases instead of relative imports:

```typescript
// Bad
import Button from "../../../components/Button.astro";

// Good
import Button from "@components/Button.astro";
```

Available aliases:

- `@/*` → `src/*`
- `@components/*` → `src/components/*`
- `@layouts/*` → `src/layouts/*`
- `@content/*` → `src/content/*`
- `@styles/*` → `src/styles/*`

## Type Definitions

### Environment Variables

- Define all environment variables in `src/env.d.ts`
- Use `VITE_` prefix for client-side variables
- Type definitions prevent runtime errors

### Component Props

- Use TypeScript interfaces for component props
- Make optional props explicit with `?`
- Provide default values for optional props

```typescript
interface Props {
  title: string;
  description?: string;
  variant?: "primary" | "secondary";
}

const { title, description = "", variant = "primary" } = Astro.props;
```

## Utility Types

Use TypeScript utility types for common patterns:

- `Partial<T>` - make all properties optional
- `Required<T>` - make all properties required
- `Readonly<T>` - make properties read-only
- `Pick<T, K>` - select specific properties
- `Omit<T, K>` - remove specific properties

## Error Handling

- Use `try/catch` blocks with proper error typing
- Use `unknown` for caught errors, not `any`
- Type guard errors before accessing properties

```typescript
try {
  // code
} catch (error) {
  if (error instanceof Error) {
    console.error(error.message);
  } else {
    console.error("Unknown error:", error);
  }
}
```

## Best Practices

- Enable strict mode in tsconfig.json
- Use `noUncheckedIndexedAccess` for safer array access (default in 5.9)
- Use `noImplicitReturns` to ensure all paths return
- Use `noUnusedLocals` and `noUnusedParameters` to catch dead code
- Use `exactOptionalPropertyTypes` for stricter optional types (default in 5.9)
- Use `moduleDetection: force` for consistent module handling (default in 5.9)
- Run `npx astro check` for type checking in Astro projects

## Common Patterns

### Type Guards

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}
```

### Discriminated Unions

```typescript
type Result = { success: true; data: Data } | { success: false; error: Error };
```

### Generic Functions

```typescript
function identity<T>(value: T): T {
  return value;
}
```

## tsconfig.json Recommendations (TypeScript 5.9)

When creating new projects, `tsc --init` now generates prescriptive defaults. For this project, ensure:

```json
{
  "compilerOptions": {
    "strict": true,
    "module": "nodenext",
    "target": "esnext",
    "moduleDetection": "force",
    "verbatimModuleSyntax": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedSideEffectImports": true
  }
}
```

</typescript_guidelines>
