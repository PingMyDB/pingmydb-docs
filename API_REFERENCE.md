# API Reference

All requests must include the `token` header for authenticated routes.

## Authentication
- `POST /api/auth/signup`: Create a new user account.
- `POST /api/auth/login`: Authenticate and receive a JWT.
- `POST /api/auth/verify-email`: Verify email with token.
- `POST /api/auth/resend-verification`: Resend email verification link.
- `POST /api/auth/forgot-password`: Request password reset email.
- `POST /api/auth/reset-password`: Update password using token.
- `POST /api/auth/check-email`: Check if email address is already registered.

## Monitors
- `GET /api/monitors`: List all monitors for the current user.
- `POST /api/monitors`: Create a new monitor.
- `PUT /api/monitors/:id`: Update an existing monitor.
- `DELETE /api/monitors/:id`: Delete a monitor.
- `POST /api/monitors/verify`: Test a database URI connection before saving.

## Profile & Settings
- `GET /api/profile`: Get current user profile information.
- `PUT /api/profile`: Update name or avatar URL.
- `POST /api/profile/send-email-otp`: Trigger email change verification.
- `PUT /api/profile/email`: Confirm email change with OTP.
- `POST /api/profile/verify-student`: Submit student verification request.

## Notifications
- `GET /api/notifications/channels`: Get all active alert channels.
- `POST /api/notifications/channels`: Add a new alert channel (Slack/Discord/Email).
- `DELETE /api/notifications/channels/:id`: Remove a channel.

## Admin (Internal)
- `GET /api/admin/stats`: System-wide health and user statistics.
- `GET /api/admin/users`: List all registered users (Paginated).
