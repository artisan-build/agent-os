## Coding style best practices

- **Small, Focused Functions**: Keep functions small and focused on a single task for better readability and testability
- **Remove Dead Code**: Delete unused code, commented-out blocks, and imports rather than leaving them as clutter
- **Backward compatability only when required:** Unless specifically instructed otherwise, assume you do not need to write additional code logic to handle backward compatability.
- **DRY Principle**: Avoid duplication by extracting common logic into reusable Action classes.
- **Low Complexity**: Do not perform any business logic within a loop. Iterate through the data and pass each item to an Action class.
- **Early Returns**: Use early returns to avoid if / else statements.
- **elseif is forbidden**: Never use elseif statements. Use an early return instead.

