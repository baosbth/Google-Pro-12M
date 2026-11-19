# Google-Pro-12M
# 🚀 Google AI Trial Tool

[English](README.md) | [Tiếng Việt](README_VI.md)

> ⚠️ **WARNING:** This method involves runtime tool injection and carries a **high risk of account suspension/ban**. Use at your own risk and avoid using your primary Google account.

Injects into the Google One process via **Frida** to invoke the `BillingClient` API, discovering and activating hidden **$0 free trial** offers.

---

## 📋 Prerequisites

| Component | Description |
| :--- | :--- |
| **MuMu Emulator** | Running, logged into a Google account, with a payment method linked |
| **adb** | Available in `PATH`, or specify via the `ADB_PATH` environment variable |
| **frida (host)** | `pip install frida-tools` (version `>= 16`; **Frida 16.7.19** is recommended) |
| **frida-server (device)** | Pushed to `/data/local/tmp/frida-server` with execute permissions |
| **module.apk (device)** | APK containing the `BillingClient` library, pushed to `/data/local/tmp/module.apk` |
| **Google One** | Installed on the device (`com.google.android.apps.subscriptions.red`) |

> **Note:** Frida `>= 17` introduces breaking changes (the Java bridge requires explicit loading). Using **Frida 16.7.19** is strongly recommended.

---

## 🛠️ Device Setup

```bash
# Connect to MuMu Emulator
adb connect 127.0.0.1:16384

# Push frida-server (download matching architecture from [https://github.com/frida/frida/releases](https://github.com/frida/frida/releases))
adb push frida-server /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"

# Push module.apk (APK containing com.android.billingclient)
adb push module.apk /data/local/tmp/module.apk
```

---

## 💻 Usage

```bash
# List all available trials for the current account
./auto-trial.sh --list

# Activate Pro (5TB, $19.99/mo) 3-month free trial
./auto-trial.sh --product pro --dur 3mo

# Activate Pro variant (5TB, $19.99/mo) 12-month free trial ★ Recommended
./auto-trial.sh --product pro12 --dur 12mo

# Activate Ultra 20x (30TB, $199.99/mo) 12-month free trial
./auto-trial.sh --product 20x --dur 12mo

# Activate Ultra 5x (20TB, $99.99/mo) 6-month free trial
./auto-trial.sh --product 5x --dur 6mo

# Automatically select the longest trial (defaults to highest tier)
./auto-trial.sh

# Specify a custom MuMu ADB port
./auto-trial.sh --port 16416 --product pro --dur 3mo
```

---

## ⚙️ Arguments & Environment Variables

### CLI Arguments
| Argument | Description | Default |
| :--- | :--- | :--- |
| `--list` | List available trials without activating | *(Activation mode)* |
| `--product` | `pro` / `pro12` / `5x` / `20x` / `auto` / full `g1.*` ID | `auto` |
| `--dur` | `1mo` / `3mo` / `6mo` / `12mo` / `longest` | `longest` |
| `--port` | MuMu ADB port | Auto-scan |
| `--adb` | Path to `adb` executable | Auto-detect |

### Environment Variables
| Variable | Description |
| :--- | :--- |
| `ADB_PATH` | Path to `adb` executable |
| `FRIDA_SCRIPT` | Path to `frida-auto-trial.js` (defaults to current directory) |
| `FRIDA_SERVER_PATH` | Path to `frida-server` on device (default: `/data/local/tmp/frida-server`) |
| `MODULE_APK_DEVICE` | Path to `module.apk` on device (default: `/data/local/tmp/module.apk`) |

---

## 📦 Product ID Reference

| Alias | Product ID | Storage | Monthly Price | Trial Duration |
| :--- | :--- | :--- | :--- | :--- |
| `pro12` ★ | `g1.2tb.ai.subs_variant` | 5TB | $19.99 | 6 / 12 months |
| `pro` | `g1.play.2tb.ai` | 5TB | $19.99 | 3 months |
| `5x` | `g1.play.20tb` | 20TB | $99.99 | 1 / 3 / 6 / 12 months |
| `20x` | `g1.play.ai.pro` | 30TB | $199.99 | 1 / 3 / 6 / 12 months |
| — | `g1.2tb.ai` | 5TB | $19.99 | None |
| — | `g1.ai.pro` | 30TB | $199.99 | None |
| — | `g1.30tb` | 30TB | $149.99 | None |

> **Status Update (2026-11-19):** The default "Upgrade" button in the app uses non-trial IDs (`g1.2tb.ai`, etc.). Ultra `5x`/`20x` trial offers have been removed. Currently, Pro has two active trial IDs:
> - `g1.2tb.ai.subs_variant` (`pro12`): Up to **12 months free** (recommended).
> - `g1.play.2tb.ai` (`pro`): **3 months free**.

---

## ❌ Why Web-Only Methods Don't Work

Testing confirms that web-based approaches cannot bypass Google Play's server-side validation:
- **Static URLs:** `rpctoken`, `_gfid`, and the payments blob are session-bound and expire immediately.
- **Console Script (`ear` modification):** The 25-byte `ear` parameter stores the product at `ear[13]` and an integrity signature at `ear[24]`. Changing the product invalidates the signature, causing the server to return `400`.
- **Chrome Extension RPC Hook:** Community workarounds intercepting the `Ngbm` RPC payload are no longer available (repos returned 404).

**Conclusion:** Hidden trials (e.g., `g1.play.2tb.ai`) are not exposed on any web page. The only reliable method is invoking the `BillingClient` API directly on-device via Frida.

---

## ⚠️ Important Notes

- The script only triggers the Google Play payment dialog; you must **manually confirm** (enter your password).
- Subscriptions **auto-renew** after the trial ends—remember to cancel in advance.
- The account must have a **linked payment method** and **no active subscription**, otherwise trials will be unavailable.
- `launchBillingFlow` returning `code=0` may consume your trial eligibility even if the payment popup fails to display.
- Perform all steps inside an emulator and use a low-balance/virtual card as a safety net.

---

## 📁 Project Files

- `auto-trial.sh` — Shell launcher that auto-detects devices, manages `frida-server`, injects the script, and monitors results.
- `frida-auto-trial.js` — Frida injection script that loads `BillingClient` via `DexClassLoader` to query and activate trials.
