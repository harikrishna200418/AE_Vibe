# FCM backend integration for AEVibe2

This app is still a pure WebView wrapper. FCM only adds real-time alerts and optional deep-link routing.

## 1) Android app contract

The app now:

- Generates an FCM token with Firebase SDK.
- Sends token to backend endpoint:
  - `POST https://aevibe.click/api/mobile/register-fcm-token`
  - JSON body:

```json
{
  "token": "<fcm_device_token>",
  "platform": "android",
  "packageName": "com.example.aevibe",
  "deviceId": "<android_secure_id>",
  "androidVersion": "14",
  "sdkInt": 34
}
```

- Receives FCM push even when app is closed.
- Opens `MainActivity` when notification is tapped.
- If payload contains `targetUrl` (or `url` or `path`), app loads that website route.

## 2) Backend requirements

Implement two backend pieces without changing website UI:

1. **Token registration API**
   - Upsert token against the authenticated user account.
   - Suggested table columns: `user_id`, `fcm_token`, `platform`, `device_id`, `updated_at`, `is_active`.

2. **Admin update trigger**
   - When admin updates user-specific data, send FCM to that user's token(s).

## 3) FCM message payload

Use Firebase Admin SDK (preferred) or FCM HTTP v1 API and include data keys like:

```json
{
  "message": {
    "token": "<fcm_token>",
    "notification": {
      "title": "Profile updated",
      "body": "Your document status changed to Approved"
    },
    "data": {
      "targetUrl": "https://aevibe.click/dashboard/messages"
    },
    "android": {
      "priority": "high"
    }
  }
}
```

Supported deep-link keys in this app: `targetUrl`, `url`, `path`.

## 4) Quick test flow

1. Install app with valid `google-services.json` in `app/google-services.json`.
2. Log in inside WebView.
3. Confirm token row is stored for that user.
4. Trigger admin action for that user.
5. Verify push appears when app is foreground/background/terminated.
6. Tap notification and verify app opens correct website page.

