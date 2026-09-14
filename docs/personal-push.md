# Buzz Personal private deployment

This fork supports a separately signed iPhone app and an operator-owned HTTPS
push gateway. It does not enable push in Block's App Store application.

The gateway accepts an operator-configured HTTPS delivery URL, including a
non-default TLS port. Credentials, query strings, fragments, and paths other
than `/v1/deliveries/apns` remain invalid. App Attest, relay signatures, token
encryption, delegation scoping, replay protection, and Apple certificate
verification remain enforced.

The upstream `buzz-ios-dogfood` wire profile is retained for compatibility;
our gateway binds that profile to our own Apple Team ID, bundle identifier,
and APNs certificate. The `push.buzz.xyz` strings inside App Attest transcripts
are protocol audience constants, not network destinations. They must stay
identical between the native client and gateway.

The private iOS identity is configured in the ignored
`mobile/ios/Flutter/AppOverrides.xcconfig`. Both app and Notification Service
Extension must be signed, share the configured App Group and Keychain group,
and match the gateway's Apple identity. The gateway verifies production App
Attest, so use `BUZZ_APP_ATTEST_ENVIRONMENT = production` even when the APNs
transport uses sandbox with a development provisioning profile.

Build with `flutter build ios --release --dart-define=BUZZ_PUSH_GATEWAY_URL=https://your-gateway:8444`.
The relay descriptor activates push and supplies the relay signing authority;
the gateway address is an existing mobile compile-time setting, not a field
discovered from the descriptor. Set the relay's `BUZZ_PUSH_GATEWAY_DELIVERY_URL`
to the same origin followed by `/v1/deliveries/apns`.

Push payloads contain a generic activity alert and default sound. The extension
fetches and verifies message content from the relay. Tailscale connectivity is
needed for private gateway enrollment and rich notification retrieval. The Mac
must stay awake and online while hosting the relay and gateway.

Acceptance requires real device enrollment, a relay push lease, a gateway/APNs
delivery, visible lock-screen notification, sound under appropriate phone
settings, and tapping to the correct message. Automated tests and APNs acceptance
alone do not establish phone presentation.
