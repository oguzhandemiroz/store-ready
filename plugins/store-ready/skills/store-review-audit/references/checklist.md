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
- **Purpose string wording (5.1.1(ii)):** read every string in every
  language against Apple's current guidance at
  https://developer.apple.com/help/app-review/guideline-reference/5-1-1-purpose-strings.
  Each string names the feature that uses the resource and what the app
  does with it, with a concrete example, in one complete sentence. A string
  that only restates the resource ("To use the camera."), promises a vague
  benefit ("to provide a better experience"), hides who uses the data
  behind a passive sentence, or reads like marketing copy is a **RISK**.
  Quote each weak string in the report with a rewrite that uses only what
  the code actually does.
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
- **In-app selling copy matches the build.** Read the paywall, onboarding,
  upgrade prompts, and every IAP and subscription display name, description
  and review screenshot. List each feature they sell or promise, then find
  it in the code. Start with the features removed or renamed since the live
  version (`git log` for deleted screens, routes and strings): a feature cut
  from the app often stays on the paywall. A paid tier that sells a feature
  the build does not have is a **BLOCKER**: 3.1.2(c) asks that the paywall
  say clearly what the user gets for the price (cite the number from the
  live page).
- Nothing unfinished is visible: no "coming soon", locked placeholder
  options, empty states that look broken, dead buttons, broken links.
- Backend and remote content are live for the review period.
- **Sign-in:** if any feature needs an account, a working demo account (or a
  demo mode) is in the review details. If subscriptions exist, an account
  whose subscription has expired helps the reviewer test the paywall.
- **Hidden or switchable features:** no feature that is hidden from the
  reviewer, dormant, or turned on by remote config after approval. List every
  remote flag you find and what it controls.
- **Originality, copycat and spam:** run the "Originality" checks at the
  end of this phase for every app, not only ones that look generic.
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

### Originality: copycat, minimum functionality and spam

Copycat (4.1), minimum functionality (4.2) and spam (4.3) rejections are
judgement calls, often made from the product page and the first minute of
use, and they are slow to argue after the fact. Cite the numbers from the
live guidelines page; when copying is in question, also read Apple's
https://developer.apple.com/help/app-review/guideline-reference/4-1-prevent-copycat-rejection

1. **Find the closest apps on the store.** Use Apple's public search API (no
   key, read-only):
   `https://itunes.apple.com/search?term=<words>&entity=software&country=<cc>&limit=25`.
   Search with the app's name, its core mechanic or use in plain words
   ("arrow puzzle", "hidden camera detector", "expense tracker"), and its
   main keywords, in the app's primary storefront and in `us`. Each result
   carries `trackName`, `sellerName`, `userRatingCount`, `description`,
   `artworkUrl512` and `screenshotUrls`. Keep the 5 to 10 closest.
2. **Copycat check (4.1).** Compare the app's name, subtitle, icon and first
   screenshots with those apps, the large ones first.
   - Another app's or brand's name, icon or trademark in the app's name or
     icon: **BLOCKER**. In any other metadata field (subtitle, keywords,
     description, IAP names, screenshots): **RISK**.
   - A name that differs from a popular app's by a word, a suffix or word
     order; an icon with the same subject, composition and colors;
     screenshots that copy another app's layout: **RISK**.
   - Search the repo, docs, commit messages and store copy for "clone",
     "remake", "inspired by", "like <app>", "copy of". If the app recreates a
     named game or app, mark **RISK** and ask the user what a player of the
     original would see as different in the first minute.
3. **Same-developer duplicates (4.3).** List the developer's other apps with
   `https://itunes.apple.com/lookup?id=<artistId>&entity=software` (the
   `artistId` comes from the app's own lookup, or ask). Two apps with the
   same core and a different skin, theme, city or content set are a
   **RISK**; the usual fix is one app with the variants inside it. Code from
   a template, white-label kit or reskin (a template name in the source,
   placeholder bundle ids, assets shared with another of the developer's
   apps) raises the risk.
4. **Crowded categories (4.3).** The live 4.3 text names categories Apple
   treats as already saturated; read the current list there. Treat simple
   puzzles, trackers, detectors, timers, wallpapers, soundboards and quiz
   apps the same way. If the app is in one, write one sentence on what a
   reviewer can see in the first minute that the closest apps from step 1
   do not have. If you cannot write it from the build and the screenshots,
   mark **RISK**.
5. **Minimum functionality (4.2).** Not a wrapped website, not a single
   screen, enough content for lasting use (a game with a handful of levels
   is a risk). Note the level or content count you found.
6. **Make the difference visible.** The difference from step 4 appears in
   the review notes and in the first two screenshots. Draft the one or two
   review-note sentences for the user.

In the report, add a "Similar apps" table: name, seller, rating count, what
matches this app, what differs. Most findings here are **RISK**, because a
reviewer decides; never write that an app is safe from 4.1 or 4.3.

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
- **Legal links, inside the app too:** collect every Terms of Use (EULA),
  Privacy Policy and support URL from the store metadata (description, EULA
  field, privacy URL) **and** from the app (search the source and the
  binary's strings for `http`; check the paywall, settings, onboarding and
  sign-up screens). Fetch each one and check that it loads with no 404,
  login wall or parked domain, that it is the real document and not a
  homepage it redirects to, that it names this app and not another one or a
  template, and that the app and the store point to the same documents. A
  broken or wrong legal link on a subscription paywall is a **BLOCKER**
  (see the paywall disclosure check in Phase 5 and Apple's subscriptions
  page, SKILL.md Phase 0, item 7); elsewhere it is a **RISK**.
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
  Compare them with Apple's current "Provide information for a complete
  review" page (SKILL.md Phase 0, item 5), which also asks for the devices
  and OS versions tested, regional differences, and any authentication code
  in advance. Apple says incomplete submissions are rejected.
- **Sensitive permissions in the notes:** for every protected resource the
  app requests (camera, microphone, location, Bluetooth, local network,
  health, contacts, tracking), the notes say which screen asks for it, the
  steps to get there, and what the reviewer should see happen. A permission
  the reviewer cannot trigger from the notes is a **RISK**.
- **Video:** if the app needs external hardware, Apple's complete-review
  page asks for a video of the app running on a physical Apple device as it
  pairs with and uses the hardware, and says a screen recording does not
  count. Flag it as a **BLOCKER** when it is missing. For features that are
  hard to reach or depend on sensors or the environment, suggest a short
  video linked in the notes; the guidelines ask for "supporting
  documentation where appropriate". Attach it in the App Review Information
  section or link it in the notes.
- **Contact info** (name, phone, email) is filled and current.
- **Release type** (manual, automatic, phased) is what the user intends.
- Tooling that writes metadata (e.g. `deliver`) can overwrite review notes or
  the release type on its next run. If the project uses such tooling, check
  its config for review information and release settings and warn if it will
  clobber what is in the console.
- A long time in "Waiting for Review" is normal. Do not recommend withdrawing
  a submission just because it is slow. If a review runs far past the times
  Apple states on its "Request an expedited review" page
  (https://developer.apple.com/help/app-review/after-submitting-for-review/request-expedited-review),
  the routes are Contact App Review, or an expedited review for a genuine
  urgency (a critical bug fix or a dated event, explained in the Notes).
  Read the current wording; do not promise a timeline.

## Phase 9: Play Console state (if Android is in this release)

Store listing in every language, Data safety, content rating questionnaire,
target audience and content, ads declaration, app access (test credentials
for gated features), government/financial/health declarations if relevant,
countries, and the release notes. Same rules as Phase 7 for accuracy.
