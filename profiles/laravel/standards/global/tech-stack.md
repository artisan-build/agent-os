## Tech stack

Define your technical stack below. This serves as a reference for all team members and helps maintain consistency across the project.

### Framework & Runtime
- **Application Framework:** Laravel (Latest Version)
- **Language/Runtime:** PHP (Latest Stable Version)
- **Package Manager:** composer

### Frontend
- **JavaScript Framework:** Apline
- **CSS Framework:** Tailwind CSS
- **UI Components:** FluxUI Pro (https://fluxui.dev)

### Database & Storage
- **Database:** PostgreSQL
- **ORM/Query Builder:** Eloquent
- **Caching:** PostgresSQL Unlogged Table via Laravel Cache Facade

### Testing & Quality
- **Test Framework:** Pest
- **Linting/Formatting:** Pint
- **Static Analysis:** PHPStan (Level 6) with Larastan
- **Automatic Refactoring:** Rector with driftingly/rector-laravel
- **Full Suite:** `php artisan ready` runs all tools and should run cleanly without errors before committing any code

### Deployment & Infrastructure
- **Hosting:** Laravel Forge
- **CI/CD:** GitHub Actions
