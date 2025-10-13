## Test Coverage Best Practices

- **Comprehensive Coverage**: All business logic must be covered by tests. This is non-negotiable. If code contains business logic, it must have corresponding tests.
- **Test-Driven Development**: When implementing new features, write tests first. When fixing bugs, write a failing test that demonstrates the bug, then fix the code.
- **Pest PHP**: All tests must be written using Pest PHP syntax and conventions.
- **Test Behavior, Not Implementation**: Focus tests on what the code does from the user's perspective, not how it does it internally.
- **Clear Test Names**: Use descriptive test names that read like natural language sentences explaining the expected behavior.
- **Avoid Unnecessary Mocking**: Laravel facades provide testing affordances (like `fake()`) that should always be used instead of mocking. External APIs should only be used via SDKs that provide testing affordances. Mocking should be a last resort, not a default approach.

## Smoke Test All Routes

All routes except for Filament admin panels must have a smoke test using PestPHP's browser testing feature (https://pestphp.com/docs/browser-testing) that should contain the following assertions:

```php
    test('smoke test', function () {
    $page = visit('/some-route');
        
    $page->assertNoSmoke()
        ->assertNoAccessibilityIssues()
        ->assertNoConsoleLogs()
        ->assertNoJavaScriptErrors();
        
    $page = visit('/some-route')->on()->mobile();
        
    $page->assertNoSmoke()
        ->assertNoAccessibilityIssues()
        ->assertNoConsoleLogs()
        ->assertNoJavaScriptErrors();
        
    $page = visit('/some-route')->inDarkMode()
        
    $page->assertNoSmoke()
        ->assertNoAccessibilityIssues()
        ->assertNoConsoleLogs()
        ->assertNoJavaScriptErrors();
});
```

## Browser Testing for User-Facing Pages

Write browser tests for all expected user interactions with the application. Do not write browser tests for any admin-facing pages.

## Laravel Testing Affordances

Laravel provides powerful testing tools that eliminate the need for manual mocking:

### Facade Fakes
Always use Laravel's built-in fake methods instead of mocking:
- `Mail::fake()` - Test email sending
- `Queue::fake()` - Test queued jobs
- `Event::fake()` - Test event dispatching
- `Notification::fake()` - Test notifications
- `Bus::fake()` - Test command bus
- `Storage::fake()` - Test file operations
- `Http::fake()` - Test HTTP client requests

### Database Testing
- Use Laravel's database transactions or `RefreshDatabase` trait
- Leverage factories for test data creation
- Use database assertions: `assertDatabaseHas()`, `assertDatabaseMissing()`

### HTTP Testing
- Use Pest's HTTP testing methods: `get()`, `post()`, `put()`, `patch()`, `delete()`
- Assert responses: `assertStatus()`, `assertJson()`, `assertRedirect()`
- Test authentication: `actingAs($user)`

## External API Testing

When integrating with external APIs:

1. **Use SDKs with Testing Support**: Only integrate external services via SDKs that provide testing/mocking capabilities
2. **Fake at the SDK Level**: Use the SDK's built-in testing features rather than mocking HTTP calls
3. **Document SDK Testing Patterns**: When adding a new external service, document its testing approach in the spec

## Test Organization

### Test Structure
```php
it('creates a new user with valid data', function () {
    // Arrange - Set up test data and state
    $userData = [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ];

    // Act - Perform the action being tested
    $user = User::create($userData);

    // Assert - Verify the expected outcome
    expect($user->name)->toBe('John Doe')
        ->and($user->email)->toBe('john@example.com');

    $this->assertDatabaseHas('users', $userData);
});
```

### Test Categories

**Unit Tests**
- Test individual methods and functions
- Focus on business logic in models, actions, and services
- Fast execution (milliseconds)
- Does not boot the Laravel framework

**Feature Tests**
- Test complete user workflows
- Test HTTP endpoints and their responses
- Test interactions between multiple components
- Verify database state changes
- Boots the Laravel framework

**Integration Tests**
- Test interactions with external services (using SDK fakes)
- Test complex multi-step processes
- Verify system behavior end-to-end

## What to Test

### Always Test
- Business logic in models, actions, and services
- API endpoints and their responses
- Authorization and permission checks
- Data validation rules
- Database relationships and constraints
- Email/notification sending (using fakes)
- Job dispatching and processing (using fakes)
- File uploads and storage operations (using fakes)

### Test Thoroughly
- Edge cases in business logic
- Error handling and failure scenarios
- Boundary conditions
- State transitions

### Testing Philosophy

The goal is not just code coverage, but confidence:
- Every test should verify actual user-facing behavior
- Tests should fail when the behavior breaks
- Tests should pass when the behavior is correct
- Tests should be maintainable as the codebase evolves

## Performance

- Unit tests should run in milliseconds
- Feature tests should complete in seconds
- Use database transactions to keep tests fast
- Leverage Laravel's testing optimizations
- Run the full test suite frequently to catch regressions early

## Test Quality Standards

All tests must:
- Have clear, descriptive names
- Follow the Arrange-Act-Assert pattern
- Test one concept per test
- Be independent (not rely on other tests)
- Be repeatable (produce same results every time)
- Clean up after themselves (via transactions or teardown)
