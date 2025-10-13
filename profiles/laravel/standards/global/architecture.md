# Architecture

### Action Pattern

We use the Action Pattern to define the business logic of our application. Do not create service classes, repositories, or any other type of class that contains business logic.

**Class Location:** All Action classes must be in the app/Actions directory and be descriptively named. If they are specific to a feature, the name should be prefixed with the feature name. Do not create subdirectories for Action classes.

**Class Structure:** All Action classes must be invokable classes and only have an invoke() method. If you feel the need to add an additional method, then create a new Action class to perform that task and use dependency injection to add the dependency in the constructor as a property. Always use constructor property promotion.

**Tests:** All Action classes must have a corresponding test file in the tests/Actions directory.

### Livewire

**Full-Page Components:** Any route that sends HTML to the browser should use Livewire instead of a controller. Full-page components must be appended with the word "Page" in the name, eg: 'App\Livewire\HomePage'

**Partial Components:** Small, resuable components should only be created using Livewire if they require reactivity. If they are just used to render HTML, then use Blade Components instead.

### Blade Components

**Always Use Class-Based Components:** Even for simple components, always use class-based components. This consistency makes it easier to for us to find the component when we need it.

### Enums

**Location:** All Enums must be in the app/Enums directory.

**Naming:** All Enums should have a singular name, eg: `UserType` unless the Enum is integer backed where the integers are bitmasks, in which case it should be Plural: eg: `Sports`

**All Enums must be backed:** Because we often rely on the Symfony serializer, which cannot serialize unbacked Enums, all Enums must be backed. If there is no clear reason to back an Enum, then simply use the case name as its value, eg: `case Admin = 'Admin'`

**Use Enums Liberally:** We do not want any magic strings or integers in our code. If you find yourself using string or integer values in business logic, you probably need an Enum.

### Types

**Use Strict Typing:** All code should be written in strict typing. Only use docblocks for typing when required to clarify the type, such as the shape of an array.

**PHPStan Level 6**: We use PHPStan level 6 to enforce type safety.