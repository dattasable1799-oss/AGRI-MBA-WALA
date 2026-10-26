# AGRI MBA WALA — Firebase backend setup

This package is a front-end starter, not a live connected backend.

## 1. Create Firebase project
1. Open https://console.firebase.google.com/ and create a project.
2. Add a Web App and copy the Firebase web configuration.
3. Enable Authentication → Sign-in method → Phone.
4. Create a Cloud Firestore database.
5. Add your final hosting domain under Authentication authorized domains.

## 2. Phone OTP admin
- Firebase Phone Auth uses reCAPTCHA and can have SMS costs/quotas and abuse protection.
- Add an admin user using Phone Auth and record the resulting Firebase Auth UID.
- Never rely on a hidden admin URL as security.
- Only allow approved admin UIDs to read/update leads.

## 3. Suggested Firestore collection
`leads/{autoId}`:
- name: string
- phone: string
- service: `service_booking | crop_guidance | buyer_request`
- district: string
- details: string
- status: `new | contacted | in_progress | closed`
- createdAt: server timestamp
- consent: true

## 4. Example Firestore rules (replace the UID)
These rules are intentionally restrictive. They allow only the specified admin to read/update records.
For production public submissions, do not open unrestricted writes to Firestore. Use a Cloud Function or other server endpoint with validation, rate limiting, spam prevention, and consent checks.

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isAdmin() {
      return request.auth != null
        && request.auth.uid == "REPLACE_WITH_ADMIN_AUTH_UID";
    }
    match /leads/{leadId} {
      allow read, update, delete: if isAdmin();
      allow create: if false; // Public form writes must go through a validated server endpoint.
    }
  }
}
```

## 5. WhatsApp behavior
The public website opens a pre-filled WhatsApp message to `+919172115522`. The visitor must press Send in WhatsApp. This does not save the lead in a database.
For automatic logging and dashboard display, implement a validated backend endpoint that saves the lead and then optionally opens WhatsApp. For automatic outbound WhatsApp messages, use the official WhatsApp Business Platform and its opt-in/template rules.

## 6. Hosting
You can host the static front end on Firebase Hosting or another static host. A custom domain usually costs extra. Confirm current plan, quotas, and regional availability before launch.

## 7. Before launch
- Connect real backend and test OTP and Firestore rules.
- Add privacy notice, terms, contact details, consent and data retention/deletion workflow.
- Verify business contact number and service provider/ buyer listings.
- Do not claim data is saved until end-to-end tests confirm it.
