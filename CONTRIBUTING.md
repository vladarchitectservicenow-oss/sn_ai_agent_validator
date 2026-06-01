# Contributing to AI Agent Readiness Validator

## Development Setup

1. Clone the repository
2. All source code is in ServiceNow Studio XML format under `src/`
3. Tests use Node.js mock runtime (see `tests/README` if present)
4. Run `node tests/test_aascanner.js` before submitting changes

## Commit Conventions

- Use conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `chore:`
- Reference issue numbers when applicable
- Keep commits focused — one logical change per commit

## Pull Request Process

1. Ensure all tests pass before opening a PR
2. Update documentation if adding or changing features
3. Verify README word count stays above 2000
4. Check that LICENSE copyright headers are present on all new files
5. Request review from the repository owner

## Code Style

- JavaScript: ES5-compatible (ServiceNow Rhino runtime)
- Use `Class.create()` for Script Includes
- Document all public methods with JSDoc comments
- No `console.log` in production code — use `gs.debug()` / `gs.info()`
