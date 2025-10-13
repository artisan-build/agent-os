# Naming

Favor long, descriptive names over short, generic ones.

**Good:** `class GetIpAddressesForAllActiveUserSessions`
**Bad:** `class GetIps`

**Class Names:** Use PascalCase, and except for full-page Livewire components which must be suffixed by `Page`, do not use a suffix to indicate the purpose of the class. We should know it from its location, eg `ImportFromGoogleSheets` is better than `ImportFromGoogleSheetsCommand`.
**Method Names:** Use camelCase
**Variable Names:** Use snake_case unless the value of the variable is a callable, in which case use camelCase
**Property Names:** Use snake_case
**Constants:** Use ALL_CAPS (Generally favor Enums over constants unless the constants are used exclusively within the defining class)