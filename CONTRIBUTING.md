# Contributing to PingMyDb

Thank you for your interest in contributing! PingMyDb is a developer-focused tool, and we welcome improvements of all kinds.

## Getting Started
1. Fork the repository.
2. Clone your fork locally.
3. Follow the setup instructions in the `README.md` of both `frontend` and `backend` folders.

## Folder Structure
- `/frontend`: React application (UI/UX).
- `/backend`: Express API, Monitoring Workers, and Database migrations.
- `/docs`: High-level system documentation.

## Development Workflow
1. **Branching**: Create a feature branch for your changes (`git checkout -b feature/awesome-new-thing`).
2. **Linting**: Ensure your code follows the existing style patterns.
3. **Commit Messages**: Use descriptive commit messages (e.g., `feat: add Discord notification support`).

## Adding New Databases
To add support for a new database type:
1. Update `frontend/types.ts` to include the new type.
2. Add a new connection handler in `backend/workers/monitoringWorker.js`.
3. Update the `MonitorCreation` UI to include the new provider icon and options.

## Submitting Pull Requests
- Keep PRs focused on a single change.
- Provide a clear description of what the PR accomplishes and any testing you've done.
- Include screenshots for any UI/UX changes.
