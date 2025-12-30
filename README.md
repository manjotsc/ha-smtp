# SMTP Integration for Home Assistant

A Home Assistant custom integration for sending email notifications via SMTP with full UI configuration support.

## Features

- **UI-Based Setup**: Configure everything through the Home Assistant interface - no YAML needed
- **Multiple Accounts**: Add multiple SMTP accounts for different email providers
- **Template Support**: Use Jinja2 templates in messages to include sensor data
- **HTML Emails**: Send rich HTML emails with dynamic content
- **Attachments**: Attach images to your emails
- **Diagnostic Sensors**: Monitor connection status, errors, and last sent time

## Installation

### HACS (Recommended)

1. Open HACS in Home Assistant
2. Click on "Integrations"
3. Click the three dots menu → "Custom repositories"
4. Add this repository URL and select "Integration" as the category
5. Click "Install"
6. Restart Home Assistant

### Manual Installation

1. Download the `smtp` folder from this repository
2. Copy it to your `custom_components` directory
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

## Configuration

1. Go to **Settings** → **Devices & Services**
2. Click **+ Add Integration**
3. Search for "SMTP"
4. Fill in your email server details:

| Field | Description |
|-------|-------------|
| Mail server | SMTP server address (e.g., smtp.gmail.com) |
| Port | Server port (587 for STARTTLS, 465 for SSL/TLS, 25 for none) |
| Security | Encryption type: STARTTLS (recommended), SSL/TLS, or None |
| Login | Your email username |
| Password | Your email password or app-specific password |
| From address | Email address to send from |
| From name | Display name shown in emails (optional) |
| To address | Default recipient(s), comma-separated |
| Connection timeout | Seconds to wait for server response |
| Verify SSL certificate | Keep enabled for security |
| Enable debug logging | Log SMTP communication for troubleshooting |

### Common SMTP Settings

| Provider | Server | Port | Security |
|----------|--------|------|----------|
| Gmail | smtp.gmail.com | 587 | STARTTLS |
| Outlook/Office 365 | smtp.office365.com | 587 | STARTTLS |
| Yahoo | smtp.mail.yahoo.com | 587 | STARTTLS |
| iCloud | smtp.mail.me.com | 587 | STARTTLS |

> **Note**: Gmail and other providers require an "App Password" instead of your regular password. Enable 2FA and generate an app password in your account settings.

## Usage

### Service: `smtp.send_message`

Send emails using the `smtp.send_message` service.

#### Basic Example

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456  # Your SMTP config entry ID
  subject: "Home Assistant Alert"
  message: "Motion detected in the living room!"
```

#### With Templates

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "🏠 Home Assistant Status"
  message: >
    Home Assistant is running version
    {{ state_attr('update.home_assistant_core_update', 'installed_version') }}
```

#### HTML Email with Sensor Data

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "🏠 Home Assistant Version & Update Status"
  message: "Plain text fallback for email clients that don't support HTML."
  html: |
    <!doctype html>
    <html>
      <body style="margin:0;padding:16px;font-family:Arial,sans-serif;">
        <h1>Home Assistant Status</h1>
        <p>Installed: {{ state_attr('update.home_assistant_core_update', 'installed_version') }}</p>
        <p>Latest: {{ state_attr('update.home_assistant_core_update', 'latest_version') }}</p>
      </body>
    </html>
```

#### Multiple Recipients

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

#### With Attachments

```yaml
service: smtp.send_message
data:
  config_entry: abc123def456
  subject: "Camera Snapshot"
  message: "See attached image"
  images:
    - "/config/www/camera_snapshot.jpg"
```

### Service Fields

| Field | Required | Description |
|-------|----------|-------------|
| `config_entry` | Yes | The SMTP account to send from (use dropdown in UI) |
| `message` | No* | Plain text body. Supports templates. |
| `subject` | No | Subject line. Supports templates and emojis. |
| `to` | No | Recipients list. Leave empty for default from setup. |
| `from_name` | No | Override the "From" display name. |
| `html` | No* | HTML content. Supports Jinja2 templates. |
| `images` | No | List of image file paths to attach. |

*Either `message` or `html` should be provided.

## Diagnostic Sensors

Each SMTP account creates three diagnostic sensors:

| Sensor | Description |
|--------|-------------|
| Status | Current state: "Connected", "Sending", or "Error" |
| Last error | Most recent error message, or "None" |
| Last sent | Timestamp of last successful email |

## Automation Examples

### Send Daily Report

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
          subject: "🌅 Good Morning Report"
          message: >
            Temperature: {{ states('sensor.temperature') }}°C
            Humidity: {{ states('sensor.humidity') }}%
```

### Alert on Motion

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
          subject: "🚨 Motion Detected!"
          message: "Motion detected at front door at {{ now().strftime('%H:%M:%S') }}"
```

## Troubleshooting

### Connection Issues

1. Verify your server address and port
2. Check if your email provider requires an app password
3. Try different security settings (STARTTLS vs SSL/TLS)
4. Enable debug logging to see detailed SMTP communication

### Authentication Errors

- Gmail: Use an [App Password](https://support.google.com/accounts/answer/185833)
- Outlook: Use an [App Password](https://support.microsoft.com/en-us/account-billing/using-app-passwords-with-apps-that-don-t-support-two-step-verification-5896ed9b-4263-e681-128a-a6f2979a7944)
- Check if 2FA is enabled on your account

### Emails Not Arriving

1. Check spam/junk folder
2. Verify the recipient address
3. Check the "Last error" sensor for issues
4. Review Home Assistant logs for SMTP errors

## License

This project is licensed under the MIT License.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.
