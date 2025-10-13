## General development conventions

- **Consistent Project Structure**: Follow the Laravel project structure and conventions. When creating new files, use the `php artisan make:` commands to generate the boilerplate code, ensuring that our own stubs are used.
- **Clear Documentation**: Maintain up-to-date README files with setup instructions, architecture overview, and contribution guidelines. In particular, all commands should be documented with their signature, options, arguments, expected output, and (if applicable) the schedule on which they are set to run.
- **Version Control Best Practices**: Use clear commit messages, feature branches, and meaningful pull/merge requests with descriptions
- **Environment Configuration**: Use environment variables for configuration; never commit secrets or API keys to version control
- **Dependency Management**: Keep dependencies up-to-date and minimal; document why major dependencies are used
- **Code Review Process**: Establish a consistent code review process with clear expectations for reviewers and authors
- **Testing Requirements**: Define what level of testing is required before merging (unit tests, integration tests, etc.)
- **Feature Flags**: Use feature flags for incomplete features rather than long-lived feature branches
- **Changelog Maintenance**: Keep a changelog or release notes to track significant changes and improvements
