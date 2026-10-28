# Seller Central reports: a skill for Claude

This skill teaches Claude (Claude Code, for example in VS Code) about your Amazon Seller Central reports: where each one is, how to download it or pull it through Amazon's Selling Partner API, what every column means, and the traps in the data. It covers your Sponsored Products ads reports from the Amazon Ads console and Amazon Ads API too. With it installed, Claude can help you read your reports and build dashboards without researching it all first.

## Install it

The easy way, in VS Code with the Claude Code extension: paste this into your browser's address bar and allow it to open VS Code. The Claude panel opens at the install screen: choose **Install for you**. ([Anthropic: plugin install links](https://code.claude.com/docs/en/vs-code))

```
vscode://anthropic.claude-code/install-plugin?plugin=seller-central-reports&marketplace=FBAMogul/amazon-fba-skills
```

Or in a terminal, with Claude Code installed:

```
claude plugin marketplace add FBAMogul/amazon-fba-skills
claude plugin install seller-central-reports@fbamogul
```

Then start a new Claude chat.

Or copy it by hand: copy the `skills/seller-central-reports` folder into one of these, then start a new chat.

- **One project only:** the `.claude/skills/` folder inside your project folder, so you end up with `.claude/skills/seller-central-reports/SKILL.md`. Create the `.claude` and `skills` folders if they aren't there.
- **Every project:** the `.claude/skills/` folder in your home folder (`~/.claude/skills/` on a Mac, `%USERPROFILE%\.claude\skills\` on Windows).

## Use it

Just ask Claude about your reports. It picks the skill up by itself. For example:

- "What does the reimbursements report tell me?"
- "Where do I find my storage fees in Seller Central?"
- "Here's my All Orders report. Build me a dashboard of sales by month."
- "Which report shows stock Amazon lost or damaged?"

To call it up directly, type `/seller-central-reports:seller-central-reports` if you installed the plugin, or `/seller-central-reports` if you copied the folder.

## What it never does

It's knowledge only. It doesn't connect to Amazon, doesn't hold or ask for any keys, and changes nothing in your account. If you want Claude to pull reports for you through the API, `skills/seller-central-reports/references/getting-access.md` walks you through getting your own access and keeping the keys safe.

## Check what matters

This skill was researched in October 2026, and its menu paths and Seller Central downloads were checked on 28 Oct 2026. The ads reports page comes from Amazon's documentation only. Amazon changes its reports, menus and rules, so check anything that matters (money, stock, account health) against Seller Central before you act on it.

Not affiliated with or endorsed by Amazon. Licence: [CC BY 4.0](../../LICENSE).
