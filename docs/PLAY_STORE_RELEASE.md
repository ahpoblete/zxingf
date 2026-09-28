# Play Store release checklist

Reusable steps for publishing this app (and future ones) to Google Play.
Parts specific to this app are marked **[this app]**; the rest applies to any
app from this developer account.

## 1. Build a signed release bundle

1. Generate an upload keystore once (keep it forever, back it up):
   ```
   keytool -genkey -v -keystore upload-keystore.jks -keyalg RSA -keysize 2048 \
       -validity 10000 -alias upload
   ```
2. Copy `android/keystore.properties.example` to `android/keystore.properties`
   and fill in the real `storeFile` (absolute path), `storePassword`,
   `keyAlias`, `keyPassword`. This file is gitignored — never commit it.
3. Build the bundle Play requires (not an APK):
   ```
   ./gradlew :android:bundleRelease
   ```
   Output: `android/build/outputs/bundle/release/android-release.aab`

## 2. Create the app in Play Console

Play Console → **Create app** →
- App name, default language, "App" (not game), Free/Paid.
- Declarations: confirm it meets Developer Program Policies and US export laws.

## 3. Store listing (Grow → Store presence → Main store listing)

- Short description (≤80 chars), full description (≤4000 chars).
- App icon (512×512 PNG), feature graphic (1024×500), at least 2 phone
  screenshots (16:9 or 9:16, min 320px on the short side).
- Category and tags. **[this app]** Category: Tools.
- Contact details: email (required), website/phone (optional).

## 4. App content (Policy → App content) — this is the section with the two forms you asked about

Go through every item on this page; Play won't let you release until they're
all "Complete":

### Privacy policy
- Paste the **public URL** where you've published `PRIVACY_POLICY.md` from
  this repo (GitHub Pages, your own site, a Gist raw link, etc. — any URL
  that's publicly reachable without login works).

### Data safety form
This is the form buyers see as "Data safety" on the store listing. Answer
based on what the app actually does (see `PRIVACY_POLICY.md`):

- **Does your app collect or share any of the required user data types?**
  → **Yes** (**[this app]**: it transmits scanned barcode text to Google's
  Book/Product search APIs for the optional "more info" lookup, and reads
  one contact at a time via the system picker to encode it as a QR code).
- Data types to declare **[this app]**:
  - **Personal info → Name** (from a picked contact, only when you use
    "Share → Contact"): Collected: No off-device transmission by this app
    beyond what's baked into the generated barcode image you choose to
    share yourself. Typically answer "Not collected" here since the app
    itself doesn't transmit it — only *encodes* it locally. If unsure,
    answer "Collected, not shared, user-initiated, not required."
  - **App activity → Other user-generated content**: the scanned barcode
    value sent to Google for the optional lookup.
    - Purpose: **App functionality**.
    - Is it shared with third parties? **Yes** (Google's public search
      APIs) — purpose: app functionality, not for advertising.
    - Is it processed ephemerally? **Yes** — it's a stateless lookup, not
      stored by this app or (to your knowledge) retained by Google beyond
      normal request logs.
    - Can users opt out? **Yes** — Settings → "Retrieve more info" toggle.
    - Is data encrypted in transit? **Yes** — all three app-initiated
      lookups (Google Books API, Google Product Search, and "search inside
      this book") use HTTPS.
  - Everything else (location, financial info, health, messages, photos,
    audio, files/docs, device identifiers, analytics) → **Not collected**.
- **Is all user data encrypted in transit?** Answer honestly per above.
- **Do you provide a way for users to request data deletion?** You can
  answer "not applicable" since nothing is stored server-side; local data
  is deleted by clearing history/uninstalling.

### Ads
- **[this app]** No ads → select "No, my app does not contain ads."

### Content rating
- Start the questionnaire (IARC). **[this app]** category: **Utility,
  Productivity, Communication, or Other**.
- Answer "No" to violence, sexual content, gambling, controlled substances,
  user-generated content shared publicly, and location sharing.
- Result should come back **Everyone / PEGI 3**.

### Target audience and content
- Target age groups: pick the realistic range (e.g. 18+, or 13+ if you're
  comfortable — avoid selecting an audience that includes children unless
  the app is designed for them; that triggers extra Families Policy
  requirements you don't want here).
- "Appeal primarily to children?" → No.

### Government apps / financial features / health / news
- **[this app]** None apply — answer No / not applicable.

### Data safety - other required declarations
- US export compliance, content guidelines acknowledgement — confirm.

## 5. Set up a release

Release → **Testing → Internal testing** (recommended first) or
**Production**:
1. Create new release.
2. Upload `android-release.aab` from step 1.
3. Release name / notes.
4. Opt in to **Play App Signing** when prompted on your first upload
   (Google re-signs your app with its own key for distribution; you keep
   your upload key to sign future releases — this is the recommended,
   effectively required path for new apps).
5. Review the release, then **Save → Review release → Start rollout**.

Internal testing releases are visible only to testers you add by email;
Production goes live to everyone (subject to Google's review, which can
take a few hours to a few days for a first submission).

## 6. Subsequent releases

Bump `versionCode` and `versionName` in `android/build.gradle`, rebuild
(`bundleRelease`), upload a new release to your chosen track. No service
account or Play Developer API needed for manual uploads — that's only
required if you want CI to upload automatically, which this repo
intentionally does not do.
