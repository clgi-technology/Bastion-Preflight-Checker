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

```
git clone https://your-repo-url/bastion-check.git
cd bastion-check
```

2. Make Executable
```
chmod +x bastion_check.py
```

3. Run the Check

```
./bastion_check.py user@yourdomain.com
```

Optional Flags
Option	Description
--json	Output results to a structured JSON file
--verbose	Show full command output for all checks

Example:

```
./bastion_check.py user@yourdomain.com --json --verbose
```

📄 Output Example

```
🔍 Running Bastion Pre-Access Check for: user@yourdomain.com


[✅ PASS] Centrify Agent Status
[✅ PASS] Zone Configuration
[✅ PASS] User Zone Mapping
[❌ FAIL] Kerberos Ticket (kinit)
Output:
kinit: Password incorrect


...
📄 JSON output written to bastion_check_user_yourdomain_com_20250714_132055.json

```

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


🚀 bastion_check.py
```
#!/usr/bin/env python3
import subprocess
import sys
import argparse
import json
from datetime import datetime

def run_cmd(cmd, capture_output=True):
    try:
        result = subprocess.run(
            cmd,
            shell=True,
            check=False,
            stdout=subprocess.PIPE if capture_output else None,
            stderr=subprocess.STDOUT
        )
        output = result.stdout.decode().strip() if result.stdout else ""
        return result.returncode, output
    except Exception as e:
        return 1, f"Exception: {str(e)}"

def check_agent_status():
    return run_cmd("/opt/centrifydc/bin/adinfo")

def check_zone():
    return run_cmd("/opt/centrifydc/bin/adinfo --zone")

def check_user_zone(user):
    return run_cmd(f"/opt/centrifydc/bin/lsa {user}")

def check_user_roles(user):
    return run_cmd(f"/opt/centrifydc/bin/cdquery -W \"select * from CentrifyUnixRight where UserName = '{user}'\"")

def test_kinit(user):
    return run_cmd(f"echo | kinit {user}", capture_output=True)

def check_getent(user):
    return run_cmd(f"getent passwd {user}")

def check_ssh(user):
    return run_cmd(f"ssh -o BatchMode=yes -o ConnectTimeout=5 {user}@localhost 'exit'", capture_output=True)

def check_sudo_rights(user):
    return run_cmd(f"dzinfo -R {user}")

def check_ntp_sync():
    return run_cmd("chronyc tracking || ntpstat")

def check_dns():
    return run_cmd("host ad.example.com")  # Replace with actual domain controller FQDN

def summarize(name, code, output):
    status = "✅ PASS" if code == 0 else "❌ FAIL"
    print(f"[{status}] {name}")
    if code != 0 or "--verbose" in sys.argv:
        print(f"Output:\n{output}\n")

def main():
    parser = argparse.ArgumentParser(description="Check Bastion Access Readiness")
    parser.add_argument("user", help="Username in user@domain format")
    parser.add_argument("--json", help="Output results to JSON file", action="store_true")
    args = parser.parse_args()

    results = {}

    checks = [
        ("Centrify Agent Status", check_agent_status),
        ("Zone Configuration", check_zone),
        ("User Zone Mapping", lambda: check_user_zone(args.user)),
        ("User Role Assignments", lambda: check_user_roles(args.user)),
        ("Kerberos Ticket (kinit)", lambda: test_kinit(args.user)),
        ("getent passwd", lambda: check_getent(args.user)),
        ("SSH Test to Localhost", lambda: check_ssh(args.user)),
        ("Sudo Role Check", lambda: check_sudo_rights(args.user)),
        ("NTP/Time Sync", check_ntp_sync),
        ("DNS Resolution", check_dns),
    ]

    print(f"🔍 Running Bastion Pre-Access Check for: {args.user}\n")

    for label, check_func in checks:
        code, output = check_func()
        summarize(label, code, output)
        results[label] = {
            "status": "PASS" if code == 0 else "FAIL",
            "output": output
        }

    if args.json:
        ts = datetime.now().strftime("%Y%m%d_%H%M%S")
        file = f"bastion_check_{args.user.replace('@','_')}_{ts}.json"
        with open(file, "w") as f:
            json.dump(results, f, indent=2)
        print(f"\n📄 JSON output written to {file}")

if __name__ == "__main__":
    main()

```
