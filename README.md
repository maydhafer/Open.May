![Project Header](May%20Dhafer.png)
بوت كود كلاود (Claude Code) على تليجرام 🤖
مشروع يخليك تسولف مع Claude Code القوي من جوالك وفي أي وقت.

وش فكرة المشروع؟
تخيل عندك مبرمج محترف (اللي هو Claude Code) شغال على جهازك، هذا المشروع يسوي "جسر" يخليك ترسل له رسائل من تليجرام وهو يرد عليك. يعني كأنك فاتح الـ Terminal من جوالك وقاعد تأمر كلاود يسوي لك اللي تبيه.

ليش هذا البوت رهيب؟
كامل الصلاحيات: يستخدم نسخة الـ CLI الكاملة، يعني يقدر يوصل لملفاتك، ويستخدم أدوات البرمجة، وخوادم الـ MCP.

شغال دايماً: فيه ملفات تخلي البوت يشتغل بالخلفية وما يطفي حتى لو قفلت الجهاز.

أمان عالي: تقدر تبرمجه بحيث ما يرد إلا عليك أنت بس (عن طريق الـ User ID).

كيف تشغله بـ ٤ خطوات؟
حمل الكود: اسحب الملفات لحسابك.

ثبت المكتبات: شغل أمر bun install.

جهز مفاتيحك: حط توكن البوت حق تليجرام في ملف الـ .env.

انطلق: شغل البوت بأمر bun run src/relay.ts.

وش تقدر تسوي فيه؟
برمجة سريعة: تطلب منه يكتب كود أو يصلح خطأ وأنت برا البيت.

ملخص صباحي: يرسل لك كل يوم ملخص لمهامك وإيميلاتك (موجود مثال جاهز في ملف examples).

ذاكرة قوية: يتذكر وش قلت له قبل عشان يكمل معك السالفة.

تطوير: May Dhafer (مبني على فكرة Goda الأصلية). الرخصة: MIT - يعني خذ الكود وعدل عليه براحتك!


# Claude Code Telegram Relay

**A pattern for running Claude Code as an always-on Telegram bot.**

> **This is a reference implementation, not a copy-paste solution.** Take the patterns here and build your own system tailored to your needs.

## What This Is

A minimal relay that connects Telegram to Claude Code CLI. You send a message on Telegram, the relay spawns `claude -p`, and sends the response back. That's it.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Telegram   │────▶│    Relay     │────▶│  Claude CLI  │
│    (you)     │◀────│  (always on) │◀────│   (spawned)  │
└──────────────┘     └──────────────┘     └──────────────┘
```

## Why This Approach?

| Approach | Pros | Cons |
|----------|------|------|
| **This (CLI spawn)** | Simple, uses full Claude Code capabilities, all tools available | Spawns new process per message |
| Claude API direct | Lower latency | No tools, no MCP, no context |
| Claude Agent SDK | Production-ready, streaming | More complex setup |

The CLI spawn approach is the simplest way to get Claude Code's full power (tools, MCP servers, context) accessible via Telegram.

## Requirements

- [Bun](https://bun.sh/) runtime (or Node.js 18+)
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed and authenticated
- Telegram Bot Token (from [@BotFather](https://t.me/BotFather))
- Your Telegram User ID (from [@userinfobot](https://t.me/userinfobot))

## Quick Start

```bash
# Clone (or fork and customize)
git clone https://github.com/YOUR_USERNAME/claude-telegram-relay
cd claude-telegram-relay

# Install dependencies
bun install

# Copy and edit environment variables
cp .env.example .env
# Edit .env with your tokens

# Run
bun run src/relay.ts
```

## Cross-Platform "Always On" Setup

The relay needs to run continuously. Here's how on each platform:

### macOS (LaunchAgent)

LaunchAgent keeps the bot running and restarts it if it crashes.

```bash
# Copy the template
cp daemon/launchagent.plist ~/Library/LaunchAgents/com.claude.telegram-relay.plist

# Edit paths in the plist to match your setup
nano ~/Library/LaunchAgents/com.claude.telegram-relay.plist

# Load it
launchctl load ~/Library/LaunchAgents/com.claude.telegram-relay.plist

# Check status
launchctl list | grep claude

# View logs
tail -f ~/Library/Logs/claude-telegram-relay.log
```

**To stop:** `launchctl unload ~/Library/LaunchAgents/com.claude.telegram-relay.plist`

### Linux (systemd)

```bash
# Copy the template
sudo cp daemon/claude-relay.service /etc/systemd/system/

# Edit paths and user
sudo nano /etc/systemd/system/claude-relay.service

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable claude-relay
sudo systemctl start claude-relay

# Check status
sudo systemctl status claude-relay

# View logs
journalctl -u claude-relay -f
```

### Windows (Task Scheduler)

**Option 1: Task Scheduler (built-in)**

1. Open Task Scheduler (`taskschd.msc`)
2. Create Basic Task
3. Trigger: "When the computer starts"
4. Action: Start a program
   - Program: `C:\Users\YOU\.bun\bin\bun.exe`
   - Arguments: `run src/relay.ts`
   - Start in: `C:\path\to\claude-telegram-relay`
5. In Properties, check "Run whether user is logged on or not"
6. In Settings, check "Restart if the task fails"

**Option 2: PM2 (recommended)**

PM2 works on all platforms and handles restarts, logs, and monitoring.

```bash
# Install PM2
npm install -g pm2

