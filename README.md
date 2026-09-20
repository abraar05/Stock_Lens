# StockLens Mobile

A restrained Android operations app for the StockLens inventory and IMEI audit workflow.

## Access model

StockLens Mobile uses Google OAuth for identity and the connected Google Sheet as the data gate:

1. A person signs in with Google.
2. The app reads the signed-in email from Google.
3. The app reads the `Users` tab from the permitted spreadsheet.
4. The email must be present with `active=true`.
5. The person must also have at least **Viewer** permission on the spreadsheet in Google Drive.
6. Only users with `admin` role can add allowlisted users from the Settings tab. Admin write access to the `Users` tab is required for this action.

This is deliberately not a public-sheet design. Keep the spreadsheet restricted to named Google accounts. The app requests Google Sheets access so users cannot see operational data unless their own Google account can read the connected spreadsheet.

## Google Sheet structure

Use a spreadsheet with these tabs:

- `Master_Records`: the data tab. The first row is treated as headers. The app recognizes `IMEI / Serial`, `IMEI`, `IMIE`, `IMEI 1`, and `IMEIs`, plus date, status, party/customer, product/model, wing/location, quantity, source file, and source sheet variants.
- `Users`: required columns are `email`, `role`, `active`, and `note`. Roles are `viewer`, `reviewer`, and `admin`.

Recommended `Users` example:

```text
email | role | active | note
owner@company.com | admin | true | Primary administrator
reviewer@company.com | reviewer | true | Warranty and refund review
staff@company.com | viewer | true | Daily operations
```

## Required Google Cloud setup

The Sheets API and OAuth branding are configured in project `amazing-sunset-509016-s3` as **StockLens Mobile**, using an External audience and `bahalul1964@gmail.com` as the support/admin account. The Web OAuth client is stored in the project environment as `EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID`, and the runtime also consumes `EXPO_PUBLIC_STOCKLENS_ADMIN_EMAIL` and `EXPO_PUBLIC_GOOGLE_PROJECT_ID`.

The only remaining data-source setup is the live Spreadsheet ID. Enter it in **Settings → Google Sheet source → Spreadsheet ID**, or provide `EXPO_PUBLIC_GOOGLE_SHEET_ID` at build time. The ID is the long value between `/d/` and `/edit` in the Sheet URL. Keep the `Master_Records` and `Users` tab names unless your workbook uses different names. These are public client configuration values; never place a Google client secret or a service-account private key in the app.

For a standalone Android APK, create an Android OAuth client after the first EAS build and register the generated package name and signing SHA-1. The Web client is already configured for the Expo web preview callback.

## Local development

```bash
pnpm install
pnpm check
pnpm dev
```

## Android install build

The project is Expo SDK 54. A signed APK requires an Android application identifier and signing credentials. Use either EAS Build or a local Android toolchain.

```bash
npx expo config --type public
npx eas build:configure
npx eas build --platform android --profile preview
```

Use an `eas.json` preview profile with `buildType: "apk"` for a directly installable APK. For Play Store release, use an Android App Bundle (`.aab`) and a production signing key.

The current environment can validate the Expo project and preview UI. It cannot mint a production-signed APK without the final Google OAuth client IDs and Android signing credentials. The app source is ready for that final build step.

## Core screens

- Overview: live Sheet metrics and latest source activity
- IMEI audit: full event timeline and model/customer/status/source comparison
- Reviews: ownership, warranty, and refund decision-support queue
- Settings: Sheet connection, user allowlist, roles, and sync controls

The app does not approve refunds automatically. It surfaces evidence and flags; staff should verify invoices, delivery proof, return authorization, inspection evidence, and policy before making a final decision.
