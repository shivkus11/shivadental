# Dental Clinic Management System

A simple clinic-management project for **Shree Gurudev Daat Clinic**, based on the uploaded mini-project presentation.

## Technology used

| Area | Technology |
|---|---|
| Web portal (patient + staff + doctor) | HTML, CSS, JavaScript |
| Patient mobile app | Android Studio, Java (no Kotlin) |
| Backend API | Node.js + Express |
| Main database | PostgreSQL |
| X-ray / photo / document storage | Cloudflare R2 (S3-compatible API) |
| Hosting | Render Web Service + Render PostgreSQL |

**Firebase is not used.** User accounts, patient details, appointments, medical history, treatments, payments, consent records and file metadata are stored in PostgreSQL. Actual images and PDFs are stored privately in Cloudflare R2. The backend issues short-lived presigned upload/download URLs so the R2 secret keys are never included in the web or Android application.

## Project layout

```text
DentalClinicManagementSystem/
  backend/
    src/                 Express API, authentication and R2 integration
    migrations/          PostgreSQL schema
    scripts/             Database migration and clinic-user creation scripts
    .env.example
    package.json
  web/                   Patient portal + staff / doctor portal (HTML, CSS, JavaScript)
  android/               Native Android patient app (Java only)
  docker-compose.yml     Local PostgreSQL for development
  README.md
  DEPLOY_RENDER.md
  API_REFERENCE.md
  DATABASE_DESIGN.md
  SECURITY_NOTES.md
```

## What is implemented

- Patient registration and login on both website and Android, plus clinic staff / doctor / admin login on the website.
- Unique patient ID and patient profile.
- Medical history: allergies, conditions, medicines, previous surgeries and notes.
- Appointment requests from the patient website or Android app; staff can confirm, complete or cancel requests in the clinic portal.
- Treatment records with tooth number, diagnosis, notes and charges.
- Dental X-ray, photo and document upload to private Cloudflare R2 storage.
- Short-lived file viewing links after server-side access checks.
- Payment entries and patient-visible payment status.
- Treatment consent creation in the staff portal and patient acknowledgement in Android.
- Follow-up scheduling and a basic JSON dental-chart field.
- Audit log for selected actions.
- Role checks: patients can access only their own patient-linked records; staff, doctors and admins can manage clinic records. Only the linked patient account can accept or reject that patient’s consent.

The browser portal supports both **patients** and **clinic staff / doctors**. Patients can register or sign in on the website and access their own appointments, medical history, treatment records, X-rays/photos, payment status, consent forms and follow-ups. Staff and doctors continue to use the clinic-management dashboard. The native Android app is also a patient app. Both patient clients use the same account and shared backend / PostgreSQL database.

### Open patient registration directly

- Sign out of the current staff/doctor account to return to the login screen.
- Click **Create a patient account** below the sign-in form.
- You can also open `http://localhost:3000/?register=1` to go directly to patient registration, even when this browser previously remembered a staff login.
- While signed in as clinic staff or a doctor, the header also shows **Patient account registration**. This shortcut opens the registration page after clearing the current browser session.
- Patients should not share passwords. Use the same patient email/password in the website and Android app.

## Requirements

- Node.js 20 or newer.
- PostgreSQL 13 or newer (or Docker Desktop).
- Android Studio with Android SDK 35 and Java 17.
- A Cloudflare account with an R2 bucket and an R2 API token.

---

# Run locally (Windows)

## 1. Start PostgreSQL

Install Docker Desktop, open PowerShell in the project root, and run:

```powershell
docker compose up -d postgres
```

This creates a local database named `dental_clinic` on port `5432`.

## 2. Configure the backend

Open `backend` and copy `.env.example` to `.env`:

```powershell
cd backend
Copy-Item .env.example .env
```

Edit `backend/.env`. For local PostgreSQL, these settings are suitable:

```env
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/dental_clinic
PGSSL=false
PORT=3000
NODE_ENV=development
JWT_SECRET=replace_with_a_random_secret_at_least_32_characters
```

Generate a safe local JWT secret using Node.js:

```powershell
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Paste the generated value into `JWT_SECRET`.

Add your Cloudflare R2 details to the same `.env` file. Until R2 is configured, the rest of the API can run, but image upload will return an R2-not-configured message.

```env
R2_ACCOUNT_ID=your_account_id
R2_ACCESS_KEY_ID=your_access_key_id
R2_SECRET_ACCESS_KEY=your_secret_access_key
R2_BUCKET=dental-clinic-files
R2_ENDPOINT=https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com
R2_PRESIGN_SECONDS=300
MAX_UPLOAD_BYTES=26214400
CORS_ORIGINS=http://localhost:3000
```

Keep `.env` private. Do not upload it to GitHub.

## 3. Install packages and create tables

Run inside `backend`:

```powershell
npm install
npm run migrate
```

## 4. Create the first clinic administrator

Still inside `backend`:

```powershell
npm run create-user -- --email admin@example.com --password "ChangeThisPassword123!" --role admin --name "Clinic Admin"
```

Use your own email and a strong password. To create a doctor account, use `--role doctor`; to create a receptionist account, use `--role staff`.

Example:

```powershell
npm run create-user -- --email doctor@example.com --password "ChangeThisPassword123!" --role doctor --name "Dr. Example"
```

## 5. Start the web portal and API

```powershell
npm start
```

Open **http://localhost:3000** in Chrome. Sign in with the admin / doctor / staff account created in step 4. The website detects the account role and opens the correct portal.

Patients can use **Create a patient account** on the same sign-in page. Existing clinic patients should enter the same email and phone number recorded by staff so the account links to their existing patient ID. New patients receive a new patient ID.

Health check: **http://localhost:3000/api/health**

## 6. Open the Android project

1. Open Android Studio.
2. Choose **Open** and select the `android` folder inside this project.
3. Let Android Studio sync the Gradle project. The included wrapper launcher downloads Gradle 8.7 on first use, so an internet connection is needed for the first sync.
4. Start an Android emulator and press **Run**.
5. The default API address is `http://10.0.2.2:3000/`, which points from the Android emulator to the host computer.
6. Register a new patient account in the Android app. It will use the same backend as the website.

For a physical Android phone, the phone and computer must be on the same Wi-Fi network. Build with your computer's LAN IP instead of `10.0.2.2`, for example:

```powershell
cd android
.\gradlew.bat assembleDebug -PAPI_BASE_URL=http://192.168.1.10:3000/
```

Replace `192.168.1.10` with your computer's local IP address. Allow Node.js through Windows Firewall on your private network.

---

# Deploy to Render

Follow **[DEPLOY_RENDER.md](DEPLOY_RENDER.md)**. In brief: create Render PostgreSQL, create a Node web service from the GitHub repository, run the migration during the build, add database/JWT/R2 environment variables, and use the generated HTTPS service URL in the Android build.

## Build the Android app for Render

After deployment, build the app with your Render HTTPS URL:

```powershell
cd android
.\gradlew.bat assembleDebug -PAPI_BASE_URL=https://YOUR-SERVICE.onrender.com/
```

The debug APK is created at:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

Install that APK on a test phone. For Play Store release, configure a signed release build and remove cleartext development access from the manifest.

## Important scope note

This is a mini-project starter / demonstrator, not a certified clinical information system. Use dummy data while testing. Before storing real patient records, have the clinic's IT/security and legal advisers review authentication, hosting region, access controls, backups, retention, consent wording, incident response and applicable privacy obligations. Do not use the demo admin password in a live deployment.
