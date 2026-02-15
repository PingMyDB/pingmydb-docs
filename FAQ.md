# Frequently Asked Questions (FAQ)

Everything you need to know about PingMyDb.

## 🔒 Security & Privacy

### Is it safe to provide my database URI?
Yes. Your database connection strings are encrypted at rest using **AES-256-CBC** encryption. Only our isolated monitoring workers have the temporary ability to decrypt them during a check. We never run queries on your data; we only perform simple connection handshakes or `SELECT 1` queries.

### Do you have access to my data?
Absolutely not. PingMyDb is designed as a heartbeat service. We only check if the connection is active. We don't have the permissions (and never ask for them) to inspect, modify, or delete your data. Our workers are hardcoded to only run no-op checks (`SELECT 1` or `admin.ping()`).

### Will this affect my database performance?
No. The resource footprint of a PingMyDb pulse is virtually zero. It is less than 0.1% of a standard t3.micro instance capacity. Every connection is opened, queried, and closed immediately to ensure we never exhaust your connection limits.

## 🛠 Features & Support

### Which databases are currently supported?
Currently, we support:
- **PostgreSQL** (Supabase, Neon, CockroachDB, etc.)
- **MongoDB** (Atlas, Self-hosted)
- **MySQL** (PlanetScale, DigitalOcean, etc.)

Support for Redis and Microsoft SQL Server is planned for future releases.

### How does the "Prevent Cold Start" feature work?
Cloud providers like Supabase or Neon often put free-tier databases into a "sleep" or "idle" state after a period of inactivity. This causes the next request to be slow (a "cold start"). PingMyDb sends periodic pings to keep the connection warm, ensuring your database is always ready to serve requests instantly.

### Can I share my database status with others?
Yes! Every user can generate a **Public Status Page** (e.g., `pingmydb.com/status/your-project`). You can choose which monitors to show publicly and share this URL with your team or users to show real-time uptime health.

### Can I monitor databases behind a VPC?
Currently, our workers require public internet access to reach your host. We are exploring SSH Tunneling and Site-to-Site VPN options for future updates. For now, you must ensure your database is accessible from the internet (security through IP whitelisting is recommended).

## 💳 Pricing & Plans

### How do I upgrade to the Student plan?
The Student plan offers Pro features at a heavily discounted rate (₹39/mo). 
1. Go to **Settings > Profile**.
2. Navigate to the **Student Verification** section.
3. Submit your **Institution Name** and a proof of ID.
4. Once our team verifies it (usually 24-48 hours), the Student plan will unlock on the Pricing page.

### What are the limits on the Free plan?
The Free plan allows for:
- 2 Active Monitors
- 1-hour Ping Interval
- Email Notifications only
- Basic Status Page

### How does billing work?
All payments are handled securely through **Razorpay**. Subscriptions are billed monthly. You can cancel or change your plan at any time from the billing section in your dashboard.

## 📡 Troubleshooting

### Why is my monitor showing as "Offline"?
1. **Network Access**: Ensure your database provider allows connections from external IPs. You may need to whitelist `0.0.0.0/0` or specify our worker's IP (if using a fixed worker).
2. **Credentials**: Double-check that your URI is correct and hasn't expired.
3. **Database State**: If the database is completely down or being maintained, the ping will fail and mark the monitor as offline.

### My alerts aren't reaching me.
Check your **Notification Settings**. Ensure your Email, Discord, or Slack webhook is correctly configured and that the "Enable Channel" toggle is active.
