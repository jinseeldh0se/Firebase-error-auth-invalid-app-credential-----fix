# Firebase-error-auth-invalid-app-credential-----fix

(this is not only for vite this applicable to every farmeworks)

📌 Understanding Vite Network IPs and Firebase Authentication Errors

When working with Firebase Authentication inside a Vite-based React application, many developers run into issues where OTP or sign-in flows fail with errors such as:

auth/invalid-app-credential
auth/unauthorized-domain


These problems often come from a misunderstanding of how Vite exposes local development servers and how Firebase validates origins.

This note explains both concepts clearly.

1. Vite Network IPs Are the Actual Origins Your App Runs On

A typical Vite dev server output looks like:

Local:    http://localhost:5173/
Network:  http://192.168.56.1:5173/
Network:  http://192.168.220.3:5173/


The “Network” entries are private LAN IPs coming from your device’s network interfaces. They are only accessible inside your local network and therefore safe to share publicly. They cannot be used to track or hack your system.

Important:
These IPs are the real origins the browser may use when loading your application.

2. Firebase Requires an Exact Origin Match

Firebase Authentication validates the exact origin of the request:

scheme + domain/IP + port


For example, if the browser accesses the app using:

http://192.168.56.1:5173


Firebase must have that exact IP whitelisted under Authorized Domains.

Whitelisting unrelated or unused IPs like 121.0.0.1 has no effect unless the dev server is actually running on that address.

3. Why Firebase Shows auth/invalid-app-credential

This error occurs when Firebase rejects the request’s origin.

Common causes include:

• The app is opened from an IP not included in Authorized Domains

Example:
Vite serves http://192.168.56.1:5173, but Firebase only whitelisted localhost.

• The developer whitelisted an IP the Vite server is not bound to

Example:
Adding 121.0.0.1 when Vite is not running on that IP.

• Opening the app at localhost still resolves internally to a different IP

So Firebase sees a different origin than expected.

Firebase is strict — the origin must match exactly, or authentication fails.

4. When You Need to Expose Additional IPs in Vite

If you want the dev server to be reachable from any network address (LAN, VM adapter, custom IP), Vite must be started with host binding:

npm run dev -- --host


This binds Vite to all available interfaces, allowing Firebase to validate additional origins.

5. Security Best Practices

Safe to expose:

localhost

Private LAN IPs (e.g., 192.168.x.x, 10.x.x.x)

Vite dev server output

Firebase Authorized Domains list

Not safe to expose:

Public IP address

API keys

Firebase service account keys

JWT tokens

Cloud credentials

Summary

The IPs Vite or any other prints under “Network” are your true application origins.
Firebase must explicitly whitelist those exact origins to allow authentication.
Errors like auth/invalid-app-credential occur when the application’s origin and the Authorized Domains list do not match.
Whitelisting unused IPs has no effect unless the dev server is actually bound to that IP.

Vite’s output IPs are private and safe to share publicly; they cannot be used to track or compromise your system.

Use --host if you need Vite to expose additional IPs.
