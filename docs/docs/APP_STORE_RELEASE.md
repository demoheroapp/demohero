# DemoHero Mac App Store release

This is the owner checklist for the first Mac App Store version and the first in-app purchases. The app binary already implements StoreKit 2, free limits, and the three Pro plans. App Store Connect setup and the signed MAS archive still require the paying Apple Developer account.

**Version in this repo:** `0.1.0` (build `1`)  
**Bundle ID:** `com.demohero.DemoHero`  
**Category:** Video (`public.app-category.video`)

Do not create different product IDs. The app hard-codes these:

| Plan | Type | Product ID | US price |
| --- | --- | --- | --- |
| DemoHero Pro Monthly | Auto-renewable subscription | `com.demohero.DemoHero.pro.monthly` | $3.99 / month |
| DemoHero Pro Yearly | Auto-renewable subscription | `com.demohero.DemoHero.pro.yearly` | $19.99 / year |
| DemoHero Pro Lifetime | Non-consumable | `com.demohero.DemoHero.pro.lifetime` | $49.99 |

Monthly and yearly are the same subscription level in one group named **DemoHero Pro**. Lifetime is a separate non-consumable. Any one of them unlocks every shipping feature (1080p / 4K export and recordings longer than 10 minutes). Parked v1.x features are not part of Pro.

Local StoreKit testing uses [`src/Resources/DemoHero.storekit`](../src/Resources/DemoHero.storekit) on the DemoHero scheme. The in-app sheet reads `Product.displayPrice` from that file (currently $3.99 / $19.99 / $49.99). Prices are not cached in DemoHero; if Xcode still shows older amounts, quit the app, use **Debug → StoreKit → Manage Transactions…** to delete leftover purchases, then Run again so StoreKit reloads the configuration.

---

## 1. Account prerequisites (do these first)

Apple will reject the first paid / subscription submission if any of these are missing.

