## UI component best practices

### In Admin Panels

**Lean on FilamentPHP**: Filament provides all the UI components you should need to build all functionality within the admin panel. You should not need to create any new components here.

### In User-Facing Pages

**Lean on FluxUI**: If FluxUI offers a component that does what you need, use it. Sacrifice pixel perfection for simplicity and consistency.

**Avoid Inline Styles**: Use Tailwind CSS to accomplish what you need.

**Use Blade Components**: Use class-based Blade Components to encapsulate reusable UI elements.

**Do Not Define Colors**: The color palette is defined by FluxUI. Use Flux variants for components that require a specific look and feel.

**You are not a designer and we are not designers**: We lean heavily on FluxUI because it provides a clean, consistent design system.