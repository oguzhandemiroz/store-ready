# Checklist: phases 2 to 9

Work through every phase that applies to this release. Record evidence for each check. Status values are defined in SKILL.md.

## Phase 2: The binary and build settings (iOS)

Prefer the real artifact: an `.ipa`, `.xcarchive` or built `.app`. Ask for
its path if you cannot find it. Unzip to a temp directory if needed. Config
files describe intent; the binary is the truth.

- **Upload minimums:** built with the SDK/Xcode currently required
  (`DTSDKName`, `DTXcode`, `DTPlatformVersion` in the built `Info.plist`),
  deployment target at or above the current minimum (SKILL.md Phase 0, item 3).
- **Version:** `CFBundleShortVersionString` and `CFBundleVersion` are higher
  than the live version and the last uploaded build.
- **Purpose strings:** scan the binary and its frameworks for protected API
  families (`nm -u`, `otool -L`, `strings`): Photos, Camera, Microphone,
  Location, Contacts, Calendars/Reminders, Motion (CoreMotion), Bluetooth,
  Local Network, Face ID, Tracking (ATT), Speech, HealthKit, HomeKit, NFC,
  Media Library. Every family that is linked needs a specific purpose string
  in every shipped language, even when your code never calls it (SDKs pull
  frameworks in; a disabled config flag does not remove them). Every purpose
  string must describe a real use. A string for an API the app never uses is
  a finding too.
- **Privacy manifests:** `PrivacyInfo.xcprivacy` exists for the app; every
  required-reason API the app or its SDKs use is declared with a valid reason
  code. Take the categories and codes from Apple's current page, not memory.
  SDKs on Apple's list (SKILL.md Phase 0, item 4) ship their own signed manifest at a
  version that has one.
- **Export compliance:** `ITSAppUsesNonExemptEncryption` is set, and its
  value matches reality.
- **App icon:** 1024x1024 marketing icon present, **no alpha channel**
  (check the PNG colour type; an RGBA file with all-opaque pixels still
  fails), no transparency, no rounded-corner pre-masking needed.
- **iPad and orientation:** if the app runs on iPad, either all four
  orientations are supported or `UIRequiresFullScreen` is set; the UI is
  usable on iPad sizes (reviewers test on iPad).
- **Entitlements and capabilities** (`codesign -d --entitlements :- <app>`):
  each entitlement matches a real feature (push, iCloud, Game Center, Sign in
  with Apple, Associated Domains, App Groups) and nothing is left over.
- **Private API and URL schemes:** no `App-Prefs:`/`prefs:` URLs, no private
  framework use, no leftover debug frameworks.
- **Build freshness:** the binary was built after the last commit that
  belongs in this release. Compare the artifact timestamp or embedded
  version/commit with `git log`. A stale build means the reviewer sees an app
  that does not match the screenshots or notes.
- **Release hygiene:** no debug menus, test or staging endpoints, test ad
  unit IDs in production (or production IDs in a test build you are about to
  ship), sandbox/test payment keys, verbose logging of user data, lorem ipsum,
  placeholder images, "TODO"/"TBD"/"Coming soon" in user-facing strings.
- **Upload warnings:** if a build was already uploaded, ask for (or read
  through read-only tooling) the processing warnings and any ITMS emails.
  Common: ITMS-90683 (missing purpose string), ITMS-90474 (iPad
  orientations), ITMS-90717 (icon alpha), ITMS-91061 (missing SDK privacy
  manifest). The cheapest catch is an early TestFlight upload of a candidate
  build; recommend it if none exists.

## Phase 3: The Android build (if in this release)

- **Target API level** meets the current requirement (SKILL.md Phase 0, item 9);
  `versionCode` is higher than the last upload.
- **Merged manifest permissions:** read the permissions of the built AAB/APK
  (`aapt2 dump permissions`, `bundletool dump manifest`), not only the source
  manifest. Libraries add permissions. Every permission matches a real
  feature and the Data safety form.
- **Advertising ID:** `AD_ID` permission present only if ads/attribution use
  it; Families rules if children may be in the audience.
- **Platform leaks:** no iOS-only UI on Android (Apple sign-in page, Game
  Center, "App Store" wording, Apple-branded screens), no store products
  shown that cannot be bought on Play.
