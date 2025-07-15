# Bastion-Preflight-Checker
Checks AD and Centrify to confirm Bastion Access for a username

🛡️ Bastion Access Pre-Flight Checker
Bastion Access Pre-Flight Checker is a diagnostic tool designed for environments using Centrify (or now Delinea) and Active Directory to provide controlled SSH access to bastion hosts. It automates pre-access checks to proactively identify common misconfigurations or failures before tenants are onboarded.

This script is ideal for multi-tenant bastion setups where user provisioning and environment inconsistencies can result in delayed or failed access.

🔧 Features
✅ Checks Centrify agent status and domain join

✅ Confirms zone and user mapping to AD

✅ Verifies role assignments (e.g., Login, SSH Access)

✅ Simulates Kerberos authentication (kinit)

✅ Validates user identity via getent

✅ Tests SSH access to bastion

✅ Confirms sudo role assignments

✅ Checks time sync (NTP/Chrony)

✅ Confirms DNS resolution for domain controllers

✅ Outputs structured JSON (optional)

🚀 Quick Start
1. Clone or Download
bash
Copy
Edit
git clone https://your-repo-url/bastion-check.git
cd bastion-check
2. Make Executable
bash
Copy
Edit
chmod +x bastion_check.py
3. Run the Check
bash
Copy
Edit
./bastion_check.py user@yourdomain.com
Optional Flags
Option	Description
--json	Output results to a structured JSON file
--verbose	Show full command output for all checks

Example:
bash
Copy
Edit
./bastion_check.py user@yourdomain.com --json --verbose
📄 Output Example
sql
Copy
Edit
🔍 Running Bastion Pre-Access Check for: user@yourdomain.com

[✅ PASS] Centrify Agent Status
[✅ PASS] Zone Configuration
[✅ PASS] User Zone Mapping
[❌ FAIL] Kerberos Ticket (kinit)
Output:
kinit: Password incorrect

...
📄 JSON output written to bastion_check_user_yourdomain_com_20250714_132055.json
⚙️ Requirements
Python 3.6+

Linux system with:

Centrify (Delinea) tools installed (adinfo, lsa, dzinfo, etc.)

kinit, getent, ssh, ntpstat or chronyc, host utilities

🔍 What It Checks (In Detail)
Check	Purpose
Centrify Agent Status	Ensures the bastion is domain-joined and agent is operational
Zone Configuration	Confirms bastion is assigned to the right Centrify zone
User Zone Mapping	Verifies the user is recognized in the current zone
Role Assignments	Checks AD roles like "Login" or "SSH Access"
Kerberos Ticket (kinit)	Tests if the user can authenticate via Kerberos
getent passwd	Confirms UNIX identity resolution
SSH to Localhost	Tests interactive access simulation
Sudo Role Check	Confirms dzinfo shows sudo privileges if needed
NTP Sync	Validates time sync (required for Kerberos)
DNS Resolution	Ensures domain controller names can be resolved

🛠️ Troubleshooting
Permission denied for user: Ensure user has a valid login role in Centrify.

Kerberos fails: Check password, time sync, or realm config.

getent fails: Confirm NSS and PAM are configured properly for AD users.

SSH test fails: Check sshd_config for AD restrictions.

📌 Notes
Replace ad.example.com in the script with your actual domain controller FQDN in the check_dns() function.

Designed for environments using CentrifyDC, but can be adapted for SSSD or Delinea CLI with minor modifications.

📬 License
This project is provided under the MIT License.

🤝 Contributions
Feel free to submit pull requests for:

Adding support for non-Centrify setups (e.g., SSSD)

Enhanced alerting/logging

Integration with Slack or email
