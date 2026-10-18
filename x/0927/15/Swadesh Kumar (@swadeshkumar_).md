---
author: "Swadesh Kumar"
handle: "@swadeshkumar_"
source: "https://x.com/swadeshkumar_/status/2103311773390807428"
date: 2026-09-25T02:32:59.000Z
type: tweet
likes: 289
reposts: 36
replies: 19
bookmarks: 405
views: 20768
---
# Swadesh Kumar (@swadeshkumar_)

Most people think using Claude Code is about writing better prompts.  
  
It’s not.  
  
The real unlock is structuring your repository so Claude can think like an engineer.  
  
If your repo is messy, Claude behaves like a chatbot.  
  
If your repo is structured, Claude behaves like a developer living inside your codebase.  
  
Your project only needs 4 things:  
  
• the why → what the system does  
• the map → where things live  
• the rules → what’s allowed / forbidden  
• the workflows → how work gets done  
  
I call this:  
  
The Anatomy of a Claude Code Project 👇  
  
━━━━━━━━━━━━━━━  
  
1️⃣ CLAUDE.md = Repo Memory (Keep it Short)  
  
This file is the north star for Claude.  
  
Not a massive document.  
  
Just three things:  
  
• Purpose → why the system exists  
• Repo map → how the project is structured  
• Rules + commands → how Claude should operate  
  
If CLAUDE.md becomes too long, the model starts missing critical signals.  
  
Clarity beats size.  
  
━━━━━━━━━━━━━━━  
  
2️⃣ .claude/skills/ = Reusable Expert Modes  
  
Stop repeating instructions in prompts.  
  
Turn common workflows into reusable skills.  
  
Examples:  
  
• code review checklist  
• refactoring playbook  
• debugging workflow  
• release procedures  
  
Now Claude can switch into specialized modes instantly.  
  
Result:  
  
More consistent outputs across sessions and teammates.  
  
━━━━━━━━━━━━━━━  
  
3️⃣ .claude/hooks/ = Guardrails  
  
Models forget.  
  
Hooks don’t.  
  
Use hooks for things that must always happen automatically.  
  
Examples:  
  
• run formatters after edits  
• trigger tests after core changes  
• block sensitive directories (auth, billing, migrations)  
  
Hooks turn AI workflows into reliable engineering systems.  
  
━━━━━━━━━━━━━━━  
  
4️⃣ docs/ = Progressive Context  
  
Don’t overload prompts with information.  
  
Instead, let Claude navigate your documentation.  
  
Examples:  
  
• architecture overview  
• ADRs (engineering decisions)  
• operational runbooks  
  
Claude doesn’t need everything in memory.  
  
It just needs to know where truth lives.  
  
━━━━━━━━━━━━━━━  
  
5️⃣ Local CLAUDE.md for Critical Modules  
  
Some areas of your system have hidden complexity.  
  
Add local context files there.  
  
Example:  
  
src/auth/CLAUDE.md  
src/persistence/CLAUDE.md  
infra/CLAUDE.md  
  
Now Claude understands the danger zones exactly when it works in them.  
  
This dramatically reduces mistakes.  
  
━━━━━━━━━━━━━━━  
  
Here’s the shift most people miss:  
  
Prompting is temporary.  
  
Structure is permanent.  
  
Once your repository is designed for AI:  
  
Claude stops acting like a chatbot...  
  
…and starts behaving like a project-native engineer. 🚀  
  
Follow @swadeshkumar_ for more !!

![Image](<Swadesh%20Kumar%20(@swadeshkumar_).assets/image-01.jpg>)