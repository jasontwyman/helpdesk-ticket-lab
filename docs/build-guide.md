# Build Guide — Reproducible osTicket Portion (AD Prerequisite)

This guide reproduces the **osTicket portion** of the help-desk lab on an isolated VirtualBox
network. It assumes the Active Directory environment already exists; build or review that
prerequisite in the companion
**[Active Directory Home Lab](https://github.com/jasontwyman/active-directory-home-lab)**.

All example users, names, messages, and tickets must be synthetic. The guide does not reproduce
or claim a Windows client validation workflow, and it does not turn the illustrative ticket
narratives into live incidents.

## Scope and version policy

* **Tested lab snapshot:** Ubuntu Server 24.04 LTS, osTicket 1.18.4, Apache 2.4,
  MariaDB 10.11, and PHP 8.3 were observed in the lab.
* **Pinned here:** the osTicket download URL pins release **1.18.4** so the application portion
  is repeatable.
* **Not pinned here:** Ubuntu `apt` packages resolve to the supported versions in the enabled
  Ubuntu repositories at build time. Record the installed versions if exact reconstruction is
  required; do not assume every future 24.04 install returns the same patch release.
* **Prerequisite:** `DC01` and DNS for `jtwyman.test` already operate on VirtualBox internal
  network `intnet` at `10.10.10.10/24`.
* **Client boundary:** the companion Active Directory repository now has separate Windows 11
  domain-join and client-policy evidence. This guide does not treat that later `H:`/`tuser`
  validation as retroactive proof of the synthetic HD-004 `S:`/`mlee` ticket.

This is a lab build, not a production deployment. HTTPS, a documented host-firewall policy,
backups/restore testing, monitoring, mail delivery, and enterprise secret management are
**production-hardening work not implemented here**.

---

## 1. Architecture

```text
VirtualBox internal network "intnet" — 10.10.10.0/24

DC01 / AD DS + DNS                    helpdesk / osTicket
10.10.10.10                           10.10.10.30
(existing prerequisite)              (built by this guide)

Windows 11 endpoint: validated in the companion AD repository; not used as
retroactive evidence for this repository's original HD-004 ticket narrative
```

The helpdesk VM starts with two adapters:

1. **NIC 1 — NAT, temporary:** package and application download access, plus host-only port
   forwards during setup.
2. **NIC 2 — `intnet`, permanent:** isolated communication with the AD lab.

After installation, shut down the guest cleanly and remove NIC 1. The final VM has no bridged
adapter and no default route. This is useful lab isolation; it is not a substitute for a
production firewall or security architecture.

---

## 2. Host shell context and VirtualBox executable

Run `VBoxManage` commands on the **Windows host**, not inside Ubuntu. The examples below use
**Git Bash** syntax. Quote the executable variable because the default path contains spaces:

```bash
VBOXMANAGE='C:/Program Files/Oracle/VirtualBox/VBoxManage.exe'
VM='helpdesk'
VM_DIR='C:/Users/<host-user>/VirtualBox VMs/helpdesk'
ISO='C:/path/to/ubuntu-24.04.x-live-server-amd64.iso'

"$VBOXMANAGE" --version
```

If `VBoxManage.exe` is already on `PATH`, this is also valid:

```bash
VBOXMANAGE='VBoxManage'
"$VBOXMANAGE" --version
```

PowerShell and Command Prompt have different variable/quoting syntax; do not paste the Git Bash
snippets into those shells unchanged.

Create the VM and storage:

```bash
"$VBOXMANAGE" createvm --name "$VM" --ostype Ubuntu24_LTS_64 --register
"$VBOXMANAGE" modifyvm "$VM" --memory 2560 --cpus 1 --vram 16 --firmware efi \
  --paravirtprovider default --graphicscontroller vmsvga
"$VBOXMANAGE" createmedium disk --filename "$VM_DIR/$VM.vdi" --size 25000 --format VDI
"$VBOXMANAGE" storagectl "$VM" --name SATA --add sata --controller IntelAhci \
  --portcount 2 --bootable on
"$VBOXMANAGE" storageattach "$VM" --storagectl SATA --port 0 --device 0 \
  --type hdd --medium "$VM_DIR/$VM.vdi"

# NIC 1: temporary NAT and host port forwards
"$VBOXMANAGE" modifyvm "$VM" --nic1 nat --nictype1 82540EM
"$VBOXMANAGE" modifyvm "$VM" --natpf1 'ssh,tcp,127.0.0.1,12122,,22'
"$VBOXMANAGE" modifyvm "$VM" --natpf1 'http,tcp,127.0.0.1,18080,,80'

# NIC 2: permanent isolated segment
"$VBOXMANAGE" modifyvm "$VM" --nic2 intnet --intnet2 'intnet' --nictype2 82540EM
```

On the tested Windows host, VirtualBox used the Hyper-V/NEM backend. A two-vCPU Linux guest
soft-locked during installation, while the one-vCPU configuration completed. Treat `--cpus 1`
as a tested host-specific workaround, not a universal VirtualBox requirement.

Install Ubuntu Server without placing the password in the process arguments or shell history.
VirtualBox 7.2 uses `--user-password` / `--user-password-file`; the example reads a private value
silently and supplies it through standard input:

```bash
read -rsp 'Temporary Ubuntu install password: ' INSTALL_PASSWORD
printf '\n'
printf '%s' "$INSTALL_PASSWORD" | \
"$VBOXMANAGE" unattended install "$VM" --iso="$ISO" \
  --user=helpdesk --user-password-file=stdin \
  --full-user-name='Help Desk Admin' --hostname=helpdesk.lab \
  --locale=en_US --country=US --time-zone=UTC --start-vm=headless
unset INSTALL_PASSWORD
```

Use a unique temporary value and rotate/remove it after setup. The prompt keeps it out of the
command line and history; do not echo, log, or commit it. Let partitioning, package work, and
reboot finish; do not force power off during unattended upgrades.

If an interrupted installation leaves package management incomplete, inspect the current apt
configuration before changing it, then use the appropriate recovery commands:

```bash
# Ubuntu guest shell
sudo dpkg --configure -a
sudo apt-get -f install
```

Do not blindly delete repository entries; confirm a stale CD-ROM source actually exists first.

---

## 3. Discover interfaces and configure `intnet`

The guest interface names can differ. Do not assume the internal adapter is always `enp0s8`.
From the Ubuntu guest, discover names, addresses, MAC addresses, and routes:

```bash
ip -br link
ip -br address
ip route
```

From the Windows host, compare the VM adapter MAC addresses when needed:

```bash
"$VBOXMANAGE" showvminfo "$VM" --machinereadable | \
  grep -E '^(macaddress|nic|intnet)[12]='
```

Identify the adapter attached to `intnet`, then set it explicitly. The example below assumes
discovery showed `enp0s8`; replace that name if yours differs.

```yaml
# /etc/netplan/99-intnet.yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: false
      addresses:
        - 10.10.10.30/24
      nameservers:
        addresses:
          - 10.10.10.10
        search:
          - jtwyman.test
```

```bash
# Ubuntu guest shell
sudo chmod 0600 /etc/netplan/99-intnet.yaml
sudo netplan generate
sudo netplan apply
ip -br address show enp0s8
ping -c 2 10.10.10.10
getent hosts dc01.jtwyman.test
```

A successful ping proves IP reachability only. DNS resolution is a separate test.

---

## 4. Install and test the LAMP prerequisites

Run these commands in the **Ubuntu guest shell** while temporary NAT is present:

```bash
sudo apt-get update
sudo apt-get install -y apache2 mariadb-server unzip wget ca-certificates \
  php libapache2-mod-php php-cli php-common php-mysql php-gd php-imap \
  php-mbstring php-xml php-intl php-curl php-apcu php-zip php-bcmath
sudo systemctl enable --now mariadb apache2
```

Record versions and test services, Apache configuration, and required PHP modules:

```bash
lsb_release -ds
apache2 -v
mariadb --version
php --version
sudo systemctl is-active apache2 mariadb
sudo apache2ctl configtest

required='mysqli gd imap mbstring intl curl apcu zip bcmath xml dom fileinfo'
installed="$(php -m | tr '[:upper:]' '[:lower:]')"
for module in $required; do
  printf '%-12s ' "$module"
  printf '%s\n' "$installed" | grep -qx "$module" && echo OK || echo MISSING
 done
```

Resolve every `MISSING` module before continuing. `Syntax OK` and active services validate
configuration/startup; they do not by themselves prove the application workflow.

---

## 5. Create and test the MariaDB database

Invoke MariaDB explicitly as the local administrative user:

```bash
sudo mariadb
```

At the `MariaDB>` prompt, run:

```sql
CREATE DATABASE osticket CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'osticket'@'localhost' IDENTIFIED BY '<unique-database-password>';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticket'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Do not commit the database password. Test that the database exists and that MariaDB answers:

```bash
sudo mariadb --batch --skip-column-names \
  --execute="SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME='osticket';"
sudo mariadb --execute='SELECT VERSION();'
```

Optionally test the application account interactively with `mariadb -u osticket -p osticket` so
the password is not exposed in the process list or shell history.

---

## 6. Download, verify, and deploy osTicket

```bash
cd /tmp
OSTICKET_VERSION='1.18.4'
OSTICKET_ARCHIVE="osTicket-v${OSTICKET_VERSION}.zip"
OSTICKET_URL="https://github.com/osTicket/osTicket/releases/download/v${OSTICKET_VERSION}/${OSTICKET_ARCHIVE}"
wget --https-only "$OSTICKET_URL" -O "$OSTICKET_ARCHIVE"
```

Verification guidance:

1. Confirm the final URL and release tag are the intended official osTicket GitHub release.
2. If the publisher supplies an authenticated checksum/signature for this release, compare it
   before extraction. This repository does **not** provide or invent a digest.
3. `sha256sum "$OSTICKET_ARCHIVE"` can record the exact artifact you used, but a locally
   generated digest alone does not authenticate the download.
4. Test archive integrity before deployment:

```bash
unzip -t "$OSTICKET_ARCHIVE"
rm -rf /tmp/osticket-release
mkdir /tmp/osticket-release
unzip -q "$OSTICKET_ARCHIVE" -d /tmp/osticket-release
```

Deploy with Apache-readable defaults. The installer needs temporary write access only to
`ost-config.php`:

```bash
sudo install -d -o root -g www-data -m 0750 /var/www/osticket
sudo cp -a /tmp/osticket-release/upload/. /var/www/osticket/
sudo chown -R root:www-data /var/www/osticket
sudo find /var/www/osticket -type d -exec chmod 0750 {} +
sudo find /var/www/osticket -type f -exec chmod 0640 {} +
sudo cp /var/www/osticket/include/ost-sampleconfig.php \
  /var/www/osticket/include/ost-config.php
sudo chown root:www-data /var/www/osticket/include/ost-config.php
sudo chmod 0660 /var/www/osticket/include/ost-config.php  # installer window only
```

Configure `/etc/apache2/sites-available/000-default.conf`:

```apache
<VirtualHost *:80>
    ServerName helpdesk.lab
    DocumentRoot /var/www/osticket

    <Directory /var/www/osticket>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Enable rewriting and validate before reloading:

```bash
sudo a2enmod rewrite
sudo apache2ctl configtest
sudo systemctl reload apache2
sudo systemctl is-active apache2
curl -I http://127.0.0.1/setup/install.php
```

---

## 7. Complete the web installer and lock down configuration

While temporary NAT and the host forward are active, browse from the Windows host to:

`http://127.0.0.1:18080/setup/install.php`

Use synthetic system/admin identity data and a unique lab credential. Configure database host
`localhost`, database `osticket`, user `osticket`, the database password created above, and
prefix `ost_`.

Immediately after a successful install, restore least-privilege configuration ownership and
remove the installer:

```bash
sudo chown root:www-data /var/www/osticket/include/ost-config.php
sudo chmod 0640 /var/www/osticket/include/ost-config.php
sudo rm -rf /var/www/osticket/setup

stat -c '%U:%G %a %n' /var/www/osticket/include/ost-config.php
test ! -e /var/www/osticket/setup && echo 'setup directory removed'
sudo apache2ctl configtest
sudo systemctl is-active apache2 mariadb
```

The expected `stat` result is `root:www-data 640`. Do not leave the whole web root owned by the
web-service account and do not leave `ost-config.php` installer-writable.

During the temporary-NAT phase, the staff login is:

`http://127.0.0.1:18080/scp/`

---

## 8. Configure and seed synthetic case studies

In **Admin Panel → Manage → Help Topics**, add the lab categories:

* Account & Access
* New Hire Onboarding
* Network & Remote Access
* Security Incident

Optional lab configuration includes a second synthetic agent, a Security department/team, and
SLA plans. Seed the seven **synthetic case-study records** without importing real names,
messages, or addresses.

The repository screenshots show the seeded osTicket records as **Open / Unassigned**. Do not
label those records closed/resolved, and do not claim replies, assignments, escalations, or SLA
completion unless the corresponding osTicket state is captured and verified.

---

## 9. AD exercises and evidence boundaries

The AD prerequisite is outside this guide. If you repeat the portfolio exercises, use only
disposable synthetic accounts/groups and read the companion AD lab first.

* **HD-001 through HD-003:** controlled exercises were executed on the lab domain controller.
  Capture only the output needed to support the stated action, and sanitize secrets.
* **HD-004:** treat this as a configuration diagnostic. `Get-GPO` and
  `Get-GPPermission` show GPO inventory and permissions; they do not prove endpoint policy
  application or drive mapping. A verified remediation would require an intended domain client,
  `gpresult`/RSoP evidence, and a post-refresh mapped-drive check. That evidence is not present.
* **HD-005 through HD-007:** illustrative documentation only; do not manufacture execution
  evidence for VPN, MFA, malware, mail, or Security-team workflows.

### Cleanup discipline for any repeated AD mutation

Before changing AD state, record the exact original membership/account state and use uniquely
owned disposable objects. After the exercise, restore that state and query it again. Keep the
commands, timestamps, and before/after output needed for review, but redact passwords, recovery
codes, tokens, and other secrets. A screenshot of a command or configuration is not proof of a
client outcome. Do not alter production accounts, and do not delete shared lab objects merely to
make a cleanup claim.

### Post-review cleanup performed for this portfolio

After review identified that an onboarding credential had been published and that the
cross-department membership was still standing, DC01 was remediated directly: the exposed
`kross` credential was reset through an undisclosed `SecureString` variable, `kross` was
disabled, and `nalvarez` was removed from `SG-Finance`. The evidence shows the reset invocation
without a visible error and the final queried state; it does not independently verify the
replacement value or how it was generated. See
`evidence/screenshots/18-ad-remediation-cleanup.png`.

This cleanup verifies the final directory state only. It does not prove a secure onboarding
handoff, first sign-in, resource ACLs, or effective share access.

---

## 10. Remove temporary elevation artifacts

An unattended or automation workflow may have created a temporary passwordless sudo rule such
as `/etc/sudoers.d/99-helpdesk-nopasswd`. Do **not** delete a guessed path blindly.

First discover and inspect the exact NOPASSWD include; do not assume the example filename. Keep a
separate root shell open as a recovery path. Move only the confirmed temporary file out of the
included directory, validate sudoers, and then prove normal password-based sudo works while the
temporary rule is inactive. If validation fails, restore the file from the root shell.

```bash
# Review matches manually and set RULE to the one confirmed as build-only.
sudo grep -RIl -- 'NOPASSWD' /etc/sudoers /etc/sudoers.d
RULE='/etc/sudoers.d/<confirmed-build-only-file>'
sudo test -f "$RULE"
sudo sed -n '1,120p' "$RULE"
sudo visudo -cf "$RULE"

# Keep a separate root shell open before this move.
BACKUP="/root/$(basename "$RULE").disabled"
sudo mv -- "$RULE" "$BACKUP"
sudo visudo -c
sudo -k
sudo -v                         # must now use the intended non-temporary admin path

# Delete only after sudo -v and visudo both succeed with the temporary rule inactive.
sudo rm -- "$BACKUP"
sudo visudo -c
sudo test ! -e "$RULE"
```

Do not paste the block unattended: each validation gates the next step. If the file does not
exist, its purpose is uncertain, `visudo` fails, or password-based `sudo -v` does not succeed,
stop and restore the moved file from the open root shell rather than risking administrative
lockout.

---

## 11. Clean shutdown, remove NAT, and verify exact isolation

First request an orderly ACPI shutdown from the Windows host. Do not use VirtualBox `poweroff`
for routine shutdown:

```bash
"$VBOXMANAGE" controlvm "$VM" acpipowerbutton
```

Wait for the guest to reach `poweroff` state and verify it before modifying adapters:

```bash
"$VBOXMANAGE" showvminfo "$VM" --machinereadable | grep '^VMState='
# Continue only when the output is exactly: VMState="poweroff"
```

Delete the temporary forwards, disable NIC 1, and explicitly preserve NIC 2 as `intnet`:

```bash
"$VBOXMANAGE" modifyvm "$VM" --natpf1 delete ssh
"$VBOXMANAGE" modifyvm "$VM" --natpf1 delete http
"$VBOXMANAGE" modifyvm "$VM" --nic1 none
"$VBOXMANAGE" modifyvm "$VM" --nic2 intnet --intnet2 'intnet'

"$VBOXMANAGE" showvminfo "$VM" --machinereadable | \
  grep -E '^(nic1|nic2|intnet2|Forwarding\([0-9]+\))='
```

Verify the reported values, not just command exit status:

* `nic1="none"`
* `nic2="intnet"`
* `intnet2="intnet"`
* no `Forwarding(...)` entries

Start the VM and verify inside Ubuntu:

```bash
ip -br address
ip route
```

The intended steady state is `10.10.10.30/24` on the internal adapter, a connected
`10.10.10.0/24` route, and **no `default` route**. Also verify the service from another machine
on `intnet`:

* `http://10.10.10.30/`
* or `http://helpdesk.<lab-dns-name>/` after creating and testing an appropriate lab DNS record

The old host URL `http://127.0.0.1:18080/` should no longer work because NIC 1 and its forward
were removed. Confirm both the positive internal-network access test and the negative host-port
forward test. Do not describe the VM as isolated if a bridged/NAT adapter or default route
remains.

For this portfolio's final host-side check, VirtualBox reported `nic1="none"`,
`nic2="intnet"`, and `intnet2="intnet"`. The sanitized captured output is stored at
`evidence/config/helpdesk-vm-network.txt`. Guest-side route and service checks remain separate
acceptance criteria and should not be inferred from the host adapter query.

---

## 12. Final validation checklist

* [ ] `apache2ctl configtest` returns `Syntax OK`.
* [ ] `systemctl is-active apache2 mariadb` reports both services active.
* [ ] Every required PHP module reports `OK`.
* [ ] MariaDB contains the `osticket` schema and answers a version query.
* [ ] `ost-config.php` is exactly `root:www-data` mode `0640`.
* [ ] `/var/www/osticket/setup` is absent.
* [ ] Temporary NOPASSWD sudoers content is absent **only after** verifying the intended admin path.
* [ ] VirtualBox reports NIC 1 `none`, NIC 2 `intnet`, no NAT forwards, and the correct internal-network name.
* [ ] The guest has `10.10.10.30/24`, only the intended connected lab route, and no default route.
* [ ] osTicket is reachable from `intnet` by `10.10.10.30` or tested lab DNS.
* [ ] Seed data is synthetic and osTicket workflow states are described exactly as captured.
* [ ] Any repeated AD mutations were restored and verified; evidence limitations are documented.

## Production-hardening backlog (not implemented)

A real deployment would still need, at minimum: HTTPS and certificate lifecycle management; an
explicit host and network firewall policy; encrypted backups plus successful restore tests;
monitoring/logging and alerting; patch and change management; secrets rotation and storage;
mail transport protections; and documented availability/recovery objectives. These are not
implemented or evidenced by this lab guide.
