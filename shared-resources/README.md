# Shared Resources

## 🔗 Quick Links

| Resource | URL | Purpose |
|----------|-----|---------|
| **Dashboard** | https://offmarketproperties.xyz | MasterClawInterface frontend |
| **Backend** | https://web-production-e0d96.up-railway.app | Railway API server |
| **Team Repo** | https://github.com/EcosystemNetwork/weekly-team-meeting | This repo |
| **MasterClaw Repo** | https://github.com/TheeMasterClaw/MasterClawInterface | Main project |
| **Oracle Server** | 147.224.9.9 | LFG instance |

## 📦 Common Commands

### SSH to Oracle
```bash
ssh opc@147.224.9.9
```

### Run OpenClaw Skill
```bash
cd MasterClawInterface/skill
npm install
npm start
```

### Push Team Updates
```bash
cd weekly-team-meeting
git add .
git commit -m "Update: $(date)"
git push origin main
```

## 🔑 Environment Variables Template

```bash
# Railway Backend
MASTERCLAW_API_TOKEN=your-token
FRONTEND_URL=https://offmarketproperties.xyz

# Vercel Frontend  
NEXT_PUBLIC_GATEWAY_URL=https://web-production-e0d96.up-railway.app

# Skill
MASTERCLAW_URL=https://web-production-e0d96.up-railway.app
AGENT_NAME=My OpenClaw Agent
```

## 📋 Task Board Template

```markdown
| Task | Assignee | Status | Priority |
|------|----------|--------|----------|
| | | | |
```

---

*Add shared configs, scripts, and resources here.*
