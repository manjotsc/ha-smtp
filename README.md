# SMTP Integration for Home Assistant

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Integration-41BDF5?style=for-the-badge&logo=homeassistant)](https://www.home-assistant.io/)
[![HACS](https://img.shields.io/badge/HACS-Custom-orange?style=for-the-badge)](https://hacs.xyz/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

> **UI-first SMTP integration** — Configure and manage email notifications entirely through the Home Assistant interface.

---

## Why This Integration?

The built-in Home Assistant SMTP notification requires manual YAML configuration. This integration brings **full UI support** for SMTP setup, making email notifications accessible to everyone:

| Feature | Built-in SMTP | This Integration |
|---------|:-------------:|:----------------:|
| UI Configuration | - | Yes |
| Multiple Accounts | - | Yes |
| Reconfigure Without Restart | - | Yes |
| Diagnostic Sensors | - | Yes |
| Service Selector in UI | - | Yes |

---

## Features

- **Complete UI Configuration** — Set up SMTP accounts through Settings > Devices & Services
- **Multiple Email Accounts** — Add Gmail, Outlook, Yahoo, and others side by side
- **Rich Email Support** — Send HTML emails with Jinja2 templates and attachments
- **Live Diagnostics** — Monitor connection status, errors, and delivery timestamps
- **No Restart Required** — Add, modify, or remove accounts without restarting HA

---

## Quick Start

### Installation via HACS

1. Open **HACS** > **Integrations**
2. Click **⋮** > **Custom repositories**
3. Add this repository URL as **Integration**
4. Search for **SMTP** and install
5. Restart Home Assistant

<details>
<summary><strong>Manual Installation</strong></summary>

1. Download the `smtp` folder from this repository
2. Copy to `config/custom_components/`
3. Restart Home Assistant

```
custom_components/
└── smtp/
    ├── __init__.py
    ├── config_flow.py
    ├── const.py
    ├── manifest.json
    ├── notify.py
    ├── sensor.py
    ├── services.yaml
    ├── strings.json
    └── translations/
        └── en.json
```

</details>

### Setup

1. Go to **Settings** > **Devices & Services**
2. Click **+ Add Integration**
3. Search for **SMTP**
4. Enter your email server details

---

## Configuration Options

| Option | Description |
|--------|-------------|
| **Mail server** | SMTP server address (e.g., `smtp.gmail.com`) |
| **Port** | `587` (STARTTLS), `465` (SSL/TLS), or `25` (none) |
| **Security** | STARTTLS (recommended), SSL/TLS, or None |
| **Login** | Your email username |
| **Password** | Email password or app-specific password |
| **From address** | Sender email address |
| **From name** | Display name shown to recipients |
| **To address** | Default recipient(s), comma-separated |
| **Connection timeout** | Seconds to wait for server response |
| **Verify SSL** | Certificate validation (keep enabled) |
| **Debug logging** | Log SMTP traffic for troubleshooting |

<details>
<summary><strong>Common Provider Settings</strong></summary>

| Provider | Server | Port | Security |
|----------|--------|------|----------|
| Gmail | `smtp.gmail.com` | 587 | STARTTLS |
| Outlook/365 | `smtp.office365.com` | 587 | STARTTLS |
| Yahoo | `smtp.mail.yahoo.com` | 587 | STARTTLS |
| iCloud | `smtp.mail.me.com` | 587 | STARTTLS |

> **Note:** Gmail, Outlook, and other providers require an **App Password** when 2FA is enabled. Generate one in your account security settings.

</details>

---

## Usage

### Sending Emails

Use the `smtp.send_message` service to send emails from automations, scripts, or the developer tools.

**Basic Example:**

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "Home Assistant Alert"
  message: "Motion detected in the living room!"
```

**With Templates:**

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "Daily Report"
  message: |
    Current temperature: {{ states('sensor.temperature') }}°C
    Humidity: {{ states('sensor.humidity') }}%
```

<details>
<summary><strong>HTML Email Example</strong></summary>

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "Home Assistant Status"
  message: "Plain text fallback"
  html: |
    <!doctype html>
    <html>
      <body style="font-family: Arial, sans-serif; padding: 20px;">
        <h1>Home Assistant Status</h1>
        <p><strong>Installed:</strong> {{ state_attr('update.home_assistant_core_update', 'installed_version') }}</p>
        <p><strong>Latest:</strong> {{ state_attr('update.home_assistant_core_update', 'latest_version') }}</p>
      </body>
    </html>
```

</details>

<details>
<summary><strong>Multiple Recipients</strong></summary>

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "Alert"
  message: "This goes to multiple people"
  to:
    - "person1@example.com"
    - "person2@example.com"
```

</details>

<details>
<summary><strong>With Attachments</strong></summary>

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "Camera Snapshot"
  message: "See attached image"
  images:
    - "/config/www/camera_snapshot.jpg"
```

</details>

### Service Parameters

| Parameter | Required | Description |
|-----------|:--------:|-------------|
| `config_entry` | Yes | SMTP account to use (selectable in UI) |
| `message` | * | Plain text body (supports templates) |
| `subject` | No | Email subject line |
| `to` | No | Recipients list (defaults to configured address) |
| `from_name` | No | Override sender display name |
| `html` | * | HTML content (supports templates) |
| `images` | No | List of image paths to attach |

*At least one of `message` or `html` should be provided.

---

## Diagnostic Sensors

Each SMTP account creates diagnostic entities to monitor email delivery:

| Sensor | Description |
|--------|-------------|
| **Status** | `Connected`, `Sending`, or `Error` |
| **Last Error** | Most recent error message |
| **Last Sent** | Timestamp of last successful delivery |

---

## Automation Examples

<details>
<summary><strong>Daily Morning Report</strong></summary>

```yaml
automation:
  - alias: "Daily Status Email"
    trigger:
      - platform: time
        at: "08:00:00"
    action:
      - service: smtp.send_message
        data:
          config_entry: abc123def456
          subject: "Good Morning Report"
          message: |
            Temperature: {{ states('sensor.temperature') }}°C
            Humidity: {{ states('sensor.humidity') }}%
            Weather: {{ states('weather.home') }}
```

</details>

<details>
<summary><strong>Motion Detection Alert</strong></summary>

```yaml
automation:
  - alias: "Motion Alert Email"
    trigger:
      - platform: state
        entity_id: binary_sensor.front_door_motion
        to: "on"
    action:
      - service: smtp.send_message
        data:
          config_entry: abc123def456
          subject: "Motion Detected!"
          message: "Motion at front door — {{ now().strftime('%H:%M:%S') }}"
```

</details>

---

## Troubleshooting

<details>
<summary><strong>Connection Issues</strong></summary>

1. Verify server address and port
2. Check if your provider requires an app password
3. Try switching between STARTTLS and SSL/TLS
4. Enable debug logging to inspect SMTP traffic

</details>

<details>
<summary><strong>Authentication Errors</strong></summary>

- **Gmail:** Generate an [App Password](https://support.google.com/accounts/answer/185833)
- **Outlook:** Create an [App Password](https://support.microsoft.com/account-billing/using-app-passwords-5896ed9b-4263-e681-128a-a6f2979a7944)
- Ensure 2FA is enabled before generating app passwords

</details>

<details>
<summary><strong>Emails Not Arriving</strong></summary>

1. Check spam/junk folder
2. Verify recipient address
3. Check the **Last Error** sensor
4. Review Home Assistant logs

</details>

---

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the [MIT License](LICENSE).
