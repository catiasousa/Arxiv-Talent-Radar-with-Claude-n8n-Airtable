# PROMPT.md

Paste the prompt below into the n8n AI Assistant to build the **arXiv Talent Radar** workflow from scratch. After it builds, connect your Anthropic, GitHub, and Airtable credentials and set your Airtable base/table (see `README.md`).

---

Build an n8n workflow called "arXiv Talent Radar" that sources engineering candidates from new arXiv papers. Use these nodes, wired in a single line in this order:

1. **RSS Feed Read Trigger** named "arXiv cs.LG Feed"
   - Feed URL: `http://arxiv.org/rss/cs.LG`
   - Poll once per day at 06:00.

2. **Limit** named "Limit to 2 (Test)"
   - maxItems: 2, keep firstItems.
   - Add a sticky note next to it saying this is a test guard that caps the run to 2 papers, and to delete it (or raise maxItems) for production.

3. **HTTP Request** named "Claude Relevance Check"
   - POST to `https://api.anthropic.com/v1/messages`
   - Authentication: predefined credential type `anthropicApi`.
   - Headers: `anthropic-version: 2023-06-01` and `content-type: application/json`.
   - JSON body using model `claude-haiku-4-5-20251001`, max_tokens 400.
   - Use exactly this expression for the JSON body field (paste into the n8n expression editor):
   \`\`\`
   ={{
   {
     "model": "claude-haiku-4-5-20251001",
     "max_tokens": 400,
     "system": "You are a technical recruiting assistant. Read arXiv paper titles and abstracts and decide if the authors would be relevant candidates for a specific engineering role. Be strict — only say relevant=true if the paper is directly relevant. Return only valid JSON with no extra text.",
     "messages": [
       {
         "role": "user",
         "content":
           "Read this arXiv paper and decide if the authors are relevant candidates for my role.\n\n" +
           "ROLE I AM HIRING FOR:\n" +
           "Senior ML Engineer. Must have: PyTorch or JAX, LLM training or fine-tuning, distributed systems knowledge\n\n" +
           "PAPER:\n" +
           "Title: " + ($json.title || "") + "\n" +
           "Abstract: " + ($json.contentSnippet || "") + "\n" +
           "Authors: " + ($json.creator || "") + "\n" +
           "Published: " + ($json.pubDate || "") + "\n" +
           "URL: " + ($json.link || "") + "\n\n" +
           "Return this exact JSON:\n" +
           "{\n" +
           "  \"relevant\": true or false,\n" +
           "  \"relevance_score\": 0-10,\n" +
           "  \"reason\": \"one sentence\",\n" +
           "  \"first_author_name\": \"first author full name\",\n" +
           "  \"all_authors\": [\"name1\", \"name2\"],\n" +
           "  \"what_they_built\": \"one sentence on what they built or proved\",\n" +
           "  \"outreach_hook\": \"one sentence cold email opener referencing this paper\",\n" +
           "  \"paper_url\": \"" + ($json.link || "") + "\"\n" +
           "}"
       }
     ]
   }
   }}
   \`\`\`

4. **Code** node named "Parse Relevance" (run once for each item):
   \`\`\`js
   const raw = $json.content[0].text;
   const cleaned = raw.replace(/```json|```/g, '').trim();
   const result = JSON.parse(cleaned);
   return { json: result };
   \`\`\`

5. **IF** node named "Is it Relevant?"
   - Condition: `{{$json.relevant}}` is boolean true.

6. **HTTP Request** named "Find Author on GitHub" (on the true branch)
   - GET `https://api.github.com/search/users?q={{ encodeURIComponent($json.first_author_name) }}&type=users&per_page=3`
   - Authentication: predefined credential type `githubApi`.
   - Header: `Accept: application/vnd.github+json`.

7. **Code** node named "Build Candidate Record" (run once for each item):
   \`\`\`js
   const gh = $json;
   const items = Array.isArray(gh.items) ? gh.items : [];
   const match = items.length ? items[0] : null;
   const claude = $('Parse Relevance').item.json;
   const paper = $('arXiv cs.LG Feed').item.json;
   return { json: {
     full_name: claude.first_author_name || '',
     github_url: match ? match.html_url : '',
     github_login: match ? match.login : '',
     fit_score: (claude.relevance_score != null ? claude.relevance_score : ''),
     what_they_built: claude.what_they_built || '',
     reason: claude.reason || '',
     outreach_hook: claude.outreach_hook || '',
     paper_url: claude.paper_url || paper.link || '',
     paper_title: paper.title || ''
   }};
   \`\`\`

8. **Airtable** node named "Save to Airtable"
   - Resource: record, Operation: create.
   - Base: `YOUR_AIRTABLE_BASE_ID`, Table: `YOUR_AIRTABLE_TABLE_ID` (I will set these).
   - Map fields: `full_name` = `{{$json.full_name}}`, `github_url` = `{{$json.github_url}}`, `source` = "arXiv", `date_sourced` = `{{$now.toISO()}}`, `fit_score` = `{{$json.fit_score}}`, `key_strengths` = `{{$json.what_they_built}}`, `key_gaps` = `{{$json.reason}}`, `outreach_hook` = `{{$json.outreach_hook}}`, `profile_summary` = `Paper: {{$json.paper_title}} — {{$json.paper_url}}`.
   - Enable typecast.

Do not hardcode any API keys — use n8n credentials for all authenticated nodes.
