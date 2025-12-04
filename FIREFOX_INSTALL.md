# Firefox Installation Guide

## Why is Firefox Installation Different?

Firefox requires extensions to be **signed by Mozilla** for permanent installation. Since this is a custom fork, it's not signed, which means:

- ❌ Double-clicking the `.xpi` file shows "corrupt" error
- ❌ Regular installation doesn't work
- ✅ Temporary installation works (but removed on browser restart)
- ✅ Developer mode allows permanent installation

## Installation Methods

### Method 1: Temporary Installation (Easiest)

**Pros**: Works immediately, no config changes  
**Cons**: Removed when Firefox closes

1. Download `dist-gecko.zip` from [Releases](https://github.com/ak-47-brar/jumpcutter/releases)
2. Extract the zip file
3. Open Firefox and navigate to: `about:debugging#/runtime/this-firefox`
4. Click **"Load Temporary Add-on..."**
5. Select the `manifest.json` file from the extracted folder
6. ✅ Extension is now active!

**Note**: You'll need to repeat this every time you restart Firefox.

---

### Method 2: Permanent Installation (Firefox Developer Edition)

**Pros**: Stays installed permanently  
**Cons**: Requires Firefox Developer Edition or Nightly

1. Install [Firefox Developer Edition](https://www.mozilla.org/en-US/firefox/developer/) or [Firefox Nightly](https://www.mozilla.org/en-US/firefox/channel/desktop/#nightly)
2. Open Firefox and go to: `about:config`
3. Search for: `xpinstall.signatures.required`
4. Double-click to set it to **`false`**
5. Download `dist-gecko.zip` from [Releases](https://github.com/ak-47-brar/jumpcutter/releases)
6. Extract the zip file
7. Go to: `about:debugging#/runtime/this-firefox`
8. Click **"Load Temporary Add-on..."** (yes, still says temporary)
9. Select the `manifest.json` file
10. ✅ Extension stays installed permanently!

---

### Method 3: Install from XPI (Advanced)

**Coming Soon**: We can create a self-signed `.xpi` file for easier installation.

For now, use Method 1 or 2 above.

---

## Troubleshooting

### "This add-on could not be installed because it appears to be corrupt"

This error appears because Firefox requires signed extensions. Use **Method 1 (Temporary)** or **Method 2 (Developer Edition)** instead.

### Extension disappears after closing Firefox

You used Method 1 (temporary installation). This is normal. Either:
- Reload it each time using Method 1, or
- Use Method 2 for permanent installation

### "Load Temporary Add-on" is grayed out

Make sure you're on the correct page: `about:debugging#/runtime/this-firefox`

---

## Comparison with Chrome

| Feature | Chrome/Edge | Firefox (Regular) | Firefox (Dev Edition) |
|---------|-------------|-------------------|----------------------|
| Permanent Install | ✅ Yes | ❌ No (requires signing) | ✅ Yes (with config) |
| Easy Installation | ✅ Load unpacked | ⚠️ Temporary only | ✅ With config change |
| Survives Browser Restart | ✅ Yes | ❌ No | ✅ Yes |

---

## Recommended Approach

**For testing/occasional use**: Use Method 1 (Temporary)  
**For daily use**: Use Method 2 (Firefox Developer Edition) or use the Chrome version in Edge/Brave

---

## Questions?

If you encounter issues, please [open an issue](https://github.com/ak-47-brar/jumpcutter/issues) with:
- Your Firefox version
- Installation method you tried
- Complete error message (if any)
