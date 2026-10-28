# FBAMogul skills for Claude

Free skills that make Claude genuinely useful for running an Amazon FBA business. Built by [FBAMogul](https://fbamogul.com) for UK sellers who use Claude Code to read their reports, build dashboards and make their own tools. Most of it applies to EU sellers too.

## Why use them

Claude is good at spreadsheets, but it doesn't know Amazon's reports. Give it a Seller Central download and it will guess what the columns mean, and the guesses are often wrong in ways a dashboard won't show you:

- VAT is included in some reports and left out of others, so fee and profit totals can be off by a fifth.
- Some reports write `£` in a SKU as `&pound;`, so joining two reports silently drops rows.
- One file writes dates day-first, the next month-first, and Business Reports stores `£1,234.56` as text.
- Seller Central has just moved its menus, so Claude sends you to pages that aren't there any more.

These skills hand Claude that knowledge up front. Everything in them comes from Amazon's own documentation or was checked against real report files, and each skill says when it was last checked.

They're knowledge only: they don't connect to Amazon, hold no keys, never scrape Amazon's website, and can't change anything in your account.

## Install

The easy way, in VS Code with the Claude Code extension: paste this into your browser's address bar and allow it to open VS Code. The Claude panel opens at the install screen: choose **Install for you**. ([Anthropic: plugin install links](https://code.claude.com/docs/en/vs-code))

```
vscode://anthropic.claude-code/install-plugin?plugin=seller-central-reports&marketplace=FBAMogul/amazon-fba-skills
```

Or in a terminal, with Claude Code installed:

```
claude plugin marketplace add FBAMogul/amazon-fba-skills
claude plugin install seller-central-reports@fbamogul
```

Then start a new Claude chat and ask about your reports. Claude picks the skill up by itself.

Updates don't arrive by themselves: Claude Code leaves auto-update off for marketplaces like this one ([Anthropic](https://code.claude.com/docs/en/discover-plugins#keep-plugins-updated)). To turn it on, run `/plugin`, open the **Marketplaces** tab, select **fbamogul** and choose **Enable auto-update**. Or add this to your `~/.claude/settings.json` ([settings reference](https://code.claude.com/docs/en/settings-reference#extraknownmarketplaces)):

```json
{
  "extraKnownMarketplaces": {
    "fbamogul": {
      "source": { "source": "github", "repo": "FBAMogul/amazon-fba-skills" },
      "autoUpdate": true
    }
  }
}
```

To copy a skill by hand instead, see its own README.

## Skills

| Skill | What Claude learns | Install |
|---|---|---|
| [seller-central-reports](plugins/seller-central-reports/) | Every Seller Central report a UK FBA seller can download or pull through Amazon's API: where it is in the new and classic Seller Central, what every column means, how a Seller Central download differs from the API file, which reports hold buyer details, and the traps seen in real files. Also the Sponsored Products ads reports from the Ads console and Amazon Ads API, and how to get your own private API access and keep the keys safe. | `claude plugin install seller-central-reports@fbamogul` |

## Try asking

- "Here's my Business Report for last month. What were my sales, sessions and conversion rate?"
- "Which report shows stock Amazon lost or damaged, and is anything owed to me?"
- "Build me a dashboard of profit by month from my settlement reports."
- "Where do I find my storage fees in the new Seller Central?"
- "Walk me through getting my own Amazon API access."

## Ground rules the skills give Claude

- Never make up a number: if it isn't in the file, say so.
- Check a figure against the file before any money or stock decision.
- Take buyer names, addresses and emails out before looking at a report.
- Keep your figures and SKUs private.
- Read-only: never change anything in your Amazon account.
- Never ask for, print or store API keys.

## Licence

[CC BY 4.0](LICENSE): use, share and adapt these skills, including commercially, as long as you credit FBAMogul.

Not affiliated with or endorsed by Amazon. Amazon changes its reports, menus and rules, so check anything that matters (money, stock, account health) in Seller Central before you act on it.
