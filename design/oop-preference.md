# Object-Oriented Programming Preference

When the language supports OOP, prefer object-oriented design: behaviour belongs on the type that owns the data.

## Context

*Applies to:* Code in OOP-capable languages — Java, C#, TypeScript, JavaScript, Kotlin, Python, Ruby, etc.
*Level:* Tactical/Operational — applies on every design decision about where logic lives
*Audience:* All developers and AI agents

## Core Principles

1. *Behaviour goes with data:* If a function operates primarily on the fields of one type, it belongs as a method on that type — not as a free function that takes the type as an argument
2. *Encapsulation by default:* The type that owns the data is the natural place to expose operations on it; callers should not need to know the internal shape

## Rules

### Must Have (Critical)

- *RULE-001:* When designing logic in an OOP-capable language, default to placing methods on the class that owns the data they operate on. Free functions are appropriate only when the logic does not belong to any single type (e.g. genuinely cross-cutting utilities, pure transformations between unrelated types).
- *RULE-002:* If you find yourself writing a free function whose first or only argument is an instance of a class, stop and put it on the class instead. The signature `isExpired(course: Course): boolean` is a method on `Course`, not a standalone function.

### Should Have (Important)

- *RULE-101:* Convert plain data containers (interfaces / records / `type` aliases) into classes when they accumulate behaviour. A type that started as a DTO but now has three associated free functions is a class wearing a disguise.
- *RULE-102:* This preference does not override functional-first conventions in languages where they are idiomatic (Haskell, Elm, Clojure, Rust without `impl`). Apply OOP where the language and ecosystem already lean OOP.

## Patterns & Anti-Patterns

### ✅ Do This

```typescript
class Course {
  constructor(private readonly schedule: Schedule) {}

  isExpired(): boolean {
    const sevenDaysInMillis = 7 * 24 * 60 * 60 * 1000
    return new Date() >= new Date(this.schedule.startDate.getTime() - sevenDaysInMillis)
  }
}

// Caller
if (course.isExpired()) { ... }
```

### ❌ Don't Do This

```typescript
interface Course {
  schedule: Schedule
}

// Free function operating on a single Course — belongs on the class
function isCourseExpired(course: Course): boolean {
  const sevenDaysInMillis = 7 * 24 * 60 * 60 * 1000
  return new Date() >= new Date(course.schedule.startDate.getTime() - sevenDaysInMillis)
}

if (isCourseExpired(course)) { ... }
```

## Related Rules

- agents/ee-llm-toolkit/rules/clean-code.md — single responsibility and intent-revealing interfaces
- agents/ee-llm-toolkit/rules/solid-principles.md — SRP and encapsulation
- design/code-organisation.md — where logic lives in the call stack

---

## TL;DR

*Critical Rules:*
- In OOP-capable languages, methods on the owning class beat free functions on the owning type
- A free function whose first argument is `T` should be a method on `T`

*Quick Decision Guide:*
About to write `function doX(thing: Thing, ...)`? Make it `thing.doX(...)` instead.
