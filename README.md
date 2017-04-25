# Xiaomi Bootloader Research

### Background
Xiaomi's newer devices have bootloaders which require unlocking through a process on Xiaomi's website. It's a terrible implementation, requiring you to use the stock MIUI browser, but nevermind.

There's obvious issues with this:
1. What happens if Xiaomi stop maintaining the service?
2. What if they stop responding to applications?

So I decided to have a look.

**NOTE: This is not set in stone, but from what I can deduce from reversing the unlock application. If any of this is wrong, send a pull request.**

### TL;DR
1. If the service dies, you better hope your BL is unlocked
2. It's actually implemented rather well

### MiFlashUnlock Changes
The recent (2.x) versions of MiFlashUnlock have switched to a new version of the API, which seems to report more information back to Xiaomi when an unlock occurs. This information isn't strictly identifiable, but does include a unique PC ID.

2.x also includes obfuscation, using VMProtect on the main executable. Both versions still give an unspecified error when internet access is lost after login to the application, showing Xiaomi's desire to keep details limited.

### The unlock process
Fastboot provides a number of properties for reasons such as this. One of these, with the key ```token```, is a 24 character string, ending in ```LAAAA```.

This token is encrypted using a 2048-bit RSA key stored at ```/unlock_key```, and temporary stored in the MiFlashUnlock directory as ```<<serial>>_sig.data```.

This data is then sent in a POST request to an API endpoint ```/api/(v1|v2)/ahaUnlock```, alongside the codename of the phone in later versions, and returns a signed version.

This 256 character ciphertext is then transmitted via fastboot via ```fastboot oem unlock "<<filename>>"```. The signature is verified and unlocked, or rejected and a reboot occurs.

This reboot, as well as other fastboot commands, can cause the token to change. This prevents the possibility of dumping the unlock data once for future use.

### The good

This is a pretty nice way to do things. No worry of hard bricking the device ala-Motorola. If the private key was released in the event of the service shutting down it'd also be rather portable.

### The Bad
Unless server-side code is released, unlocking will require the service to be maintained.
