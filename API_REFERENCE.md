# API Reference

All requests must include the `token` header for authenticated routes.

## Authentication
- `POST /api/auth/register`: Create a new user account.
- `POST /api/auth/login`: Authenticate and receive a JWT.

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

## Payments
- `POST /api/payments/create-order`: Initialize a Razorpay checkout session.
- `POST /api/payments/verify`: Confirm payment signature and update user plan.

## Admin (Internal)
- `GET /api/admin/stats`: System-wide health and user statistics.
- `GET /api/admin/users`: List all registered users (Paginated).
