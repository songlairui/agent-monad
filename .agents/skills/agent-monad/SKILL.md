```markdown
# agent-monad Development Patterns

> Auto-generated skill from repository analysis

## Overview
This skill teaches you the core development patterns and conventions used in the `agent-monad` TypeScript codebase. You'll learn about file naming, import/export styles, commit message conventions, and testing patterns. This guide will help you contribute code that matches the project's established style and structure.

## Coding Conventions

### File Naming
- Use **camelCase** for file names.
  - Example: `agentMonad.ts`, `userSessionManager.ts`

### Import Style
- Use **relative imports** for referencing modules within the project.
  - Example:
    ```typescript
    import { agentMonad } from './agentMonad';
    ```

### Export Style
- Use **named exports** rather than default exports.
  - Example:
    ```typescript
    // agentMonad.ts
    export function agentMonad() { ... }
    ```

### Commit Messages
- Follow **conventional commit** style.
- Use the `feat` prefix for new features.
  - Example: `feat: add session management`
- Keep commit messages concise (average ~33 characters).

## Workflows

### Adding a New Feature
**Trigger:** When implementing a new capability or module.
**Command:** `/add-feature`

1. Create a new file using camelCase naming.
2. Write your code using named exports.
3. Use relative imports for dependencies.
4. Write corresponding tests in a `.test.ts` file.
5. Commit your changes with a `feat:` prefix.
   - Example: `feat: implement user session logic`

### Writing Tests
**Trigger:** When adding or updating functionality.
**Command:** `/write-test`

1. Create a test file with the same base name as the source file, ending with `.test.ts`.
   - Example: `agentMonad.test.ts`
2. Write tests using the project's preferred (unknown) testing framework.
3. Use relative imports to bring in the module under test.
   - Example:
     ```typescript
     import { agentMonad } from './agentMonad';
     ```
4. Run the test suite to ensure correctness.

## Testing Patterns

- Test files follow the pattern: `*.test.ts`
- Tests are colocated with or near the modules they test.
- Use relative imports in test files.
- The specific testing framework is not detected; follow existing test file patterns.

## Commands
| Command        | Purpose                                   |
|----------------|-------------------------------------------|
| /add-feature   | Start the process for adding a new feature|
| /write-test    | Begin writing tests for a module          |
```
