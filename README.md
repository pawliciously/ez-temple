# temple.nz security audit

static analysis of temple, a paid minecraft fabric client from temple.nz

## what temple does to your computer

### screenshots your desktop

the loader binary imports windows gdi screenshot function: `BitBlt`, `GetDIBits`, `CreateDIBSection`, `CreateCompatibleDC`. these are screen capture api. it uploads the screenshot to `verify.temple.nz/api/tamper` together with your hardware fingerprint which i mean ok sure but this is gay

this gets triggered if you have wireshark, x64dbg, ghidra, or like any of 30+ tools open. does not matter if they are for something else entirely

they also call `SetWindowDisplayAffinity` to hide their own window from being captured. so they screenshot your desktop, but you cannot screenshot theirs. nice

### collects your identity

two endpoints fire when the anti-tamper goes off:

- `/api/report` sends your session token, what triggered it, and `os.name / user.name / COMPUTERNAME`
- `/api/flag` sends same thing plus a per-user watermark tied to your account

so yeah thats your windows username, computer name, and os just being shipped off lololol

### remote code execution because why the hell not i guess

on startup the client downloads i think 60 encrypted java modules from `verify.temple.nz/api/payload`. code gets decrypted in memory, loaded through a custom `ClassLoader`, runs, then gets wiped. never touches disk in readable form ofc because W OPSEC

temple can push whatever code they want to every running client whenever they feel like it. no update needed, no asking you, nothing on disk EVER

### anti-analysis

whole anti-tamper system is just there to stop you from seeing any of this:

**loader process blacklist:** cheatengine, x64dbg, x32dbg, windbg, ollydbg, ida64, ghidra, binaryninja, frida, wireshark, mitmproxy, fiddler, charles, procmon, and like 20+ more

**jar tool blacklist:** fernflower, enigma, recaf, jadx, cfr, procyon, jd-gui, bytecodeviewer, vineflower, quiltflower, krakatau, plus 20+ sandboxes and re tools

**jvm checks:** jdwp, -xdebug, -agentlib, -javaagent, -agentpath

if any of these get found your info gets reported, desktop gets screenshotted, and `Runtime.getRuntime().halt(0)` just kills the game. no warning no error just gone. there's so many better ways to do this but i don't expect more from this vibecoded client. "hey claude, make me a tuff drm"

### invisible to recording

6 mixin classes (`StreamproofEncoder`, `StreamproofGameRenderer`, `StreamproofGui`, `StreamproofRenderPass`, `StreamproofTerrain`, `StreamproofWidget`) use java foreign function api to make layered windows that do not show in obs, discord screenshare, windows game bar. with `SetWindowDisplayAffinity` on the loader too, temple is fully invisible to any screen capture

yeah cool you cannot get caught in screenshares but you also cannot record evidence of it running so theres that. tuff

### jar integrity

`IntegrityGuard` checks every class file signature against a stored hash. you cannot touch the jar without breaking this so theres no way to add any logging or debugging to see what it actually does

### kill switch

any security check fails and a daemon thread called `temple-exit` runs `Runtime.getRuntime().halt(0)`. hard kill. no shutdown hooks, no cleanup, no message. game just dies like this shitty client

## file hashes

```
Temple-Setup.exe    SHA256: fb6f33c3658cbb031e09fb2cbdccc54b4daf8918204107ea0230c6cfcc7feab3
temple-loader.exe   SHA256: 2a1b3fc0b5e2514212d6d0ac83dfa142172fd1dfc510c30b3f87876af7abb596
temple-client.jar   SHA256: DB85E2EB87240CE585D3B40C8E69B8E36A0340134F23C39A53CA58A0876432AB
```
