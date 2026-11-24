# Security and privacy notes

This repository contains a learning / mini-project implementation. It is not certified or independently security-audited for live clinical use.

## Included safeguards

- Passwords are hashed with bcrypt (12 rounds).
- JWT authentication with an expiry.
- Staff / doctor / admin roles are checked by protected API routes.
- Patient accounts (web and Android) are restricted to their linked patient record for patient-scoped endpoints.
- SQL statements use parameterized queries.
- Express security headers and authentication rate limiting are enabled.
- Cloudflare R2 bucket keys stay on the backend; the bucket can remain private.
- Upload and download links are short-lived presigned URLs.
- File category, MIME type, ownership path and object size are checked before the file record is added.
- Only the linked patient account can acknowledge their consent; clinic staff cannot acknowledge on a patient’s behalf through this endpoint.
- Selected create, update, login and upload actions are written to `audit_logs`.

## Before real clinic use

- Replace the simple JWT-in-sessionStorage web flow with a reviewed session design (for example, secure, HttpOnly, SameSite cookies for browser login) and store Android credentials in Android Keystore-backed encrypted storage.
- Add a password-reset workflow, stronger password policy, account lockout / throttling and staff account provisioning controls.
- Add HTTPS-only production configuration and remove `android:usesCleartextTraffic="true"` for production builds.
- Review every route and patient ownership rule with negative authorization tests.
- Add malware scanning, stricter file signature inspection, file retention and deletion workflows for uploaded reports.
- Configure PostgreSQL backups, recovery drills, monitoring and least-privilege database credentials.
- Establish clinic-approved consent text and patient identity verification procedures.
- Obtain appropriate privacy, security and legal review before processing real patient data.
