選擇你的語言:3 Select your language :3

[繁體中文](README.md) [English](README.en.md)

# Native Mac Island Experiment

This experiment uses LLDB to set in-memory flags in the running `MenuBarAgent` process and enable macOS’s built-in Dynamic Island interface. It does not install an app or permanently change preferences. The effect disappears when `MenuBarAgent` or the Mac restarts.

[Original GitHub project](https://github.com/habildem/native-mac-island-experiment)

## Known compatibility

The project’s README records a successful result on macOS 27.3 Beta, Build `26C5035r`. My M4 MacBook Air also displayed the music Dynamic Island on this build. I’m not sure whether other macOS versions are compatible.
![Dynamic Island demo](dynamic-island-demo.GIF)
## A ready-to-run version

By the way, I asked ChatGPT Codex to make a version that you can run directly. Download the ZIP, extract it, and run `Enable_native_Dynamic_Island.command`.

Remember to disable SIP first. The script may ask for an administrator password; follow the prompts.

## 1. Download and check the project

Download and extract the project. Make sure the folder contains:

```text
README.md
manifest.json
startup_lldb.py
```

Then run these commands in Terminal:

```sh
sw_vers -buildVersion
uname -m
csrutil status
xcrun --find lldb
shasum -a 256 /System/Library/CoreServices/MenuBarAgent.app/Contents/MacOS/MenuBarAgent
```

Confirm that the Build, SHA-256, and UUID match `manifest.json`, and that the architecture is `arm64`.

**Stop if anything does not match.** macOS updates may change the relevant addresses, so do not use old addresses on a different build.

## 2. Disable SIP

If `csrutil status` reports `enabled`:

1. Shut down your Mac and hold the power button until startup options appear.
2. Select **Options → Continue**.
3. From the menu bar, open **Utilities → Terminal**.
4. Run:

   ```sh
   csrutil disable
   ```

5. Then run:

   ```sh
   reboot
   ```

6. After returning to the normal desktop, run:

   ```sh
   csrutil status
   ```

Confirm that SIP is `disabled`.

## 3. Pause MenuBarAgent and attach LLDB

In a regular Terminal window, run:

```sh
sudo launchctl debug gui/$(id -u)/com.apple.MenuBarAgent --start-suspended
```

After you see `Service configured for next launch.`, run:

```sh
launchctl kickstart -k gui/$(id -u)/com.apple.MenuBarAgent
xcrun lldb -n MenuBarAgent
```

The menu bar may temporarily stop responding. If LLDB attaches successfully, it should show `MenuBarAgent` stopped near `dyld_start` because of `SIGSTOP`.

## 4. Load the experiment script

At the `(lldb)` prompt, enter the following command. Replace the example with the absolute path to the script on your Mac:

```text
command script import /Users/your-username/Downloads/native-mac-island-experiment-main/startup_lldb.py
```

You should see a message similar to:

```text
Startup HUD breakpoints installed. Continue; inspect startup_lldb.hits before claiming success.
```

Then you can continue.

If an error occurs, stop and follow the recovery instructions in the project README.

## 5. Start and check the Dynamic Island

At the LLDB prompt, enter:

```text
continue
```

Wait for the menu bar to respond again. Play music or use an existing Live Activity to check whether the Dynamic Island appears. **Only count it as successful if you can see it on screen.**

Then press **Ctrl-C** and enter:

```text
script print(startup_lldb.hits)
```

Confirm that both `gate` and `flags` are greater than `0`.

## 6. Exit LLDB while keeping the Dynamic Island

After confirming that it works, enter these commands at the `(lldb)` prompt:

```text
breakpoint delete --force
process detach
quit
```

This lets `MenuBarAgent` continue running.

To keep the Dynamic Island, do not run `launchctl kickstart -k ...`, log out, or restart your Mac. This is important: the effect remains only while `MenuBarAgent` keeps running.
