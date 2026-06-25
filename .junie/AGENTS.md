# suddha-app — Development Guide

Angular **21** single-page application. This document captures the project-specific
details that are not obvious from a generic Angular setup.

## 1. Build / Configuration

- **Package manager is pinned**: `npm@11.11.0` via the `packageManager` field in
  `package.json`. Use npm (Corepack will honor this pin); do not switch to yarn/pnpm.
- **Node**: developed/verified on Node `v24.x`.
- Install deps with `npm ci` (preferred for reproducible installs) or `npm install`.

### Builder stack (important — not the legacy Webpack/Karma stack)
This project uses the modern Angular toolchain configured in `angular.json`:
- Build: `@angular/build:application` (esbuild/Vite based), entry `src/main.ts`.
- Serve: `@angular/build:dev-server`.
- Test: `@angular/build:unit-test` backed by **Vitest** (NOT Karma/Jasmine).

### Common commands
- `npm start` / `ng serve` — dev server (`development` config, optimizations off, source maps on).
- `npm run build` — production build by default (`defaultConfiguration: production`).
- `npm run watch` — incremental development build.
- `npm test` / `ng test` — run unit tests once (non-watch) via Vitest.

### Environments
- `src/environments/environment.ts` (prod) is swapped for
  `src/environments/environment.development.ts` via `fileReplacements` in the
  `development` build configuration. Add new environment-dependent values to BOTH files.

### TypeScript / Angular compiler settings (strict)
`tsconfig.json` is strict and will fail builds on:
- `strict`, `noImplicitOverride`, `noImplicitReturns`, `noFallthroughCasesInSwitch`
- `noPropertyAccessFromIndexSignature` — index-signature members must be accessed with
  bracket notation (`obj['key']`), not dot notation.
- Angular: `strictTemplates`, `strictInjectionParameters`, `strictInputAccessModifiers`.

## 2. Testing

### How tests are wired
- Test files: `src/**/*.spec.ts` (see `tsconfig.spec.json` `include`).
- `tsconfig.spec.json` adds `"types": ["vitest/globals"]`, so `describe`, `it`,
  `expect`, etc. are **globally available — do NOT import them** from `vitest`.
- Tests use Angular's `TestBed` from `@angular/core/testing`. The DOM environment is
  provided by `jsdom` (dev dependency).

### Running
- `npm test` runs all specs once and exits (exit code 0 on success). Output reports
  per-file pass counts, e.g. `src/app/app.spec.ts (2 tests)`.

### Component testing notes (zoneless / signals)
- The app is signal-based and zoneless: prefer `await fixture.whenStable()` after
  creating a component instead of `fixture.detectChanges()` to flush async/template work
  (see `src/app/app.spec.ts`).
- Standalone components are imported directly into the testing module:
  `TestBed.configureTestingModule({ imports: [App] })`.
- Component state uses Angular signals; read them by calling the signal
  (e.g. `component.title()`).

### Adding a new test
1. Create `*.spec.ts` next to the unit under test under `src/`.
2. Use the global Vitest API (no imports for `describe`/`it`/`expect`).
3. For components, configure `TestBed` with the standalone component in `imports`.
4. Run `npm test`.

### Verified example (this exact pattern was run and passed)
```ts
import { TestBed } from '@angular/core/testing';
import { App } from './app';

describe('Sample demonstration', () => {
  it('uses Vitest globals (no explicit import needed)', () => {
    expect(1 + 1).toBe(2);
  });

  it('reads a signal value from the root component', () => {
    const fixture = TestBed.createComponent(App);
    const app = fixture.componentInstance as unknown as { title: () => string };
    expect(app.title()).toBe('suddha-app');
  });
});
```
Placing this in `src/app/sample.spec.ts` and running `npm test` yields all tests
passing (4 total alongside the existing `app.spec.ts`).

## 3. Additional Development Info

- **Styling**: SCSS is the default. The component schematic is configured for
  `style: scss` and `inlineStyleLanguage` is `scss`; global styles live in
  `src/styles.scss`.
- **Component selector prefix**: `app` (configured in `angular.json`).
- **Formatting**: Prettier `^3.x` is a dev dependency. There is no committed Prettier
  config, so the formatting follows Prettier defaults; match the existing 2-space
  indentation and single quotes used across the codebase.
- **File naming**: components use the bare name without the legacy `.component` suffix
  (e.g. `app.ts`, `app.html`, `app.scss`, `app.spec.ts`). Follow this convention.
- **Routing**: routes are defined in `src/app/app.routes.ts` and provided via
  `provideRouter` in `src/app/app.config.ts`. The app bootstraps with
  `provideBrowserGlobalErrorListeners()`.
- **Production budgets** (build will warn/error): initial bundle warn at 500kB / error
  at 1MB; per-component styles warn at 4kB / error at 8kB.
