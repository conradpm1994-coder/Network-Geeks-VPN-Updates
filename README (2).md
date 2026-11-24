# Network Geeks VPN

White-label VPN app: React Native (Android first), Node backend, web admin panel.

    backend/      API: auth (email + OTP, Google), tickets, admin, store webhooks, ResellPortal
    mobile/       Android app (Expo dev client + native WireGuard module)
    admin-web/    Admin panel (static page): host on your domain and open /#admin
    .github/      CI: signed Android AAB and APK builds
    secrets-local/  (gitignored) keystore and signing values for local builds

## 1. Push to GitHub
    git init && git add . && git status    # confirm .env files and secrets-local/ are NOT listed
    git commit -m "Network Geeks VPN" && git branch -M main
    git remote add origin git@github.com:<you>/network-geeks-vpn.git && git push -u origin main

## 2. Run the backend
    cd backend && npm install && npm start
Admin login: `AdminGeek` with the password in `backend/.env`. The panel makes you set new credentials on first sign-in (you may re-enter the same ones).
`RP_TEST_MODE=1` keeps ResellPortal in test mode. Set it to 0 only after testing.
Base URL note: `https://panel.resellportal.com/mcp` is ResellPortal's MCP endpoint (for AI assistants). The backend uses their REST API at `https://panel.resellportal.com/wp-json/resellportal/v1` (RP_BASE + path in server.js).

## 3. Prepare the mobile project (once)
    cd mobile && ./bootstrap.sh      # aligns library versions, creates package-lock.json
Commit `package.json` and `package-lock.json`. Replace the placeholder icons in `mobile/assets/`.
Change `android.package` in `app.json` to your real package name, then register that package and the SHA-1 below in your Google OAuth Android client.

## 4. GitHub Actions secrets (Settings, Secrets and variables, Actions)
| Secret | Value |
| --- | --- |
| ANDROID_KEYSTORE_BASE64 | `base64 -w0 secrets-local/networkgeeks-upload.jks` (macOS: `base64 -i file`) |
| KEYSTORE_PASSWORD | keystore password |
| KEY_ALIAS | `networkgeeks` |
| KEY_PASSWORD | key password |
| EXPO_PUBLIC_API_BASE | your API URL |
| EXPO_PUBLIC_RC_ANDROID | RevenueCat Android public key |
| EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID | Google OAuth client ID |
Run "Android build" under Actions. Download the artifact: upload the `.aab` to Play Console, and use the `.apk` for device testing.

## 5. Local release build (optional)
    source secrets-local/signing.env && cd mobile && npx expo prebuild --platform android --clean && cd android && ./gradlew bundleRelease

Keystore SHA-1: DA:4A:24:72:2D:D8:1C:E0:0B:18:C6:B9:C2:33:66:43:0B:1E:05:9F
For production, create a fresh keystore on your own machine with `mobile/make-keystore.sh`.
