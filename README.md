# BBC-News-Daily-Digest-n8n-Automation-Workflow

An automated workflow built with n8n that scrapes live news from BBC every morning, formats it into a numbered digest, saves it to Google Docs, and delivers it to your inbox via Gmail — fully hands-free.

🔁 Workflow Overview
```
Schedule Trigger → HTTP Request → HTML Extract → Code (JS) → Google Docs → Gmail
```
1. Schedule Trigger --> Fires at 7:00 AM daily via cron 0 7 * * *
2. HTTP Request --> Fetches live HTML from https://www.bbc.com/news
3. HTML Extract --> Parses headlines, summaries using CSS selectors
4. Code in JavaScript --> Deduplicates, aligns, and formats into numbered list
5. Update a Document --> Writes the digest to a Google Doc
6. Send a Message --> Emails the formatted digest via Gmail

⚙️ Setup & Configuration
Prerequisites

- n8n (self-hosted or cloud)
- Google account with Docs + Gmail OAuth2 credentials configured in n8n
- Basic familiarity with n8n canvas

CSS Selectors used (HTML Extract node)
Headline --> h2[data-testid="card-headline"] --> Text
Summary --> p[data-testid="card-description"] --> Text
Section label --> span[data-testid="card-metadata-tag"] --> Text
Timestamp --> span[data-testid="card-metadata-lastupdated"] --> Text

Code Node (JavaScript)
Deduplicates headlines and aligns summaries row-by-row, then formats the output as clean numbered HTML:
```
jsconst data = $input.first().json;
const headlines = data['Headline'] || [];
const summaries = data['Summary / description'] || [];

const seen = new Set();
const unique = [];

for (let i = 0; i < headlines.length; i++) {
  const title = headlines[i]?.replace(/\n/g, ' ').trim();
  if (!title || seen.has(title)) continue;
  seen.add(title);
  unique.push({ title, summary: (summaries[unique.length] || '').replace(/\n/g, ' ').trim() });
  if (unique.length === 15) break;
}

const today = new Date().toLocaleDateString('en-GB', {
  weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'
});

const formattedText = `<h2>BBC News Digest — ${today}</h2><hr>` +
  unique.map((item, i) =>
    `<p><b>${i + 1}. ${item.title}</b><br>${item.summary || 'No summary available.'}</p>`
  ).join('');

return [{ json: { formattedText, today } }];
```

Gmail node — Message field expression
```
{{ $json.formattedText }}
```

📬 Sample Output
BBC News Digest — Sunday, 15 March 2026
────────────────────────────────────────

1. Israel launches 'wide-scale' strikes on Iran
   A relative tells BBC those killed were civilians...

2. Antonelli wins Chinese GP as McLarens fail to start
   Kimi Antonelli takes his first Formula 1 victory...

3. Severe flooding kills 62 in Kenya
   Eleven people were rescued overnight after a minibus...



🛠️ Tech Stack

n8n — workflow automation
BBC News — live news source (public HTML)
Google Docs API — document storage
Gmail API — email delivery


📌 Notes

BBC's homepage is partially JS-rendered; using /news subpage improves static HTML availability
Headlines and summaries are scraped independently and re-aligned in the Code node
Duplicate headlines are filtered before sending
Workflow runs once daily but can be adjusted to any cron schedule
