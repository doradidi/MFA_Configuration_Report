# MFA_Configuration_Report
Multi-Factor Authentication (MFA)
Configuration & Test Report

Prepared by: Dora Odidi
Course: Cybersecurity
Platform: Mentorship for Acceleration (M4ace)
Date: 19 April 2026



1. Introduction
Multi-Factor Authentication (MFA) is a security mechanism that requires users to verify their identity using two or more independent factors before gaining access to a system. This report documents the configuration of MFA for a cloud-based web application (CloudSecure), covering setup steps, supported factor types, policy configuration, and test case results.

MFA significantly reduces the risk of account compromise. Even if a user's password is stolen, an attacker cannot access the account without also possessing the second factor.

2. MFA Concepts
2.1 The Three Authentication Factors

Factor	Category	Example
Factor 1	Something you know	Password, PIN
Factor 2	Something you have	Authenticator app, SMS code, hardware token
Factor 3	Something you are	Fingerprint, face recognition (biometrics)

2.2 MFA Flow
The standard MFA authentication flow in CloudSecure:

1.	User enters their email address and password (Factor 1 — knowledge)
2.	System validates credentials against the user database
3.	If credentials are valid, user is prompted to complete Factor 2
4.	User selects an MFA method (Authenticator App, SMS, Email, or Backup Code)
5.	System generates and sends a time-based one-time password (TOTP) or sends an OTP via the chosen channel
6.	User enters the code within the expiry window (5 minutes)
7.	System validates the code; if correct, a secure session is created
8.	User gains access to the application

3. Cloud Platform MFA Configuration
3.1 Platform
Provider: Amazon Web Services (AWS)
Service Used: AWS IAM Identity Center (formerly AWS SSO) + Amazon Cognito for application-level MFA

3.2 AWS IAM — Enabling MFA for Console Users
The following steps were taken to enforce MFA for all IAM users accessing the AWS console:

9.	Log in to AWS Management Console as root or admin
10.	Navigate to IAM > Users
11.	Select a user and open the Security credentials tab
12.	Under Multi-factor authentication, click Assign MFA device
13.	Choose the device type: Virtual MFA device (authenticator app)
14.	Scan the QR code using Google Authenticator or Microsoft Authenticator
15.	Enter two consecutive TOTP codes to confirm setup
16.	Click Assign MFA — the device is now active

3.3 AWS Cognito — Application-Level MFA
For the CloudSecure web application, MFA is enforced at the application layer using Amazon Cognito User Pools:

17.	Open AWS Console > Cognito > User Pools
18.	Select the CloudSecure user pool
19.	Go to Sign-in experience > Multi-factor authentication
20.	Set MFA enforcement to: Required (mandatory for all users)
21.	Enable the following MFA methods:
•	Time-based One-Time Password (TOTP) via Authenticator App
•	SMS text message OTP
22.	Save changes and redeploy the Cognito configuration

3.4 MFA Policy Settings

Configuration Parameter	Value
MFA Enforcement	Required for all users
Supported Factor Types	TOTP (Authenticator App), SMS OTP, Email OTP, Backup Codes
OTP Expiry Window	5 minutes (300 seconds)
OTP Code Length	6 digits (numeric)
Backup Codes	10 single-use codes generated at setup
Failed Attempts Before Lockout	5 attempts
Lockout Duration	15 minutes
Remember Device	30 days (trusted devices)
Session Timeout After MFA	8 hours

4. Supported MFA Methods
4.1 TOTP — Authenticator App (Recommended)
Time-based One-Time Passwords (TOTP) follow the RFC 6238 standard. The user scans a QR code during setup to link their authenticator app. Each code is valid for 30 seconds.

Compatible apps:
•	Google Authenticator
•	Microsoft Authenticator
•	Authy
•	1Password (built-in authenticator)

4.2 SMS OTP
A 6-digit numeric code is sent to the user's registered phone number via SMS. The code expires after 5 minutes. SMS OTP is less secure than TOTP (vulnerable to SIM-swapping) but more accessible for users without smartphones.

4.3 Email OTP
A 6-digit code is sent to the user's registered email address. Suitable as a fallback method. Relies on email account security.

4.4 Backup Codes
At MFA enrollment, 10 single-use alphanumeric backup codes are generated and shown once. Users should store them securely (e.g., printed and locked away). Each code can only be used once and cannot be recovered if lost.

5. MFA Test Cases & Results
The following test cases were executed to verify the MFA implementation works correctly under all expected scenarios.

ID	Test Description	Input	Expected	Result
TC01	Valid login + correct TOTP	Correct password + code 847291	Access granted	PASS
TC02	Valid login + correct SMS OTP	Correct password + code 523817	Access granted	PASS
TC03	Valid login + correct Email OTP	Correct password + code 193045	Access granted	PASS
TC04	Valid login + backup code	Correct password + BACKUP1	Access granted	PASS
TC05	Wrong password, no MFA triggered	Wrong password	Login rejected at step 1	PASS
TC06	Correct password + wrong OTP	Correct password + code 000000	MFA verification fails	PASS
TC07	Expired OTP code (after 5 min)	Expired code submitted	Code rejected, re-entry prompted	PASS
TC08	Backup code used twice	Same backup code reused	Second use rejected	PASS
TC09	5 failed OTP attempts	5 wrong codes entered	Account temporarily locked (15 min)	PASS
TC10	MFA method switch	Select SMS after TOTP page	New code sent via SMS	PASS
TC11	Trusted device skip (30 days)	Login from remembered device	MFA skipped, direct access	PASS
TC12	Empty OTP field submitted	Submit with blank code	Validation error, not processed	PASS

Test Summary: 12/12 tests passed (100%)

6. Security Considerations
•	TOTP is the recommended MFA method — it is phishing-resistant and does not require a network connection to generate codes.
•	SMS OTP is vulnerable to SIM-swap attacks. It should be offered as a fallback, not a primary factor.
•	Backup codes must be stored securely. If lost, the user must contact an administrator to reset MFA.
•	All OTP codes are single-use — a code cannot be reused after successful verification.
•	Failed MFA attempts are logged and trigger account lockout after 5 consecutive failures.
•	All MFA events are recorded in CloudTrail (AWS) for audit purposes.
•	Session tokens issued after MFA are short-lived (8 hours) and bound to the user's IP and device fingerprint.

7. Conclusion
MFA has been successfully configured for the CloudSecure cloud-based application using AWS Cognito and IAM. The system supports four factor methods (Authenticator App TOTP, SMS, Email, and Backup Codes) with a deny-by-default policy requiring all users to complete MFA on every login.

All 12 test cases passed, confirming that valid authentication flows grant access, invalid or expired codes are rejected, and brute-force protection mechanisms work correctly. This configuration significantly improves account security compared to password-only authentication.

