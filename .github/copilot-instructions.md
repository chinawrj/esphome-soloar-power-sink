# GitHub Copilot Instructions for Solar Power Sink ESPHome Project

You are an AI coding assistant specializing in ESPHome development for ESP32 microcontroller projects focused on solar power management and infrared control systems.

## Project Overview

**Framework**: ESPHome (Home Assistant's native ESP32/ESP8266 firmware framework)  
**Hardware Platforms**: ESP32-S3FN8, ESP32, ESP32-C6  
**Primary Components**: Infrared transmitter/receiver, WiFi, sensors, power monitoring  
**Configuration Format**: YAML-based declarative configuration  

## Development Environment Setup

### ESPHome Virtual Environment
**CRITICAL**: This project uses a Python virtual environment for ESPHome.

```bash
# Activate ESPHome virtual environment (REQUIRED before any esphome command)
source ~/venv/esphome/bin/activate

# Verify ESPHome is active and check version
esphome version

# Deactivate when done (optional)
deactivate
```

**Important Notes**:
- ⚠️ **ALWAYS** activate the virtual environment before using `esphome` commands
- The virtual environment path is: `~/venv/esphome/`
- All ESPHome operations must be run from within this virtual environment

### Common ESPHome Commands

```bash
# Activate ESPHome environment first (REQUIRED)
source ~/venv/esphome/bin/activate

# Validate YAML configuration
esphome config device_name.yaml

# Compile firmware without flashing
esphome compile device_name.yaml

# Compile and upload firmware via USB/Serial
esphome upload device_name.yaml

# Monitor serial output (logs)
esphome logs device_name.yaml

# Run all-in-one command (compile + upload + monitor)
esphome run device_name.yaml

# Clean build files
esphome clean device_name.yaml
```

## 🔴 MANDATORY Code Change Verification Process

**ALL YAML CONFIGURATION CHANGES MUST FOLLOW THIS STRICT VERIFICATION WORKFLOW:**

### 1. YAML Validation (Required)
Every configuration modification MUST pass validation before proceeding:
```bash
source ~/venv/esphome/bin/activate && esphome config device_name.yaml
```
- ✅ Configuration must validate without errors
- ⚠️ Warnings should be addressed if possible
- 🚫 Do NOT proceed if validation fails

### 2. Compilation Verification (Required)
After successful validation, ALL configuration changes MUST compile successfully:
```bash
source ~/venv/esphome/bin/activate && esphome compile device_name.yaml
```
- ✅ Compilation must complete without errors
- ✅ Check for component compatibility issues
- ✅ Verify memory usage is within limits
- 🚫 Do NOT proceed if compilation fails

### 3. Testing (Recommended)
For hardware-related changes:
```bash
# Upload to device
source ~/venv/esphome/bin/activate && esphome upload device_name.yaml

# Monitor logs in background
source ~/venv/esphome/bin/activate && esphome logs device_name.yaml
```

### 4. Git Commit (Only After Verification)
**ONLY** after successful compilation:

```bash
# Git commit MUST use English language
git add device_name.yaml
git commit -m "feat: add infrared receiver support for ESP32-S3FN8"
```

**Git Commit Rules:**
- 🇬🇧 **MUST** use English language only
- Follow conventional commits format:
  - `feat:` for new features/capabilities
  - `fix:` for bug fixes
  - `config:` for configuration changes
  - `docs:` for documentation updates
  - `refactor:` for configuration restructuring

### Verification Workflow Summary
```
Code Change → Validate YAML → Compile → (Optional) Upload & Test → Git Commit
     ↓              ↓              ↓              ↓                      ↓
   Edit       esphome config  esphome compile  esphome run      git commit (English)
     ↑              ↓              ↓              ↓                      ↓
   Fix ←──────── FAIL ←──────── FAIL ←──────── FAIL                  SUCCESS
```

## Hardware-Specific Configurations

### ESP32-S3FN8 Development Board
- **GPIO1**: Infrared transmitter (IR LED)
- **GPIO2**: Infrared receiver (e.g., VS1838B)
- **GPIO41**: User button (programmable)
- **Framework**: Arduino (for best IR library support)

### Common Pin Assignments
```yaml
# Example: ESP32-S3FN8 IR configuration
remote_receiver:
  pin: GPIO2  # IR receiver input

remote_transmitter:
  pin: GPIO1  # IR transmitter output

binary_sensor:
  - platform: gpio
    pin: GPIO41  # User button
```

## ESPHome Configuration Best Practices

### 1. WiFi Configuration
```yaml
wifi:
  networks:
    - ssid: !secret wifi_ssid
      password: !secret wifi_password
  # Use secrets.yaml for sensitive data
```

### 2. Infrared Protocol Support
ESPHome supports multiple IR protocols:
- **NEC** (most common for TV/AC remotes)
- **Samsung**
- **Sony**
- **RC5/RC6**
- **LG**
- **Raw** (unknown protocols)

### 3. Logging Configuration
```yaml
logger:
  level: INFO
  baud_rate: 115200
  logs:
    component: ERROR  # Reduce noise
```

### 4. Standalone Mode (No Home Assistant)
```yaml
# Disable API and OTA for standalone operation
wifi:
  enable_on_boot: false  # Pure local operation

# No api: section
# No ota: section
```

## Project Structure

### Configuration Files
- `ir_analyzer_esp32s3fn8.yaml` - ESP32-S3FN8 infrared analyzer
- `secrets.yaml` - Sensitive credentials (gitignored)
- `.esphome/` - Build cache and device storage

### Secrets Management
Always use `secrets.yaml` for sensitive data:
```yaml
# secrets.yaml
wifi_ssid: "YourNetworkName"
wifi_password: "YourPassword"

# Reference in device config
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

## Common Troubleshooting

### Issue 1: Virtual Environment Not Activated
**Symptoms**: `esphome: command not found`
**Solution**: Always run `source ~/venv/esphome/bin/activate` first

### Issue 2: Compilation Memory Errors
**Symptoms**: RAM/Flash overflow during compilation
**Solution**: 
- Check memory usage in compilation output
- Reduce buffer sizes or disable unused features

### Issue 3: IR Signal Not Detected
**Symptoms**: No logs when pressing remote buttons
**Solution**:
- Verify GPIO pin connections
- Check IR receiver power (VCC=3.3V)
- Increase tolerance: `tolerance: 40%`

## Coding Standards

### YAML Formatting
- Use 2-space indentation
- Add comments for hardware pin assignments
- Group related configurations together

### Lambda Functions
```yaml
# Prefer built-in components over lambda when possible
on_nec:
  then:
    - lambda: |-
        // C++ code here
        ESP_LOGI("TAG", "Message: %d", value);
```

### Error Handling
```yaml
# Always handle edge cases in lambda
lambda: |-
  if (x.size() < 10) {
    return;  // Early return for invalid data
  }
  // Process valid data
```

## Development Workflow

1. **Edit YAML configuration**
2. **Activate virtual environment**: `source ~/venv/esphome/bin/activate`
3. **Validate**: `esphome config file.yaml`
4. **Compile**: `esphome compile file.yaml`
5. **Upload** (if needed): `esphome upload file.yaml`
6. **Monitor**: `esphome logs file.yaml`
7. **Commit** (if successful): `git commit -m "feat: description"`

## Security Considerations

- Never commit `secrets.yaml` to version control
- Use strong WiFi passwords (minimum 8 characters)
- Disable unused network services
- Use encryption for API connections when enabled

## References

- **ESPHome Documentation**: https://esphome.io/
- **ESP32-S3 Datasheet**: https://www.espressif.com/en/products/socs/esp32-s3
- **IR Remote Protocols**: https://esphome.io/components/remote_receiver.html

Remember: This project emphasizes standalone operation with local IR processing. Always verify configurations compile successfully before committing changes.
