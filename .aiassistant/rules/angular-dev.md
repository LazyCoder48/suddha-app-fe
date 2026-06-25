---
apply: always
---

# Angular Development Rules

These rules apply to all Angular code in this project (Angular **21**, standalone,
signal-based, zoneless, modern `@angular/build` toolchain with Vitest).

## TypeScript

- Enable and respect strict type checking (`strict` is on in `tsconfig.json`).
- Prefer type inference when the type is obvious; do not add redundant annotations.
- Avoid the `any` type; use `unknown` when the type is genuinely uncertain and narrow it.
- Honor `noImplicitOverride`, `noImplicitReturns`, and `noFallthroughCasesInSwitch`.
- Because `noPropertyAccessFromIndexSignature` is enabled, access index-signature
  members with bracket notation (`obj['key']`), not dot notation.

## Components

- Always use **standalone** components, directives, and pipes. Do not set
  `standalone: true` explicitly — it is the default in Angular 21.
- Keep components small and focused on a single responsibility.
- Use `input()` and `output()` functions instead of the `@Input()` / `@Output()`
  decorators.
- Use `computed()` for derived state.
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in the `@Component` decorator.
- Prefer inline templates for small components.
- Prefer Reactive forms over Template-driven forms.
- Do NOT use `ngClass`; use native `[class.x]` / `[class]` bindings instead.
- Do NOT use `ngStyle`; use native `[style.x]` / `[style]` bindings instead.
- Do NOT use the `@HostBinding` / `@HostListener` decorators; put host bindings and
  listeners in the `host` object of the `@Component` / `@Directive` decorator.
- Follow the project file-naming convention: bare names without the legacy
  `.component` suffix (e.g. `app.ts`, `app.html`, `app.scss`, `app.spec.ts`).
- Use the configured component selector prefix `app`.

## Templates

- Keep templates simple; avoid complex logic in the template.
- Use native control flow (`@if`, `@for`, `@switch`) instead of the structural
  directives `*ngIf`, `*ngFor`, `*ngSwitch`.
- Always provide a `track` expression in `@for` blocks.
- Use the `async` pipe to handle observables in templates.
- Use `NgOptimizedImage` for static images (note: it does not work for inline base64 images).

## State Management

- Use signals for local component state.
- Use `computed()` for derived state.
- Keep state transformations pure and predictable.
- Do NOT use `mutate` on signals; use `set()` or `update()` instead.

## Services

- Design services around a single responsibility.
- Use `providedIn: 'root'` for singleton services.
- Use the `inject()` function instead of constructor injection.

## Routing & Bootstrap

- Define routes in `src/app/app.routes.ts` and provide them via `provideRouter`
  in `src/app/app.config.ts`.
- Implement lazy loading for feature routes (`loadComponent` / `loadChildren`).
- Keep the app zoneless; do not reintroduce `zone.js` or `provideZoneChangeDetection`.

## Styling

- SCSS is the default (`style: scss`, `inlineStyleLanguage: scss`). Global styles live
  in `src/styles.scss`.
- Keep per-component styles within budget (warn at 4kB, error at 8kB).

## Testing

- Test files are `src/**/*.spec.ts`; tests run on **Vitest** via `npm test` (not Karma/Jasmine).
- The Vitest globals (`describe`, `it`, `expect`, `beforeEach`, …) are available
  globally via `"types": ["vitest/globals"]`; do NOT import them from `vitest`.
- Use Angular's `TestBed` from `@angular/core/testing`; the DOM is provided by `jsdom`.
- Import standalone components directly into the testing module
  (`TestBed.configureTestingModule({ imports: [App] })`).
- Being zoneless, prefer `await fixture.whenStable()` over `fixture.detectChanges()`
  to flush async/template work.
- Read signal state by calling the signal (e.g. `component.title()`).