- **New personal developer accounts:** production access requires a closed
  test with the current minimum testers for the current minimum days, per
  app. The internal track does not count. Ask if this is a first release.
- **Production access / declarations:** answers must describe what was
  actually done. Do not let a template claim testing or changes that did not
  happen.
- **Pre-launch report:** expect Google's automated test devices to open the
  app after every upload. Tell the user to exclude them from analytics and,
  for non-Google ad networks, to exclude the test IP ranges.

## Phase 4: Completeness and functionality

- Every feature named in the description, screenshots, promo text, preview
  video, What's New and review notes exists in this build and works.
- Nothing unfinished is visible: no "coming soon", locked placeholder
  options, empty states that look broken, dead buttons, broken links.
- Backend and remote content are live for the review period.
- **Sign-in:** if any feature needs an account, a working demo account (or a
  demo mode) is in the review details. If subscriptions exist, an account
  whose subscription has expired helps the reviewer test the paywall.
- **Hidden or switchable features:** no feature that is hidden from the
  reviewer, dormant, or turned on by remote config after approval. List every
  remote flag you find and what it controls.
- **Minimum functionality and spam risk:** if the app is in a crowded
  category (simple puzzle, flashlight, timer, tracker, wallpaper, detector,
  dating, and so on) or is one of several similar apps from the same
  developer, say what makes it distinct and check that the review notes and
  first screenshots say it too. Template or white-label apps are high risk.
- **System prompts:** permission and ATT prompts appear at a moment that
  makes sense, not stacked at launch and not over gameplay. A custom screen
  before a system prompt uses a neutral button ("Continue"), not "Allow", and
  the app does not block use after a denial. Rewards and features are not
  gated on tracking consent.
- **Review prompts:** only the system API, no custom "rate us 5 stars" flows
  that pre-filter happy users.
- **Offline / no-network claims** in metadata are true.
- **Crashes:** you usually cannot run the app. Ask whether a clean install on
  a real device (and an iPad, if supported) was tested on this exact build.
  If a simulator and build are available and the user agrees, you may launch
  it to look, but never sign in with real credentials.

## Phase 5: Monetization

- **Digital goods use IAP.** No external payment links or buttons for digital
  content unless a current regional rule allows it (re-read the live rule).
- **Every new IAP, subscription and subscription group is attached to this
  submission** with its review screenshot and review note. A version
  submitted without them means the reviewer cannot buy, and the paywall shows
  an error. On App Store Connect the first IAP of an app must ship with a
  version.
- **Restore purchases** exists where purchases can be restored.
- **Paywall disclosure** (subscriptions): name, duration, full renewal price
  per period as the most prominent price, trial length and the price after
  it, auto-renew statement, how to cancel, links to Terms of Use (EULA) and
  Privacy Policy **inside the app and in the store metadata** (description
  or the EULA field). Prices on screen come from the store, not hard-coded.
- **Paid random rewards** (loot boxes, gacha, mystery chests bought with real
  money or with currency bought with real money): odds of each item type are
  shown before purchase.
- **Purchased currency does not expire.**
- **Ads:** tracking only after ATT consent; consent (e.g. a CMP) where the
  ad network requires it in the EEA/UK; ads do not imitate UI or system
  alerts; interstitials are not shown at launch or so often the app is
  unusable; rewarded ads are opt-in; check the current rule on letting users
  report inappropriate ads. Kids category or child audience: much stricter
  rules on third-party ads and analytics; flag and cite.
- **Test vs production:** sandbox and test-store keys are not in the release
  build; demo accounts are not left showing an active subscription from old
  sandbox purchases (clear sandbox history if needed).

## Phase 6: Privacy and data

- **Privacy policy:** find it (repo, website source, metadata). If not
  found, **ask for the URL.** Fetch it and check: reachable, matches the app
  (data collected, purpose, third-party SDKs that receive data, retention,
  deletion, contact), not contradicted by any in-app screen. Linked inside
  the app and in App Store Connect.
- **App Privacy label (Apple) and Data safety (Google):** compare the
  answers (from metadata files, or ask the user to export or paste them)
  with what the binary and SDKs actually collect: identifiers, usage data,
  diagnostics, location (including IP-derived), purchases, contact info.
  Ads or attribution SDKs usually mean "used for tracking" or "shared".