1. Enroll in the [Apple Developer Program](https://developer.apple.com/programs/).
2. In App Store Connect → **Agreements, Tax, and Banking**:
   - Accept the **Paid Apps Agreement**.
   - Add a bank account.
   - Complete tax forms for every required region, at least the United States.
3. Create a **Mac App Store distribution certificate** (Apple Distribution) and a **Mac App Store provisioning profile** for `com.demohero.DemoHero`.
4. In Xcode, set the DemoHero target **Team** to that developer team. The repo ships with ad-hoc local signing (`CODE_SIGN_IDENTITY = "-"`) so clones build without a team. Replace that for archive/export.
5. Host a **public Privacy Policy URL**. The in-app text lives in [`PRIVACY.md`](PRIVACY.md). GitHub’s blob URL is a starting point; App Review often prefers a stable HTTPS page you control.
6. Use Apple’s Standard EULA as the Terms of Use URL:  
   `https://www.apple.com/legal/internet-services/itunes/dev/stdeula/`
7. Support URL / email: `mailto:demohero@googlegroups.com` (also shown in the app).

---

## 2. App Store Connect app record

1. Create a macOS app with bundle ID `com.demohero.DemoHero`.
2. Set version `0.1.0`.
3. Privacy Policy URL and Support URL (required).
4. Age rating: the app records the user’s screen and camera. Answer the questionnaire honestly (no unrestricted web, no user-generated public content).
5. App Privacy: DemoHero does not collect account data or upload media. Declare camera, microphone, and screen recording as used on-device. Purchases are processed by Apple.
6. Pricing: the app is **Free** with in-app purchases.

---

## 3. In-app purchases (create before the first binary submit)

Create all three products and the subscription group **in the same first submission** as version 0.1.0. First-of-type subscriptions cannot ship after the app is already live.

### Subscription group

- Name: **DemoHero Pro**
- Localization (en-US): display name `DemoHero Pro`, description `Unlock all DemoHero features while subscribed.`
- Monthly and yearly: **same subscription level** (level 1). A subscriber can switch between them; they do not stack.

### Monthly

- Product ID: `com.demohero.DemoHero.pro.monthly`
- Reference name: Pro Monthly
- Duration: 1 month
- US price: **$3.99** (Apple price tier that maps to $3.99)
- Localization: display name `DemoHero Pro Monthly`; description `Unlock all DemoHero features, billed monthly.`
- Review screenshot: the in-app Pro screen showing the monthly plan and price.
- Review notes: “Auto-renewable monthly unlock of 1080p/4K export and recordings longer than 10 minutes.”

### Yearly

- Product ID: `com.demohero.DemoHero.pro.yearly`
- Reference name: Pro Yearly
- Duration: 1 year
- US price: **$19.99**
- Localization: display name `DemoHero Pro Yearly`; description `Unlock all DemoHero features, billed yearly.`
- Same review screenshot family as monthly.

### Lifetime

- Type: **Non-Consumable**
- Product ID: `com.demohero.DemoHero.pro.lifetime`
- Reference name: Pro Lifetime
- US price: **$49.99**
- Localization: display name `DemoHero Pro Lifetime`; description `Unlock every DemoHero feature forever, including 1080p and 4K export and unlimited recording.`

### Tax category

Use Apple’s software / app subscription tax category (the same category used for Mac productivity/video apps sold as digital goods). Do not classify this as a physical good or a streaming media service.

### Subscription disclosures

App Review requires:

- Privacy Policy URL
- Terms of Use (EULA) URL
- Clear in-app text that subscriptions auto-renew, the period, and that users cancel in Apple ID settings

The in-app Pro screen already includes this copy plus Restore Purchases and Manage Subscription.

---

## 4. Signing, sandbox, and archive

Current shipping entitlements in [`src/Resources/DemoHero.entitlements`](../src/Resources/DemoHero.entitlements):

- App Sandbox
- Camera
- Microphone (`device.microphone`) and Hardened Runtime audio input (`device.audio-input`)
- Movies folder read/write
- User-selected files read/write

Release builds enable the **Hardened Runtime**. Debug stays ad-hoc without it so local `CODE_SIGN_IDENTITY = "-"` still runs.

### Xcode archive

1. Select the DemoHero scheme → **Any Mac** (or My Mac).
2. Set the Team and a Mac App Store signing identity.
3. Product → Archive.
4. Organizer → Distribute App → **App Store Connect**.
5. Upload the build. Wait for processing.

### Command line (after the Team is set)

```bash
xcodebuild -scheme DemoHero -project DemoHero.xcodeproj \
  -configuration Release -destination 'generic/platform=macOS' \
  -archivePath /tmp/DemoHero.xcarchive archive

xcodebuild -exportArchive \
  -archivePath /tmp/DemoHero.xcarchive \
  -exportPath /tmp/DemoHero-mas \
  -exportOptionsPlist docs/ExportOptions-MAS.plist
```

`docs/ExportOptions-MAS.plist` is a template. Fill in your team ID before exporting.

### What this repo can prove without a team

A local Release archive can be created with ad-hoc signing. It **cannot** pass App Store Connect validation until a distribution certificate and team are selected. Sandbox purchases against live ASC product IDs also require that uploaded build.

---

## 5. Sandbox and TestFlight

1. App Store Connect → Users and Access → Sandbox → create a Sandbox Apple ID.
2. On the Mac: System Settings → App Store → Sandbox Account (or sign in when StoreKit prompts).
3. Run the MAS or TestFlight build, not only the local StoreKit-configured Debug build.
4. Prove each path:
   - Buy monthly → Pro unlocks without relaunch; 1080p/4K export works; recording past 10:00 continues.
   - Cancel / expire the monthly Sandbox subscription → free limits return.
   - Buy yearly (upgrade/switch in the same group) → still Pro.
   - Buy lifetime → still Pro after the subscription is gone.
   - Restore Purchases on a second Mac / after delete.
   - Manage Subscription opens Apple’s sheet or account page.
   - Airplane mode: last verified Pro user can still launch and export; free user still records and exports 720p if products fail to load.
5. Submit the same build to TestFlight for Mac and repeat the happy path.

Local Debug/Release with the `.storekit` file is for engineering. Reviewers will use Sandbox / TestFlight.

---

## 6. Review screenshots and notes

Minimum Mac screenshots (13-inch and one larger size if ASC asks):

1. Ready-to-record settings (permissions, source, camera).
2. Recording indicator showing `elapsed / 10:00` on the free tier.
3. Editor with the 720p export hint and locked 1080p/4K (Pro) presets.
4. Pro purchase sheet with StoreKit prices, yearly highlighted, Restore, Privacy Policy, and Terms of Use.

**Review notes (paste into ASC):**

> DemoHero is a Mac screen + camera recorder. Free use is 720p export and a 10-minute recording limit. DemoHero Pro unlocks 1080p/4K and unlimited recording via three IAPs submitted with this version: monthly (`com.demohero.DemoHero.pro.monthly`, $3.99), yearly (`com.demohero.DemoHero.pro.yearly`, $19.99), and lifetime (`com.demohero.DemoHero.pro.lifetime`, $49.99). Monthly and yearly share the DemoHero Pro subscription group at the same level.  
> To test: record a short take, export 720p, open Unlock Pro from settings or a locked export preset, purchase with a Sandbox account, then export 1080p. Restore Purchases and Manage Subscription are on the Pro sheet. Privacy Policy and Terms of Use are linked in-app. Support: demohero@googlegroups.com.  
> Grant Screen Recording, Camera, and Microphone when prompted. The app is a menu-bar companion; closing the settings window does not quit.

---

## 7. Submission order

1. Paid Apps Agreement, banking, and tax are Active.
2. App record exists with Privacy Policy URL, Support URL, and EULA URL.
3. Subscription group + monthly + yearly + lifetime are **Ready to Submit**.
4. Upload the signed `0.1.0` (1) Mac build.
5. Attach all three IAPs to this version.
6. Submit **the app version and the IAPs together**.
7. After approval, Sandbox/TestFlight remains the place to re-check renewals and refunds.

Do not submit the app first and the subscriptions later. Apple requires the first subscription group and first auto-renewable products to go out with an app version.

---

## 8. M9 demo (engineering + owner)

Engineering can prove this without ASC:

- Free recording clock shows `m:ss / 10:00`.
- Locked 1080p/4K presets explain the free limit and open the Pro sheet.
- StoreKit configuration file loads the three plans with $3.99 / $19.99 / $49.99 in Xcode.
- Unit tests cover entitlement rules, free limits, and SKTestSession buy/restore/expire/refund/load-failure.

The owner still must prove on a Sandbox or TestFlight build:

- Each live product purchases and restores.
- A Pro 1080p or 4K export succeeds.
- Expired or refunded access returns to free limits.
