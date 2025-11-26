# Wazuh → Pushover Integration

Send Wazuh alerts directly to **Pushover** without SMTP, email relays, or third-party webhook services.  
This custom integration script plugs directly into the Wazuh Manager and supports:

- Multiple independent notification channels  
- Clean, readable alert formatting  
- Office 365–aware parsing  
- User-specific severity thresholds  
- Group or per-user notifications  
- Fully JSON-driven configuration  

---

## Features

- **Direct delivery** to Pushover API  
- **Multiple recipients** supported through:
  - Multiple `<integration>` blocks, or
  - A single Pushover **Group Key**
- **Severity-based routing** (e.g., level ≥ 6 to admin, level ≥ 3 to secondary user)
- **Readable notifications**, e.g.:

```
[High] Wazuh alert 
Office 365: Sent message using different permissions
Severity: High (level 6)
Rule: 91708 | Agent: wazuh-server/000

User: someone@example.com
Operation: SendAs
Workload: Exchange
Client IP: 203.0.113.10
```

- Optional debug logging to `/tmp/pushover_integration_debug.log`  
- Works on any Wazuh 4.x Manager installation  

---

## Requirements

- Wazuh Manager 4.x  
- Python 3  
- Pushover **Application Token**  
- One or more Pushover **User Keys** or **Group Keys**  

---

## Installation

### 1. Install the integration script

Save the script as:

```
/var/ossec/integrations/custom-pushover
```

Set permissions:

```bash
sudo chmod 750 /var/ossec/integrations/custom-pushover
sudo chown root:wazuh /var/ossec/integrations/custom-pushover
```

(Some systems use group `ossec` instead of `wazuh`.)

---

### 2. Add integration blocks to `ossec.conf`

Edit:

```
/var/ossec/etc/ossec.conf
```

Insert your integrations **before** `</ossec_config>`.

#### Primary user (level ≥ 6)

```xml
<integration>
  <name>custom-pushover</name>
  <hook_url>https://api.pushover.net/1/messages.json</hook_url>
  <api_key>PUSHOVER_APP_TOKEN</api_key>
  <level>6</level>
  <alert_format>json</alert_format>
  <options>
    {
      "user": "PUSHOVER_USER_KEY_PRIMARY",
      "title": "Wazuh alert",
      "priority": 0
    }
  </options>
</integration>
```

#### Secondary user (level ≥ 3)

```xml
<integration>
  <name>custom-pushover-secondary</name>
  <hook_url>https://api.pushover.net/1/messages.json</hook_url>
  <api_key>PUSHOVER_APP_TOKEN</api_key>
  <level>3</level>
  <alert_format>json</alert_format>
  <options>
    {
      "user": "PUSHOVER_USER_KEY_SECONDARY",
      "title": "Wazuh alert - Secondary",
      "priority": 0
    }
  </options>
</integration>
```

You may add as many blocks as you want (critical-only, on-call team, etc.).

---

### 3. Create symlinks for each integration name

Wazuh requires a script that matches each `<name>`.

```bash
cd /var/ossec/integrations

sudo ln -s custom-pushover custom-pushover
sudo ln -s custom-pushover custom-pushover-secondary

sudo chown root:wazuh custom-pushover custom-pushover-secondary
sudo chmod 750 custom-pushover custom-pushover-secondary
```

This allows multiple integrations to share the same script.

---

### 4. Restart Wazuh Manager

```bash
sudo systemctl restart wazuh-manager
```

---

## Optional: Use Pushover Group Keys

Instead of multiple integrations, you can create a delivery group in Pushover and use:

```json
"user": "PUSHOVER_GROUP_KEY"
```

One integration block → many recipients.

---

## Testing & Verification

Check Wazuh integration logs:

```bash
sudo tail -f /var/ossec/logs/integrations.log
```

Check script debug output:

```bash
sudo tail -f /tmp/pushover_integration_debug.log
```

Trigger a test alert:

- Incorrect SSH login to an agent  
- Restart a monitored service  
- Temporarily set `<level>1</level>` for high-volume testing  

You should see both logs populate and Pushover notifications arrive.

---

## Troubleshooting

### “Invalid element in the configuration”
Occurs when:

- An `<integrator>` block is added to `ossec.conf` (remove it)  
- `<name>` does **not** start with `custom-`  
- Script name does **not** match the integration `<name>`

### “Missing pushover_user/app_token”
The `<options>` JSON must include `"user"`.

### One user not receiving alerts
Use:

- Separate `<integration>` blocks **or**
- One Pushover **Group Key**

---

## License

MIT License — free to use, modify, and redistribute.

---