- **Consent for analytics and usage data:** check the current guideline on
  consent for usage data, and whether the app offers it or relies on a
  documented legal basis. Flag; the decision is the owner's.
- **Account deletion:** if users can create an account (including automatic
  guest accounts), deletion is available in the app, not only by email.
  With Sign in with Apple, tokens are revoked on deletion.
- **Login services:** if a third-party or social login is the main login,
  check the current equivalent-login rule.
- **AI and third-party data sharing:** if personal data goes to an AI or
  other third-party service, the app says so and asks before sending, per
  the current rule.
- **Age rating:** the questionnaire is answered for the current set of
  questions (Apple adds questions; check the Phase 0 news for deadlines) and the
  answers match the content (ads, web views, UGC, chat, gambling-like
  mechanics, location sharing).
- **EU DSA trader status** is declared in App Store Connect. Without it the
  app can be missing from EU storefronts. After release, verify with a store
  lookup in an EU country rather than trusting the console badge.
- **Regulated claims:** health, medical, finance, safety or security claims
  are accurate, hedged where needed, and not stronger in the store than in
  the app.

## Phase 7: Store metadata and assets

Read every locale, not only the primary one.

- **Name, subtitle, keywords:** within the current character limits (count
  characters, not bytes); no prices, discounts or "free"-style price claims
  where the current rule forbids them; no other platform names (e.g.
  "Android"); no competitor or trademarked names you do not own; no keyword
  stuffing in the name or subtitle.
- **Description, promo text, What's New:** accurate for this build; no
  features that are not shipped; What's New lists only real changes.
- **Screenshots and preview video:** real UI from this build (or clearly
  representative of it); no prices or discounts, no unverifiable claims
  ("#1", "millions of players", "loved by"), no other platforms' devices; the
  right device sizes; no stale screenshots from an older version. When
  screenshot files are local, compare by content hash, not by count: a new
  version can inherit the old set, and some upload modes append instead of
  replace, so a set can contain duplicates or old images and still have the
  "right" number.
- **Fields that do not carry over:** when a new version is created, some
  localized fields (often promo text and What's New) start empty. Check that
  every field is filled in every locale and matches what you intend.
- **URLs:** support URL loads and shows a way to contact the developer;
  marketing URL page is finished (no "work in progress") and claims only
  shipped features; privacy URL loads. Fetch each one.
- **In-App Events and Game Center:** event deep links open the right screen;
  leaderboards and achievements that the listing promotes are attached to
  this submission.
- **Project vocabulary rules** from Phase 1 (SKILL.md) hold in every locale and every
  visible field (name, subtitle, description, promo text, screenshot text,
  event copy).

## Phase 8: Submission state and review details

Use the project's read-only store tooling if it exists (see SKILL.md, Hard limit 2).
Otherwise ask the user to confirm each item or paste/screenshot the page.

- The version is in a state that allows submission, and **no other version is
  waiting for or in review** (a new version cannot be created while one is
  waiting, and app-level fields may be locked).
- The selected build is the intended build number and has finished
  processing without warnings.
- The submission contains **every** item that must go with it: the version
  plus all new IAPs, subscriptions, Game Center versions and In-App Events.
  Count the items; a submission holding only the version is the most common
  "looks fine, isn't" state.
- **Review notes** explain: what the app does and for whom, how to reach
  every non-obvious feature and every purchase, the demo account, what
  changed in this version, external services used (ads, analytics,
  payments, AI), and anything that might look odd (a privacy cover screen,
  a feature that needs special hardware). The notes match this build exactly.
- **Contact info** (name, phone, email) is filled and current.
- **Release type** (manual, automatic, phased) is what the user intends.
- Tooling that writes metadata (e.g. `deliver`) can overwrite review notes or
  the release type on its next run. If the project uses such tooling, check
  its config for review information and release settings and warn if it will
  clobber what is in the console.
- A long time in "Waiting for Review" is normal. Do not recommend withdrawing
  a submission just because it is slow.

## Phase 9: Play Console state (if Android is in this release)

Store listing in every language, Data safety, content rating questionnaire,
target audience and content, ads declaration, app access (test credentials
for gated features), government/financial/health declarations if relevant,
countries, and the release notes. Same rules as Phase 7 for accuracy.
