# store-ready

Claude Code tools for shipping mobile apps to the App Store and Google Play.

https://github.com/user-attachments/assets/83bde746-f8e0-48e1-a1f8-735a526958ec

| Tool | Type | What it does |
|---|---|---|
| `store-review-audit` | skill | Audits a release before you submit it for review and lists what could get it rejected. |
| `store-review-auditor` | agent | Runs the same audit in its own context, so a long audit does not fill your conversation. |

## store-review-audit

Before you submit a build, it re-reads Apple's and Google's current rules,
goes through the app end to end, and tells you what is likely to get the build
rejected or make the upload fail.

It reads. It does not fix, upload or submit anything.

### Why

App Review rejections usually come from small things: a purpose string an SDK
pulled in, a subscription not attached to the version, a screenshot that
promises a feature the build does not have, a support page that still says
"work in progress". They are cheap to fix before you submit and slow to fix
after.

We ship four apps on iOS and Android (React Native/Expo and Godot). We kept
catching these issues in the last hour before submission, so we wrote the
checks down. The known traps list comes from those near misses and upload
errors.

### What it checks

- **Rules refresh:** fetches the App Review Guidelines, Apple developer news,
  upcoming upload requirements, the SDK privacy manifest list and Google
  Play's policy pages, and tells you what changed since the last audit.
- **Binary:** SDK and Xcode version, purpose strings for every linked
  protected API, privacy manifests, export compliance, icon alpha channel,
  iPad orientations, entitlements, private URL schemes, debug leftovers, and
  whether the build is older than your last fix.
- **Android:** target API level, merged manifest permissions against Data
  safety, iOS-only screens leaking into Android, closed testing for new
  accounts.
- **Completeness:** every advertised feature exists, nothing says "coming
  soon", demo account, hidden or remotely switchable features, spam risk in
  crowded categories, permission prompt timing and wording.
- **Monetization:** IAPs and subscriptions attached to the submission,
  restore, paywall disclosure, Terms and Privacy links, loot box odds, ads
  and ATT.
- **Privacy:** privacy policy against what the app actually collects, App
  Privacy label and Data safety against the SDKs, account deletion, age
  rating, EU DSA trader status.
- **Metadata:** every locale, character limits, prices or platform names in
  copy, screenshot accuracy and duplicates, empty fields after a new version,
  support and marketing pages.
- **Submission state:** no other version in review, the right build, every
  item attached, review notes that match the build, contact info, release
  type.

It also reads your project's own rules (`CLAUDE.md`, checklists, banned word
lists) and enforces them.

You do not need to tell it the version. It reads the version and build from
your project and the newest build file, then asks you to confirm in one line.
When it cannot find something, it asks. For example: "I could not find a
privacy policy in this repo. What is its URL?"

### Output

- A short list in the chat: questions, blockers, risks, what it could not
  verify, and how many checks passed.
- A detailed report in your repo under `review-audits/` (inside `docs/` or
  `documentation/` if you have one), with evidence and the guideline for
  every finding. Your answers are saved next to it in `answers.md`, so the
  next audit does not ask again.

## Install

### Let your AI assistant install it

Copy this into Claude Code, Codex, Cursor or another coding agent:

```text
Install the store-ready tools from https://github.com/oguzhandemiroz/store-ready for me.

- If you are Claude Code, run these two commands in the shell:
  claude plugin marketplace add oguzhandemiroz/store-ready
  claude plugin install store-ready@store-ready
  If the claude command is not available to you, tell me to type these in the session instead:
  /plugin marketplace add oguzhandemiroz/store-ready
  /plugin install store-ready@store-ready
- If you are any other coding agent, run:
  npx skills add oguzhandemiroz/store-ready
  and pick the agent I am using when it asks.
- Do not change anything else in my project or settings.
- When done, show me what was installed and how to start an audit.
```

### Claude Code, by hand

```text
/plugin marketplace add oguzhandemiroz/store-ready
/plugin install store-ready@store-ready
```

From the shell:

```sh
claude plugin marketplace add oguzhandemiroz/store-ready
claude plugin install store-ready@store-ready
```

### Other agents, with skills.sh

```sh
npx skills add oguzhandemiroz/store-ready
```

This installs the skill. The agent is Claude Code only.

### Manual copy

Copy `plugins/store-ready/skills/store-review-audit/` into
`~/.claude/skills/`, and `plugins/store-ready/agents/store-review-auditor.md`
into `~/.claude/agents/`.

## Use

In your app's repo, before you submit, say:

```text
audit this release before I submit
```

or run the skill directly with `/store-ready:store-review-audit`. It works
out the platform, version and build, confirms them with you, and asks the
rest as it goes.

To run it in its own context:

```sh
claude --agent store-review-auditor
```

When the agent is started from inside another session, it cannot ask
questions as it goes. It does every check it can and puts its questions at
the top of its reply.

It works best when it can see the real build (`.ipa`, `.xcarchive`, `.aab`)
and your store metadata (for example `fastlane/metadata`). Without them it
marks those checks as unverified.

## Limits

- It does not guarantee approval. Reviewers judge, and rules change.
- It is not legal advice. It flags privacy and legal risks and points to the
  rule. You decide.
- It cannot see App Store Connect or Play Console unless your project
  already has read-only tooling for them. Otherwise it asks you to confirm.
- It usually cannot run your app. It asks whether you tested a clean install
  of this exact build.

## Safety

The skill tells the model never to edit your code, upload, submit, change
store settings, run git writes or open secrets (`.env`, `.p8`, keystores).
These are written instructions, not a sandbox. For a hard guarantee, add deny
rules to your `.claude/settings.json`, for example:

```json
{
  "permissions": {
    "deny": [
      "Bash(fastlane deliver*)",
      "Bash(fastlane supply*)",
      "Bash(eas submit*)",
      "Bash(xcrun altool*)",
      "Read(./.env*)",
      "Read(**/*.p8)"
    ]
  }
}
```

## Contributing

Store rules change a few times a year. If a check is outdated, or you hit a
rejection the audit would have missed, open an issue with the guideline
number and what happened.

## License

MIT. Made by [Oğuzhan Demiröz](https://github.com/oguzhandemiroz) at
[theworkers.co](https://theworkers.co/?utm_source=github&utm_medium=readme&utm_campaign=store-ready).
