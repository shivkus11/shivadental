# API reference

Base URL (local): `http://localhost:3000/api`

All authenticated endpoints use:

```http
Authorization: Bearer <JWT>
Content-Type: application/json
```

## Authentication

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| POST | `/auth/register` | Public | Create a patient account and link a matching clinic patient record when email and phone match |
| POST | `/auth/login` | Public | Login for patient, doctor, staff or admin |
| GET | `/me` | Authenticated | Current account and patient profile |
| GET | `/health` | Public | API / database health |

Register body:

```json
{"name":"Patient Name","email":"patient@example.com","password":"AtLeast10Characters","phone":"9999999999"}
```

Patient registration links to an unclaimed clinic patient record only when both email and phone match. Otherwise, a new patient profile is created (unless an email already exists in clinic records with a different phone, in which case registration is blocked and staff must assist). The same patient account works on the web portal and Java Android app.

Login body:

```json
{"email":"admin@example.com","password":"your-password"}
```

## Patients and records

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/patients?search=name` | Staff / doctor / admin | Search patient list |
| POST | `/patients` | Staff / admin | Create a patient record without a login |
| GET | `/patients/:id` | Own patient / staff / doctor / admin | Patient profile and medical history |
| PUT | `/patients/:id/history` | Staff / doctor / admin | Save medical history |
| GET | `/patients/:id/treatments` | Own patient / staff / doctor / admin | Treatment history |
| POST | `/treatments` | Staff / doctor / admin | Add treatment record |
| GET | `/patients/:id/dental-chart` | Own patient / staff / doctor / admin | Read dental chart JSON |
| PUT | `/patients/:id/dental-chart` | Staff / doctor / admin | Save dental chart JSON |

Patient creation body:

```json
{"fullName":"Patient Name","phone":"9999999999","email":"patient@example.com","dateOfBirth":"1995-05-10","gender":"Female","address":"Navi Mumbai"}
```

Medical history body:

```json
{"allergies":"Penicillin","conditions":"Hypertension","medications":"As prescribed","previousSurgeries":"None","notes":"Review before procedure"}
```

Treatment body:

```json
{"patientId":"PATIENT_UUID","procedureName":"Dental filling","toothNumber":"16","diagnosis":"Dental caries","treatmentNotes":"Procedure completed","charges":1200}
```

## Appointments

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/appointments` | Own patient / staff / doctor / admin | View appointment requests |
| POST | `/appointments` | Authenticated | Patient request or staff-created request |
| PATCH | `/appointments/:id/status` | Staff / doctor / admin | Change status |

Appointment body (patient app):

```json
{"appointmentAt":"2026-11-24T10:00:00+05:30","reason":"Tooth pain"}
```

Statuses: `pending`, `confirmed`, `completed`, `cancelled`.

## Cloudflare R2 files

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| POST | `/files/presign` | Own patient / staff / doctor / admin | Request a short-lived upload URL |
| POST | `/files/complete` | Own patient / staff / doctor / admin | Verify the object in R2 and save file metadata in PostgreSQL |
| GET | `/patients/:id/files` | Own patient / staff / doctor / admin | List file metadata |
| GET | `/files/:id/download` | Own patient / staff / doctor / admin | Get a short-lived download URL |

Upload sequence:

1. Call `/files/presign` with `category`, `fileName` and `contentType` (staff also sends `patientId`).
2. Send HTTP `PUT` with the file bytes to the returned `uploadUrl`. Keep the `Content-Type` identical to the value used in step 1.
3. Call `/files/complete` with `objectKey`, `category` and `fileName` (staff also sends `patientId`).

Categories: `xray`, `photo`, `document`. Supported formats: JPEG, PNG, WEBP and PDF. Default maximum size: 25 MB.

## Consent, payments, follow-up and audit

| Method | Endpoint | Access | Purpose |
|---|---|---|---|
| GET | `/patients/:id/payments` | Own patient / staff / doctor / admin | View payment status |
| POST | `/payments` | Staff / admin | Add payment entry |
| GET | `/patients/:id/consents` | Own patient / staff / doctor / admin | List consent forms |
| POST | `/consents` | Staff / doctor / admin | Create consent form |
| PATCH | `/consents/:id/acknowledge` | Linked patient account only | Accept / reject own consent |
| GET | `/patients/:id/follow-ups` | Own patient / staff / doctor / admin | View follow-ups |
| POST | `/follow-ups` | Staff / doctor / admin | Schedule follow-up |
| GET | `/audit-logs` | Admin | View recent audit entries |
| GET | `/dashboard/summary` | Staff / doctor / admin | Dashboard counts |

This API is a mini-project baseline. Add more validation and tests before real clinical use.
