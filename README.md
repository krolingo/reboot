
# 🔄 Safe Reboot Scripts for macOS with FileVault

This repository provides user-level wrapper scripts for `reboot` and `shutdown` on macOS systems with **FileVault enabled**, ensuring smooth and unattended reboots using Apple’s `fdesetup authrestart`.

## 💡 Why Use These Scripts?

On FileVault-enabled systems, a normal `reboot` or `shutdown -r now` can cause the system to pause at the FileVault disk unlock screen. This is a problem for:

- Remote access and headless reboots
- Automation and scripted maintenance
- User convenience

Apple provides the command:

```sh
fdesetup authrestart
```

...which securely reboots the system and stores a one-time unlock token in the firmware. But it requires `root` privileges — which we manage using `doas`.

## Benefits of This Approach

- Avoids alias limitations — these scripts work with `doas`, `sudo`, and scripts
- Consistent experience across shells
- Mimics standard `reboot` and `shutdown` behavior
- Minimal and user-controlled, no need to modify system binaries

---

## 🛠 Installation

### 1. Ensure `~/bin` exists and is in your `$PATH`

Add this to your `~/.zshrc`, `~/.bash_profile`, or similar:

```sh
export PATH="$HOME/bin:$PATH"
```

Then reload your shell:

```sh
source ~/.zshrc  # or source ~/.bash_profile
```

### 2. Install the Scripts

#### `~/bin/reboot`

```sh
#!/bin/sh
doas fdesetup authrestart
```

This script launches the FileVault-aware reboot using `doas`. It runs as a child process of the shell.

#### `~/bin/shutdown`

```sh
#!/bin/sh
# redirect even doas shutdown
exec doas fdesetup authrestart
```

This script does the same thing but uses `exec` to fully replace the shell process — cleaner for system-level behavior like `shutdown`.

Make both scripts executable:

```sh
chmod +x ~/bin/reboot ~/bin/shutdown
```

### 3. Configure `doas` (if not already)

Edit `/etc/doas.conf` (as root):

```conf
permit yourusername as root cmd fdesetup
```

You can test it with:

```sh
doas fdesetup authrestart
```

---

## 🧪 Verification

Check that your shell resolves the commands correctly:

```sh
which reboot     # → /Users/yourusername/bin/reboot
which shutdown   # → /Users/yourusername/bin/shutdown
```

Try:

```sh
reboot
# or
shutdown -r now
```

Both will run the safe reboot command using `fdesetup authrestart`.

---

## 🧼 Summary

| Command           | Behavior                                   |
|-------------------|--------------------------------------------|
| `reboot`          | Runs `fdesetup authrestart` via `doas`     |
| `shutdown -r now` | Same, using `exec` in wrapper script       |
| `doas shutdown`   | Intercepts if `~/bin` is in your `$PATH`   |

---

## 🙌 Why Not Aliases?

- Aliases don’t work with `doas`, `sudo`, or from scripts
- Scripts are portable, reliable, and can be version-controlled

---

## 👩‍💻 Credits

Created by N0b0d7, inspired by practical needs for remote rebooting of FileVault-enabled Macs.
