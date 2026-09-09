# Quick Start — Get the App Running in 5 Minutes

## Step 1: Create a GitHub repository (2 min)

1. Go to https://github.com/new
2. **Repository name**: `phishing-guard` (or any name you like)
3. **Visibility**: Public (free tier) or Private (doesn't matter)
4. Click **Create repository**
5. **Copy the HTTPS link** shown (looks like `https://github.com/your-username/phishing-guard.git`)

## Step 2: Upload the code to GitHub (2 min)

### Option A: Using Git (if you have it installed)

```bash
# Navigate to the unzipped phishing_guard folder
cd phishing_guard

git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/phishing-guard.git
git push -u origin main
```

### Option B: Without Git — drag and drop on GitHub (easier)

1. On your repo page (the blank one you just created), scroll down
2. Find **"uploading an existing file"** section
3. Drag and drop the entire unzipped `phishing_guard` folder onto it
4. Click **Commit changes**

## Step 3: Wait for GitHub to build the APK (5–8 min)

1. On your repo page, click the **Actions** tab
2. You'll see a workflow run called **"Build APK"** running
3. Wait for the **green checkmark** (means it succeeded)

## Step 4: Download the APK to your phone (1 min)

1. Click into that **"Build APK"** run
2. Scroll to the bottom → **Artifacts** section
3. Click **phishing-guard-apk** to download a zip file
4. Unzip it on your computer → you'll find `app-release.apk`
5. **Send this APK to your phone** via:
   - Email (attach it to yourself)
   - Google Drive / Dropbox
   - USB cable (copy to phone storage)
   - Bluetooth

## Step 5: Install on your phone (1 min)

1. On your Android phone, open the file manager / Files app
2. Navigate to where the APK is
3. Tap **app-release.apk**
4. A popup says **"Install unknown app?"** → tap **Install anyway** (or **More** → **Install anyway**)
5. Wait for it to install (takes 10–20 seconds)
6. When done, tap **Open** (or find **Phishing Guard** in your apps)

## Step 6: First launch — Set as default SMS app (30 seconds)

1. Open the app
2. You'll see a banner: **"Blocking is OFF"**
3. Tap **"Set as default SMS app"**
4. Android shows a dialog → confirm
5. ✅ **You're live!** Phishing texts will now be blocked before they reach your inbox

---

## That's it! 

- Messages come in → the app scores them instantly (on-device, offline)
- Phishing (score ≥55) → blocked, quarantined in the app only
- Suspicious (25–54) → delivered, but flagged in the app
- Safe (<25) → delivered normally
- Open **Phishing Guard** anytime to see the **History** of all scanned messages

## Optional: Add your Gemini API key for richer AI explanations

1. In the app, tap the **⚙️ Settings** icon (top right)
2. Paste a free Gemini API key (get one at https://aistudio.google.com/apikey)
3. Tap **Save key**
4. Now when you review blocked/suspicious messages, you'll see a deeper AI analysis

## Got issues?

- **"Install failed - Parse error"**: The APK is corrupted. Try downloading again.
- **App doesn't block anything**: You didn't tap "Set as default SMS app" — do that now.
- **"Permission denied"**: Android needs SMS permissions — go to **Settings > Apps > Phishing Guard > Permissions** and enable SMS.

---

**Questions?** Check the full README.md in the project for more details on tuning, testing, etc.
