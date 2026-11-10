# store-ready

Agent skills for shipping mobile apps to the App Store and Google Play. They
use the open `SKILL.md` format, so they work in Claude Code, Codex, Cursor,
Grok Build, Muse Code, Manus, Gemini CLI, GitHub Copilot and other agents
that read skills.

https://github.com/user-attachments/assets/83bde746-f8e0-48e1-a1f8-735a526958ec

| Tool | Type | Works in | What it does |
|---|---|---|---|
| `store-review-audit` | skill | any agent that reads `SKILL.md` | Audits a release before you submit it for review and lists what could get it rejected. |
| `store-review-auditor` | agent | Claude Code | Runs the same audit in its own context, so a long audit does not fill your conversation. |

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
  protected API and whether each one says what the app does with it,
  privacy manifests, export compliance, icon alpha channel,
  iPad orientations, entitlements, private URL schemes, debug leftovers, and
  whether the build is older than your last fix.
- **Android:** target API level, merged manifest permissions against Data
  safety, iOS-only screens leaking into Android, closed testing for new
  accounts.
- **Completeness:** every advertised feature exists, the paywall and IAP
  copy sell only what the build has, nothing says "coming soon", demo account, hidden or remotely switchable features, permission
  prompt timing and wording.
- **Originality (4.1, 4.2, 4.3):** searches the App Store for the closest
  apps and your own catalog, then compares name, icon and screenshots for
  copycat risk, flags duplicates and crowded categories, and drafts the
  review note that explains what makes your app different.
- **Monetization:** IAPs and subscriptions attached to the submission,
  restore, paywall disclosure, Terms and Privacy links, loot box odds, ads
  and ATT.
- **Privacy:** privacy policy against what the app actually collects, App
  Privacy label and Data safety against the SDKs, account deletion, age
  rating, EU DSA trader status.
- **Metadata:** every locale, character limits, prices or platform names in
  copy, screenshot accuracy and duplicates, empty fields after a new version,
  support and marketing pages, and every Terms and Privacy link in the app
  and the listing opened and read.
- **Submission state:** no other version in review, the right build, every
  item attached, review notes that match the build and show the reviewer
  where each permission is used, a hardware video when one is required,
  contact info, release type.

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

Copy this into whichever coding agent you use:

```text
Install the store-review-audit skill from https://github.com/oguzhandemiroz/store-ready for me.

- If you are Claude Code, run these two commands in the shell:
  claude plugin marketplace add oguzhandemiroz/store-ready
  claude plugin install store-ready@store-ready
  If the claude command is not available to you, tell me to type these in the session instead:
  /plugin marketplace add oguzhandemiroz/store-ready
  /plugin install store-ready@store-ready
- If you are any other coding agent, run:
  npx skills add oguzhandemiroz/store-ready
  and pick the agent I am using when it asks.
- If you cannot run commands, copy the folder plugins/store-ready/skills/store-review-audit
  from the repo into the folder where you load skills from.
- Do not change anything else in my project or settings.
- When done, show me what was installed and how to start an audit.
```

### Claude Code

```text
/plugin marketplace add oguzhandemiroz/store-ready
/plugin install store-ready@store-ready
```

This installs the skill and the agent.

### Codex, Cursor, Grok Build, Gemini CLI, Copilot and others

```sh
npx skills add oguzhandemiroz/store-ready
```

[skills.sh](https://skills.sh) asks which agent you use and puts the skill in
that agent's skills folder. Inside a project, Codex and Cursor read
`.agents/skills/`.

### Muse Code

Install with `npx skills add` and choose Codex or Universal, which writes to
`.agents/skills/`, or choose Claude Code. Muse Code also lists skills from
those folders.

### Manus

Download `store-review-audit.zip` from the
[latest release](https://github.com/oguzhandemiroz/store-ready/releases/latest)
and upload it under Skills, Add, Upload a skill.

### Agensi

The skill is also listed free on
[Agensi](https://www.agensi.io/skills/store-review-audit-app-store-google-play),
which serves it to agents connected through its MCP server. This repo is the
source, and the Agensi copy can lag behind it.

### Any other agent

Copy `plugins/store-ready/skills/store-review-audit/` into the folder your
agent loads skills from. The folder is self-contained: `SKILL.md` and its
`references/`.

## Use

In your app's repo, before you submit, say:

```text
audit this release before I submit
```

or call the skill by name the way your agent does it (in Claude Code,
`/store-ready:store-review-audit`). It works out the platform, version and
build, confirms them with you, and asks the rest as it goes.

In Claude Code you can also run it in its own context:

```sh
claude --agent store-review-auditor
```

In other agents, start the skill in a subagent or background task if your
agent has one. When the audit runs where it cannot ask you questions, it
does every check it can and puts its questions at the top of its reply.

It works best when it can see the real build (`.ipa`, `.xcarchive`, `.aab`)
and your store metadata (for example `fastlane/metadata`). Without them it
marks those checks as unverified. It needs a shell or a web fetch tool to
re-read the store rules; without one it says so and falls back to its dated
snapshot.

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
These are written instructions, not a sandbox. For a hard guarantee, use your
agent's own permission settings. In Claude Code, add deny rules to
`.claude/settings.json`, for example:

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

Other agents have their own approval or sandbox modes. Run the audit in a
read-only or ask-before-running mode if yours has one.

## Contributing

Store rules change a few times a year. If a check is outdated, or you hit a
rejection the audit would have missed, open an issue with the guideline
number and what happened.

## License

MIT. Made by [Oğuzhan Demiröz](https://github.com/oguzhandemiroz) at
[theworkers.co](https://theworkers.co/?utm_source=github&utm_medium=readme&utm_campaign=store-ready).
