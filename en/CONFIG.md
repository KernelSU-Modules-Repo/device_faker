# Configuration Guide

## Configuration File Path
- `/data/adb/device_faker/config/config.toml`

The configuration file uses TOML format.

## Global Settings

### default_mode (Global Default Mode)

```toml
default_mode = "lite"  # Recommended: lightweight mode (better stealth)
# default_mode = "full"  # Full mode (may be detected)
```

**Available values**:
- `"lite"` - Lightweight mode (recommended) ⭐
  - Only modifies Build class static fields
  - Unloads module after completion
  - Harder to detect
  - Suitable for 90% of applications

- `"full"` - Full mode
  - Modifies Build class + spoofs SystemProperties
  - Module stays in memory
  - May be detected
  - Only use when lite mode is insufficient

- `"resetprop"` - Resetprop mode
  - Uses resetprop tool to modify properties
  - Supports modifying read-only properties (such as `ro.build.characteristics`)

### debug (Debug Mode)

```toml
debug = true  # Enable detailed logging (for debugging)
# debug = false  # Or delete this line, default off (normal use)
```

**Description**:
- When enabled, outputs detailed Info level logs
- When disabled, only outputs Error level logs
- Recommended to disable for normal use to improve stealth

## Editing Configuration

### Method One: Device Templates

Define a `packages` list in the template, automatically apply to all package names:

```toml
# Define template and list package names
[templates.redmagic_9_pro]
packages = [
    "com.mobilelegends.mi",
    "com.supercell.brawlstars",
    "com.blizzard.diablo.immortal",
]
manufacturer = "ZTE"
brand = "nubia"
model = "NX769J"
device = "REDMAGIC 9 Pro"
fingerprint = "nubia/NX769J/NX769J:14/UKQ1.230917.001/20240813.173312:user/release-keys"

[templates.pixel_xl]
packages = [
    "com.google.android.apps.photos",
]
manufacturer = "Google"
brand = "google"
model = "marlin"
device = "Pixel XL"
product = "marlin"
fingerprint = "google/marlin/marlin:10/QP1A.191005.007.A3/5972272:user/release-keys"

# No need to write [[apps]], all package names automatically use this template
```

**Advantages**:
- ✅ Centralized management of device models and package names
- ✅ No need to repeatedly write [[apps]]
- ✅ Clear at a glance which applications use which template

### Method Two: Direct Configuration

Use [[apps]] to specify device information for individual applications:

```toml
[[apps]]
package = "com.omarea.vtools"
manufacturer = "Xiaomi"
brand = "Xiaomi"
model = "2509FPN0BC"
device = "Xiaomi 15 Pro"
product = "popsicle"
name = "popsicle"  # Product internal name (only effective in full mode)
mode = "full"  # Optional: override global mode

[[apps]]
package = "com.coolapk.market"
manufacturer = "Nothing"
brand = "Nothing"
marketname = "Nothing Phone (3)"
model = "A024"
```

**Advantages**:
- ✅ Flexible configuration
- ✅ Suitable for one-time configuration or template override

**Template Override**:
If a package name is both in the template's `packages` list and has [[apps]] configuration, [[apps]] takes priority:

```toml
[templates.redmagic_9_pro]
packages = [
    "com.mobilelegends.mi",  # Uses this template by default
]
manufacturer = "ZTE"
model = "NX769J"

[[apps]]
package = "com.mobilelegends.mi"  # Overrides template configuration
manufacturer = "Samsung"
model = "SM-S9280"
```

**Field Priority**:
```
[[apps]] direct configuration > template packages list > global default_mode
```

**Mode Priority**:
```
[[apps]].mode > [templates].mode > global default_mode
```

### Application Configuration Field Description

**Field and System Property Mapping Relationship**:
| Field | lite Mode | full Mode (SystemProperties) | Description |
|-------|----------|------------------------------|-------------|
| `manufacturer` | `Build.MANUFACTURER` | + `ro.product.manufacturer` | Manufacturer (e.g.: Xiaomi, Samsung) |
| `brand` | `Build.BRAND` | + `ro.product.brand` | Brand (e.g.: Redmi, nubia) |
| `model` | `Build.MODEL` | + `ro.product.model` | Model (e.g.: 25010PN30C, NX769J) |
| `device` | `Build.DEVICE` | (Build fields only) | Code name (e.g.: xuanyuan, NX769J) |
| `product` | `Build.PRODUCT` | (Build fields only) | Code name (e.g.: xuanyuan, NX769J) |
| `fingerprint` | `Build.FINGERPRINT` | + `ro.build.fingerprint` | Fingerprint |
| `name` | ❌ | `ro.product.name` + `ro.product.device` | Code name (e.g.: xuanyuan) |
| `marketname` | ❌ | `ro.product.marketname` | Model name (e.g.: REDMI K90 Pro Max) |
| `characteristics` | ❌ | `ro.build.characteristics` | Characteristics (e.g.: tablet) - only effective in resetprop mode |

**Notes**:
- All fields except `package` are optional
- When using template's `packages`, no need to write [[apps]] (automatically applied)
- Fields in [[apps]] will override template configuration
- `name` and `marketname` are only effective in **full mode** (affect SystemProperties)
- `name` field in full mode will simultaneously spoof `ro.product.name` and `ro.product.device`
- `characteristics` field is only effective in **resetprop mode**
- In **lite mode**, only `manufacturer`, `brand`, `model`, `device`, `product`, `fingerprint` take effect

## Mode Comparison

| Feature | lite Mode ⭐ | full Mode | resetprop Mode |
|---------|-------------|-----------|----------------|
| Build Class Spoofing | ✅ | ✅ | ✅ |
| SystemProperties Spoofing | ❌ | ✅ | ✅ |
| Read-only Property Modification | ❌ | ❌ | ✅ |
| Module Unloadable | ✅ | ❌ | ❌ |
| Stealth | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Detection Risk | Very Low | Relatively Low | Relatively Low |
| Recommendation | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

## How to Choose a Mode?

### Recommended Strategy

1. **Default to lite mode** - Covers 90% of scenarios
2. **Test the application** - See if it works normally
3. **If not** - Switch to full mode

### Judgment Criteria

**Use lite mode**:
- ✅ Most applications
- ✅ Pursuing stealth
- ✅ Don't want to be detected

**Use full mode**:
- Application reads SystemProperties
- Can still detect real device model in lite mode
- Don't mind being detected by the module

**Use resetprop mode**:
- Need to modify `ro.build.characteristics` (such as QQ tablet mode)