# Start the relay
pm2 start src/relay.ts --interpreter bun --name claude-relay

# Save the process list
pm2 save

# Setup startup script (run the command it outputs)
pm2 startup
```

**Option 3: NSSM (Windows Service)**

[NSSM](https://nssm.cc/) turns any script into a Windows service.

```bash
# Download NSSM, then:
nssm install claude-relay "C:\Users\YOU\.bun\bin\bun.exe" "run src/relay.ts"
nssm set claude-relay AppDirectory "C:\path\to\claude-telegram-relay"
nssm start claude-relay
```

## Architecture

```
src/
  relay.ts          # Core relay (what you customize)

examples/
  morning-briefing.ts   # Scheduled daily summary
  smart-checkin.ts      # Proactive check-ins
  memory.ts             # Persistent memory pattern
  supabase-schema.sql   # Optional: cloud persistence

daemon/
  launchagent.plist     # macOS daemon config
  claude-relay.service  # Linux systemd config
```

## The Core Pattern

The relay does three things:

1. **Listen** for Telegram messages
2. **Spawn** Claude CLI with the message
3. **Send** the response back

```typescript
// Simplified core pattern
bot.on("message:text", async (ctx) => {
  const response = await spawnClaude(ctx.message.text);
  await ctx.reply(response);
});

async function spawnClaude(prompt: string): Promise<string> {
  const proc = spawn(["claude", "-p", prompt, "--output-format", "text"]);
  const output = await new Response(proc.stdout).text();
  return output;
}
```

That's the entire pattern. Everything else is enhancement.

## Enhancements You Can Add

### Security (Required)
```typescript
// Only respond to your user ID
if (ctx.from?.id.toString() !== process.env.TELEGRAM_USER_ID) {
  return; // Ignore unauthorized users
}
```

### Session Continuity
```typescript
// Resume conversations with --resume
const proc = spawn([
  "claude", "-p", prompt,
  "--resume", sessionId,  // Continue previous conversation
  "--output-format", "text"
]);
```

### Voice Messages
```typescript
// Transcribe with Whisper/Gemini, send to Claude
const transcription = await transcribe(voiceFile);
const response = await spawnClaude(`[Voice message]: ${transcription}`);
```

### Images
```typescript
// Claude Code can see images if you pass the path
const response = await spawnClaude(`Analyze this image: ${imagePath}`);
```

### Persistent Memory
```typescript
// Add context to every prompt
const memory = await loadMemory();
const fullPrompt = `
Context: ${memory.facts.join(", ")}
Goals: ${memory.goals.join(", ")}

User: ${prompt}
`;
```

### Scheduled Tasks
```typescript
// Run briefings via cron/launchd
// See examples/morning-briefing.ts
```

## Examples Included

### Morning Briefing (`examples/morning-briefing.ts`)

Sends a daily summary at a scheduled time:
- Unread emails
- Calendar for today
- Active goals
- Whatever else you want

Schedule it with cron (Linux), launchd (Mac), or Task Scheduler (Windows).

### Smart Check-in (`examples/smart-checkin.ts`)

Proactive assistant that checks in based on context:
- Time since last message
- Pending goals with deadlines
- Calendar events coming up

Claude decides IF and WHAT to say.

### Memory Persistence (`examples/memory.ts`)

Pattern for remembering facts and goals across sessions:
- Local JSON file (simple)
- Supabase (cloud, searchable)
- Any database you prefer

## Environment Variables

```bash
# Required
TELEGRAM_BOT_TOKEN=       # From @BotFather
TELEGRAM_USER_ID=         # From @userinfobot (for security)

# Optional - Paths (defaults work for most setups)
CLAUDE_PATH=claude        # Path to claude CLI (if not in PATH)
RELAY_DIR=~/.claude-relay # Working directory for temp files

# Optional - Features
SUPABASE_URL=             # For cloud memory persistence
SUPABASE_ANON_KEY=        # For cloud memory persistence
GEMINI_API_KEY=           # For voice transcription
ELEVENLABS_API_KEY=       # For voice responses
```

## FAQ

**Q: Why spawn CLI instead of using the API directly?**

The CLI gives you everything: tools, MCP servers, context management, permissions. The API is just the model. If you want the full Claude Code experience on mobile, you spawn the CLI.

**Q: Isn't spawning a process slow?**

It's ~1-2 seconds overhead. For a personal assistant, that's fine. If you need sub-second responses, use the Agent SDK instead.

**Q: Can I use this with other CLIs?**

Yes. The pattern works with any CLI that accepts prompts and returns text. Swap `claude` for your preferred tool.

**Q: How do I handle long-running tasks?**

Claude Code can take minutes for complex tasks. The relay handles this by streaming or waiting. Set appropriate timeouts.

**Q: What about MCP servers?**

They work. Claude CLI uses your `~/.claude/settings.json` config, so all your MCP servers are available.

## Security Notes

1. **Always verify user ID** - Never run an open bot
2. **Don't commit `.env`** - It's in `.gitignore`
3. **Limit permissions** - Consider `--permission-mode` flag
4. **Review commands** - Claude can execute bash, be aware of what you're allowing

## Credits

Built by [Goda](https://www.youtube.com/@godago) as part of the Personal AI Infrastructure project.

## License

MIT - Take it, customize it, make it yours.
