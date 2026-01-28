**Getting Started in iOS Mobile Application Testing**

1. **Setting Up an iOS Testing Environment**

Scenario: You’re preparing to test an iOS banking app. The client only provides a production App Store build (no .ipa for testing) and does not authorize device jailbreaking.

Question: What is the appropriate next step according to PTES Pre-Engagement guidelines?

Answer: Pause testing and request a properly provisioned .ipa file or test build that permits installation via MDM/TestFlight. Production-store builds restrict dynamic instrumentation and filesystem inspection, potentially leading to incomplete results and legal issues. The client must provide a testable build and written authorization for the necessary level of device access.

Scenario: The project timeline forbids buying physical Apple devices; you must use cloud device farms. The vendor prohibits root/jail access.

Question: Is cloud-based testing acceptable for full security assessment?

Answer: Cloud device farms are OK for UI/functional checks and some runtime observations but insufficient for deep forensics, persistence checks, or low-level SSL pinning bypasses. Request a scoped exception or limited onsite device access for high-fidelity tests.

2. **Static Analysis of iOS Apps**

Scenario: You extract the app bundle and find API keys stored in a .plist file along with plaintext URLs for backend services.

Question: What vulnerability category does this fall under and what is the risk?

Answer: This is Insecure Local Storage (OWASP MASVS-STORAGE-1). Hard-coded secrets can enable attackers to:

Bypass authentication checks

Abuse backend APIs

Reverse-engineer private endpoints

Even without device compromise, a malicious actor can extract and misuse the credentials.

Scenario: You find obfuscated Objective-C class names but discover a hardcoded API endpoint in a binary string segment.

Question: How do you classify this issue and what next?

Answer: Classify as sensitive configuration disclosure / insecure secrets. Report endpoints as IOCs, attempt to identify the backend owner, and recommend moving secrets to secure server-side storage or the Keychain.

3. **Dynamic Runtime Manipulation**

Scenario: The tested app implements SSL pinning, preventing you from intercepting HTTPS traffic with a proxy.

Question: What is your next step for dynamic testing?

Answer: Use a runtime hooking tool like Frida or Objection to bypass SSL pinning in memory. Example Frida command:

frida -U -f com.company.appname -l ios-ssl-bypass.js --no-pause

This allows observation of sensitive API calls while staying within the authorized scope.

4. **Testing Data Security & Storage Protections**

Scenario: During runtime review, you find that authentication tokens are written to console logs and remain in device logs after logout.

Question: Which standard is being violated & what’s the security impact?

Answer: This violates MASVS-STORAGE-3 (sensitive data exposure in logs). Logs are accessible to other apps (especially on jailbroken devices) Attackers can hijack sessions using leaked tokens

Scenario: The app stores session tokens in NSUserDefaults and they persist across uninstall/reinstall on some devices.

Question: What does this indicate and what's the fix?

Answer: Indicates insecure token storage and possible misuse of persistent storage (or improper Keychain use). Recommend moving tokens to Keychain with proper access controls and ensure tokens are cleared on logout and on app uninstall where applicable.
