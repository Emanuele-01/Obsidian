![Uno spazio di lavoro futuristico con persone che collaborano attorno a una rete neurale olografica luminosa, circondata da simboli dati fluttuanti in un ambiente elegante ispirato alla tecnologia.](https://miro.medium.com/v2/format:webp/0*rkZchr92_7upXiZw)

Uno spazio di lavoro futuristico con persone che collaborano attorno a una rete neurale olografica luminosa, circondata da simboli dati fluttuanti in un ambiente elegante ispirato alla tecnologia.

I’m a daytime software engineer and a productivity researcher at night, which means I have two chronic problems:

1. I read way too much.
2. I forget the best parts at the exact moment I need them.

Andrej Karpathy’s “LLM knowledge base” workflow is the first approach I’ve seen that treats learning like software engineering: you **ingest raw inputs**, then you **compile** them into something navigable, queryable, and easy to extend.

Not a “second brain” that you have to lovingly maintain by hand. More like a mini Wikipedia that your LLM maintains for you.

This article shows you how to build that system from scratch, in a dummy-proof way, using plain folders, Markdown, Obsidian, and an LLM.

### What you’re building (in one mental picture)

You’re going to build a local folder that contains:

- `raw/`: messy source material you collect (articles, PDFs, repos, images, notes)
- `wiki/`: clean Markdown pages generated and maintained by the LLM
- `outputs/`: answers, slide decks, charts, “research memos” produced from your wiki
- a lightweight “index” so the LLM can navigate without fancy RAG

Then you open the whole thing in **Obsidian**, which becomes your “IDE” for knowledge.

The key idea: **you don’t edit the wiki much. The LLM does.** You mostly curate inputs and ask questions.

### Why Karpathy’s approach works (and most note systems don’t)

Most note systems fail because they rely on you to do three jobs:

1. Capture information
2. Summarize it
3. Organize and link it

You’re good at (1). You’re inconsistent at (2) because time. You’ll procrastinate (3) forever.

Karpathy flips it:

- You do capture (drop stuff in `raw/`)
- The LLM does summarization + organization (compile into `wiki/`)
- Later the LLM does Q&A and generates outputs you can file back into the wiki

It’s basically:

> **knowledge = raw inputs + repeated compilation passes**

Just like code gets better after formatting, linting, refactoring, tests, and documentation, your knowledge base improves through iterative “build steps.”

If you’re interested in exploring more about building an effective personal knowledge management system similar to what I’ve described above, there are some excellent resources available. For a comprehensive guide on [[Gestione Della Conoscenza Personale]], I recommend this article.

Before diving into building your personal knowledge management system though, it’s crucial to understand some foundational aspects which can be found in this piece titled [[Prima di costruire il tuo sistema personale di gestione della conoscenza]].

To further enhance your understanding of how to effectively structure scholarly knowledge within these systems, this comparison of [[Database Notion vs Database a Grafi Strutturare la conoscenza accademica]]

### Step 0: Pick a topic (don’t boil the ocean)

Choose a topic you genuinely care about and that has a clear boundary. Examples include:

- “LLM agents for internal tools”
- “Modern retrieval and reranking”
- “Pricing psychology for SaaS”
- “React Server Components in practice”

If you select a broad topic like “AI”, you’ll end up with a landfill of information. Instead, opt for something with defined edges.

A good rule of thumb is to choose a topic where you can envision creating a **20–100 page wiki**.

### Step 1: Create the dumb folder structure

Establish a folder anywhere, structured like this:

karpathy-kb/ raw/ inbox/ articles/ papers/ repos/ images/ wiki/ concepts/ people/ projects/ glossaries/ outputs/ answers/ slides/ charts/ tools/ README.md

This structure may seem boring, but remember, boring scales.

### What goes where?

- `raw/inbox/`: your temporary dumping ground
- `raw/articles/`: saved web pages as Markdown
- `raw/papers/`: PDFs plus extracted text if desired
- `raw/repos/`: code repos or snippets you’re studying
- `raw/images/`: images referenced by your raw docs
- `wiki/`: only LLM-generated or LLM-maintained pages
- `outputs/`: “products” of your questions (memos, slides, diagrams)

### Step 2: Install Obsidian and open the folder as a vault

Obsidian is an excellent Markdown IDE that offers:

- fast search
- backlinks
- graph view
- good at local folders
- plugins for future enhancements

Open `karpathy-kb/` as a vault. You should be able to navigate through `raw/` and `wiki/`.

Important: **don’t over-customize Obsidian.** You’re building an engine, not a theme.

Incorporating a [[Perché i sistemi di gestione della conoscenza stanno adottando modelli basati su grafi|modello basato sui grafi]] into your knowledge management system can significantly enhance its efficiency. This aligns with the concept of [[Gestione della conoscenza personale basata su grafi AI]], which is becoming increasingly popular.

If you’re interested in building an effective personal knowledge management system, consider following these [7 steps](https://medium.com/@tejas-sharma/7-steps-to-build-the-best-pkms-system-d1836870ab98) that can guide you in establishing the best PKMS system.

### Step 3: Ingest content into raw/ (fast, local, referenceable)

Karpathy mentions a workflow that matters more than it sounds:

- save web articles as Markdown
- download images locally
- keep everything referenceable by your LLM

### Option A (easy): Obsidian Web Clipper

Use Obsidian Web Clipper to save articles straight into `raw/articles/`.

If you can configure it, set:

- Default save location: `raw/articles/`
- Save as: Markdown
- Download images: enabled (or do it manually if needed)

### Option B (still easy): copy/paste + keep URLs

If you can’t clip cleanly, make a Markdown file manually:

`raw/articles/2026-04-llm-agents-foo.md`

Include:

- title
- URL
- date accessed
- the article text (even if messy)
- any key images saved into `raw/images/`

### Option C (papers): PDFs + extracted text

Drop PDFs into `raw/papers/`.

If your LLM struggles with PDFs directly, create a sibling `.md` file with extracted text.

Example:

- `raw/papers/attention-is-all-you-need.pdf`
- `raw/papers/attention-is-all-you-need.md` (title, citation, abstract, extracted text)

Don’t be perfect. The LLM can clean it during compilation.

### Step 4: Decide how the LLM will “compile” the wiki

You need *some* way to run an LLM over your folder and write files.

There are three common approaches, from simplest to most powerful:

### Approach 1: Manual copy/paste into ChatGPT/Claude (works, but slower)

In this method, you paste raw documents (or chunks) into the AI model and ask it to generate `wiki/...` Markdown pages. This approach is suitable for a small knowledge base and for familiarizing yourself with the workflow.

### Approach 2: An editor with file access (Cursor, VS Code + agent, etc.)

This approach involves using an AI coding editor that can read local files, create/update Markdown files, and follow instructions across a repository. This method aligns closely with Karpathy’s vision of an “LLM maintains the wiki” setup.

### Approach 3: CLI agent (more “Karpathy-like”)

Here, you would use a CLI tool where the LLM can scan directories, write files, and run scripts (search, lint, charts). This is the most scalable option but not necessary on day one. For beginners, I recommend sticking with Approach 2.

### Step 5: Create the “constitution” file for your wiki (the secret sauce)

In the root directory, create a `README.md` file that instructs the LLM on its behavior. Here's a starter template you can use:

### Purpose

This repository contains:

- raw/: source documents (messy, ground truth)
- wiki/: compiled markdown wiki maintained by the LLM
- outputs/: generated artifacts from questions (memos, slides, charts)

### Rules for the LLM (compiler)

- Treat raw/ as immutable sources; do not rewrite raw docs unless asked explicitly.
- You may create and update files in wiki/ and outputs/.
- Prefer many small pages over one huge page.

### Every wiki page must include:

- Sources: links to raw files (relative paths)
- Key claims as bullet points
- Open questions / uncertainties
- Related pages (wikilinks)

### Maintain index pages:

- wiki/INDEX.md: high-level map
- wiki/GLOSSARY.md: short definitions
- wiki/SUMMARY.md: 1–2 paragraph overview of the whole wiki

For a more advanced understanding of how to leverage AI tools like ChatGPT or NotebookLM for personal knowledge management, you might find this article on [using AI as your memory](https://medium.com/@theo-james/using-ai-as-your-memory-chatgpt-vs-notebooklm-for-personal-knowledge-9bbda151811c) insightful.

### Writing style

- Clear, terse, technical when needed.
- Use headings, bullets, and examples.
- Avoid filler.

### Linking

- Use Obsidian-style links: \[\[Page Name\]\]
- Store pages under wiki/ with stable names.

### Integrity

- If you’re unsure, say so in the page and list what to verify.
- When two sources conflict, record the conflict explicitly.

This file is how you prevent the KB from becoming a blob.

### Step 6: Run your first compilation pass (turn raw into wiki)

Start small. Put 5–10 raw items in `raw/inbox/` (or `raw/articles/`), then prompt your LLM:

**Compiler prompt (first pass):**

> You are the wiki compiler for this repo. Read the files in `raw/` (especially `raw/inbox/`) and create or update a structured markdown wiki under `wiki/`.
> 
> Create:

- `wiki/INDEX.md` as the navigation hub
- one page per major concept
- one page per source summarizing it with citations to raw paths
- Use Obsidian-style links between related pages.
- Add “Open questions” sections when uncertain.
- Do not edit raw files.

### What the first pass should generate

At minimum, you want:

- `wiki/INDEX.md` (table of contents)
- `wiki/sources/...` pages (one per raw doc)
- `wiki/concepts/...` pages (cross-source synthesis)
- `wiki/GLOSSARY.md` and `wiki/SUMMARY.md`

If it only generates summaries, that’s okay. On pass two, ask it to add concept pages and better linking.

Remember that effective knowledge work involves both compression and expansion, as discussed in this insightful article on [compression and expansion](https://medium.com/@ann_p/compression-and-expansion-the-two-breaths-of-knowledge-work-e6a36ad40a30).

### Step 7: The “backlinks and categorization” upgrade

Once you have 20+ pages, you’ll notice the KB starts to feel like a real system if you add:

- categories (concepts, methods, datasets, people, tools)
- backlinks (“pages that mention this page”)
- “see also” sections
- “related concepts” lists

This is similar to refactoring code after it’s working.

### The “modern tool” problem

At this point, you might be experiencing what I felt during my initial attempts with this workflow. It’s powerful, but why does it seem like I’m constantly juggling apps, tabs, prompts, and incomplete exports?

Are you in search of a modern tool that seamlessly integrates all of this in a way that aligns with your thought process? If you’re tired of the friction caused by switching between apps, then you’re in luck. [Constella](https://www.constella.app/?utm_source=medium&utm_medium=referral&utm_campaign=theo&utm_content=how-to-build-karpathy-llm-from-scratch) has graciously sponsored this article to facilitate my writing.

Do you often find yourself spending weeks researching a topic and attempting to connect thoughts across various domains only to still face difficulties? Whether for academic purposes, personal growth or corporate research for clients, many AI tools generate convoluted results that are hard to decipher. However, [Constella](https://www.constella.app/?utm_source=medium&utm_medium=referral&utm_campaign=theo&utm_content=how-to-build-karpathy-llm-from-scratch) stands out as a benchmarked deep researcher that not only shows its work on the left but also helps you quickly grasp it by highlighting key findings from around the world. It’s akin to Google Maps, but instead of navigating roads, it’s designed for traversing the complex ideas we’ve created. It can significantly reduce your research time and streamline your second brain organization into mere minutes.

Now, let’s return to constructing the DIY version because understanding the mechanics behind this process makes you a formidable force in a positive way.

To further enhance your knowledge management system, consider implementing some [modern knowledge management strategies](https://medium.com/@ann_p/elevate-your-professional-growth-with-modern-knowledge-management-strategies-d7fd3b8df7ca). These strategies can significantly improve your personal knowledge management system’s effectiveness as discussed in this article about [setting up a personal knowledge management system](https://medium.com/@tejas-sharma/is-setting-up-a-personal-knowledge-management-system-worth-it-872824b20030).

Moreover, mastering [the art of information](https://medium.com/@theo-james/mastering-the-art-of-information-a-practical-guide-to-personal-knowledge-management-a835790b63a6) through practical guides can also be beneficial. Remember to include [essential elements](https://medium.com/@tejas-sharma/the-essential-elements-of-the-best-personal-knowledge-management-system-011e5aa8cbe9) in your personal knowledge management system for optimal results.

Finally, exploring [the best personal knowledge management apps](https://medium.com/@ann_p/best-personal-knowledge-management-apps-a-comprehensive-guide-b4627261f0d1) could provide you with valuable tools to streamline your processes.

### Step 8: Ask questions against the wiki (without fancy RAG)

Karpathy’s point is subtle: at a “small scale” (say 100 articles, a few hundred thousand words), a strong model can do Q&A by:

- reading `wiki/INDEX.md`
- reading `wiki/SUMMARY.md`
- scanning relevant pages
- following links to sources

So you can often skip vector databases initially.

### The minimum viable Q&A loop

1. You ask a question.
2. The LLM reads the index and finds relevant pages.
3. It writes an answer as a new file under `outputs/answers/`.
4. You decide whether to “file” it back into `wiki/`.

Example prompt:

> Question: “What are the main failure modes of tool-using LLM agents in production, and what mitigations are supported by the sources in this wiki?”
> 
> Use the wiki to answer. Cite relevant wiki pages and raw sources by path.
> 
> Output a markdown memo to `outputs/answers/agent-failure-modes.md` with:

- Executive summary
- Failure mode list
- Mitigations
- What’s uncertain / missing

This output-first approach is huge for productivity. It keeps answers from evaporating in chat logs.

### Step 9: Generate “rendered” outputs (slides, charts, diagrams)

Karpathy likes answers that become artifacts:

- Markdown pages
- Marp slide decks
- matplotlib charts (saved as images)
- diagrams

### Slides (Marp)

Create:

`outputs/slides/agent-failure-modes-deck.md`

Prompt:

> Create a Marp slide deck summarizing the memo. Keep it to 10 slides, include a crisp diagram slide using ASCII or Mermaid. Output in Marp markdown.

In Obsidian, you can install a Marp plugin, or just keep it as Markdown and render later.

### Charts (simple and effective)

If you have numeric or categorical data (even manually extracted), have the LLM write a small script in `tools/` that generates a PNG into `outputs/charts/`.

Example:

- `tools/plot_failures.py`
- `outputs/charts/failure-modes.png`

Then reference the image in the wiki.

### Step 10: File the outputs back into the wiki (so your research “adds up”)

This is the compounding effect of knowledge management. It’s essential to ensure that knowledge is not just collected, but also organized and made easily accessible. As highlighted in this article on [personal data gravity](https://medium.com/@ann_p/personal-data-gravity-why-your-knowledge-should-be-local-first-linked-and-ai-interpretable-dd1968ddaf67), your knowledge should be local-first, linked, and AI interpretable.

If an output memo is good, don’t leave it in `outputs/` forever. Convert it into a durable page:

- `wiki/concepts/Agent Failure Modes.md`
- link to sources
- incorporate your output
- add open questions

Prompt:

> Take `outputs/answers/agent-failure-modes.md` and integrate it into the wiki as a durable concept page. Keep claims grounded in sources. Add missing citations and list claims that are not supported yet.

Now your questions become permanent improvements to the KB.

### Step 11: Run “linting” health checks (the knowledge equivalent of tests)

This is where it gets fun, because you can treat your wiki like a codebase:

- find contradictions
- find missing pages
- find orphan pages (no links)
- find claims without sources
- propose new pages to write
- propose new data to ingest

### A simple health check prompt

> Perform a health check on the wiki:

- List orphan pages (not linked from INDEX)
- List pages missing “Sources” section
- List conflicting definitions in GLOSSARY
- Identify 10 suggested new pages that would improve coverage Output to `outputs/answers/wiki-health-check.md`.

This process of [knowledge organization](https://medium.com/@ann_p/forget-tags-think-context-the-next-layer-of-knowledge-organization-6fae6a4e2ac2) not only helps in identifying gaps but also aids in structuring information better for future use. By leveraging tools for effective personal knowledge management, as discussed in this piece on [PKM tools](https://medium.com/@ann_p/8-best-pkm-tools-for-effective-personal-knowledge-management-6f4b547cb4ac), we can enhance our ability to manage and utilize our knowledge efficiently.

### Add “unit tests” for important facts

If your knowledge base (KB) is for work, some facts matter a lot. Create a page like:

`wiki/ASSERTIONS.md`

It can include lines like:

- “In our stack, retrieval uses BM25 + reranker”
- “Latency budget is 800ms p95”
- “Primary user is internal analysts”

Then ask the LLM to check if any wiki pages contradict them.

This approach aligns with the evolving trends in [personal knowledge management tools](https://medium.com/@tejas-sharma/the-future-of-personal-knowledge-management-tools-and-trends-to-watch-f0242d66d2b6), where such unit tests can enhance the reliability of the information stored.

### Step 12: Add tiny tools (search, grep, map)

Karpathy mentions vibe-coding a naive search engine.

You don’t need fancy. Even a crude tool can massively improve an agent’s effectiveness.

### Minimal tools that help immediately

- a script to list all wiki pages with titles and first paragraph
- a script to grep for a term across wiki
- a script to output a “site map” tree

Example:

`tools/wiki_map.py` outputs `outputs/wiki-map.txt`.

Then the LLM can call it and navigate faster.

If you’re using an agentic coding environment, you can say:

> If you need to find information, run `python tools/wiki_map.py` or grep the wiki before guessing.

These simple tools are reminiscent of the functionalities offered by advanced [personal knowledge management systems](https://medium.com/@theo-james/a-guide-to-personal-knowledge-management-tools-9bb791aed367) which often include features like efficient search and mapping of information.

### Common beginner mistakes (and how to avoid them)

### Mistake 1: Making raw/ too clean

Raw is allowed to be messy. That’s the point. Don’t waste energy perfecting it.

### Mistake 2: Letting the LLM write “pretty” summaries with no citations

Force it to include a **Sources** section with raw file paths.

Avoiding these common pitfalls will ensure that your KB remains a valuable resource. It’s also worth exploring potential [Siyuan alternatives for personal knowledge management](https://medium.com/@theo-james/siyuan-alternatives-personal-knowledge-management-tools-in-2025-f85be7351f45) which might offer more tailored solutions for your specific needs.

### Mistake 3: Giant pages

Big pages feel productive but quickly become unusable. It’s preferable to have small, linkable pages.

### Mistake 4: No index

Without a proper `wiki/INDEX.md`, the Knowledge Base (KB) turns into a junk drawer.

### Mistake 5: Treating the chat as the product

If the output isn’t a file, it basically doesn’t exist.

### A practical “dummy” weekly workflow (copy this)

### Daily (5–15 minutes)

- Clip 1–3 good articles/papers into `raw/inbox/`
- Save any key images locally

### Twice per week (30–60 minutes)

Run a compilation pass with these tasks:

- Summarize new raw docs into `wiki/sources/`
- Update `wiki/INDEX.md` and `wiki/SUMMARY.md`
- Create or update 1–3 concept pages

### Weekly (30 minutes)

- Ask 1 big question and generate an output memo
- File the memo into the wiki if it’s good
- Run a health check and fix the top 3 issues

This is how you get to 100 pages without it feeling like a second job.

### When to “graduate” to RAG or a database

You can often avoid fancy RAG until:

- your wiki is too large for the model to navigate reliably
- you want automated retrieval with strict recall targets
- you need multi-user collaboration with permissions
- you want structured queries over entities/relationships

A good milestone is when:

- `wiki/` hits 500+ pages, or
- you frequently notice missed relevant pages, or
- you’re doing this for a team and not just you

Until then, strong indexing, summaries, and links go surprisingly far.

For those looking to enhance their personal knowledge management systems, exploring options like [the best mobile PKMS](https://medium.com/@theo-james/the-best-mobile-pkmses-for-2025-capture-knowledge-on-the-go-fafc0059f8f2) for 2025 could be beneficial. These systems allow for efficient knowledge capture on-the-go, making your workflow even more streamlined.

### Final checklist (if you do nothing else)

If you want the Karpathy-style loop working, make sure you have:

- `raw/` with real sources (not just your notes)
- Obsidian vault pointed at the repo
- `wiki/INDEX.md` that’s updated
- one wiki page per source with citations
- concept pages that synthesize across sources
- outputs written to files, not chat bubbles
- periodic “health checks” to fix drift

That’s it. You now have a knowledge base that behaves like a codebase: it compiles, it refactors, it produces artifacts, and it gets better every time you use it. This approach is similar to my secret to turning notes into a [knowledge powerhouse](https://medium.com/@ann_p/my-secret-to-turning-notes-into-a-knowledge-powerhouse-and-how-i-turn-it-into-content-6833287d97cb). However, it’s important to note that achieving such transformations requires more than just using the right tools; it also necessitates fostering a [culture around knowledge work](https://medium.com/@ann_p/not-another-tool-why-knowledge-work-needs-culture-not-just-apps-e5136d4d2910).

### FAQs (Frequently Asked Questions)

### What is Andrej Karpathy’s LLM knowledge base workflow and how does it improve learning?

Andrej Karpathy’s LLM knowledge base workflow treats learning like software engineering by ingesting raw inputs and compiling them into a navigable, queryable, and easily extendable mini Wikipedia maintained by an LLM. This approach reduces manual maintenance and helps capture, summarize, organize, and retrieve knowledge efficiently.

### How does Karpathy’s approach differ from traditional note-taking systems?

Traditional note systems require you to capture information, summarize it, and organize/link it manually. Karpathy flips this by having you only capture raw inputs while the LLM handles summarization, organization, and Q&A. This iterative compilation improves your knowledge base similarly to how code improves through formatting and refactoring.

### What folder structure should I use to build a local knowledge base following Karpathy’s method?

Create a folder (e.g., karpathy-kb/) with subfolders: raw/ (inbox/, articles/, papers/, repos/, images/), wiki/ (concepts/, people/, projects/, glossaries/), outputs/ (answers/, slides/, charts/, tools/), plus a README.md. Raw contains messy source material; wiki holds clean markdown pages generated by the LLM; outputs store final products like memos or slides.

### Why should I use Obsidian as my knowledge management IDE in this system?

Obsidian offers fast search, backlinks, graph view, excellent support for local folders, and useful plugins. Opening your knowledge base folder as a vault lets you navigate raw and wiki materials seamlessly without over-customizing. It serves as an effective Markdown IDE to manage your compiled knowledge efficiently.

### How do I choose the right topic scope for my personal knowledge management system?

Pick a topic you genuinely care about with clear boundaries — avoid overly broad topics like ‘AI’. Aim for something where you can envision creating a 20–100 page wiki. Examples include ‘LLM agents for internal tools’ or ‘Pricing psychology for SaaS’. This focused scope prevents information overload and landfill of data.

### What are some additional resources to learn about building effective personal knowledge management systems?

Recommended resources include articles like ‘How to Build an Effective Personal Knowledge Management System in 7 Easy Steps’, ‘Before You Build Your Personal Knowledge Management System Read This First’, and comparisons such as ‘Notion Databases vs Graph Databases: Structuring Scholarly Knowledge’. These provide foundational insights and advanced structuring techniques including graph-based models and AI integration.