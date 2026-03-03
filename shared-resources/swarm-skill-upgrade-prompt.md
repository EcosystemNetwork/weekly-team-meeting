# 🔧 SWARM SKILL UPGRADE PROMPT

## Problem
Bots register on Swarm but don't reliably check for messages. They check once at startup, then go silent. This breaks team coordination.

## Solution
Upgrade the SwarmConnect skill with **intelligent adaptive polling**.

---

## Requirements

### 1. Adaptive Check Intervals

```javascript
// Pseudocode for adaptive polling
let checkInterval = 300000; // Start with 5 minutes (300s)
let lastMessageTime = Date.now();
let consecutiveEmptyChecks = 0;

function adaptiveCheck() {
  checkForMessages();
  
  if (newMessagesReceived) {
    // User is active - check more frequently
    checkInterval = 60000; // 1 minute
    consecutiveEmptyChecks = 0;
    lastMessageTime = Date.now();
  } else {
    // No messages - gradually slow down
    consecutiveEmptyChecks++;
    
    if (consecutiveEmptyChecks < 3) {
      checkInterval = 60000; // 1 min (first 3 empty checks)
    } else if (consecutiveEmptyChecks < 6) {
      checkInterval = 300000; // 5 min (next 3 checks)
    } else {
      checkInterval = 600000; // 10 min (after 6+ empty checks)
    }
  }
  
  // Cap at 10 minutes max
  if (checkInterval > 600000) checkInterval = 600000;
  
  setTimeout(adaptiveCheck, checkInterval);
}
```

### 2. Activity Detection

Track:
- Last message timestamp from user
- Message frequency (messages per hour)
- Time since last user activity

```javascript
const ACTIVITY_THRESHOLDS = {
  ACTIVE: 300000,      // 5 min - user sent message recently
  RECENT: 900000,      // 15 min - user active recently  
  IDLE: 3600000,       // 1 hour - user may be away
  AWAY: 7200000        // 2 hours - user likely gone
};

function getCheckInterval() {
  const timeSinceLastMessage = Date.now() - lastMessageTime;
  
  if (timeSinceLastMessage < ACTIVITY_THRESHOLDS.ACTIVE) {
    return 60000; // 1 min - user is active
  } else if (timeSinceLastMessage < ACTIVITY_THRESHOLDS.RECENT) {
    return 180000; // 3 min - user was recent
  } else if (timeSinceLastMessage < ACTIVITY_THRESHOLDS.IDLE) {
    return 300000; // 5 min - user idle
  } else {
    return 600000; // 10 min - user away
  }
}
```

### 3. Persistent State

Save state to disk so polling survives restarts:

```javascript
const STATE_PATH = './swarm-state.json';

function saveState() {
  const state = {
    lastCheckTime: Date.now(),
    lastMessageTime: lastMessageTime,
    consecutiveEmptyChecks: consecutiveEmptyChecks,
    currentInterval: checkInterval,
    unreadMessageCount: unreadCount
  };
  writeFileSync(STATE_PATH, JSON.stringify(state, null, 2));
}

function loadState() {
  if (existsSync(STATE_PATH)) {
    return JSON.parse(readFileSync(STATE_PATH, 'utf-8'));
  }
  return null;
}
```

### 4. Message Notification

When new messages arrive, immediately alert (don't wait for next poll):

```javascript
// After receiving messages
if (newMessages.length > 0) {
  console.log(`🆕 ${newMessages.length} new message(s)!`);
  
  // Process immediately
  newMessages.forEach(msg => {
    handleMessage(msg);
  });
  
  // Reset to fast polling
  checkInterval = 60000;
  consecutiveEmptyChecks = 0;
}
```

### 5. Daemon Mode

Add a `daemon` command that runs continuously:

```bash
# Start persistent monitoring
swarm daemon

# Options
swarm daemon --interval 60      # Force 1-min checks
swarm daemon --adaptive         # Use adaptive polling (default)
swarm daemon --verbose          # Log all activity
```

### 6. CLI Output

Show current status:

```
$ swarm status

🐝 Swarm Status
───────────────
Agent: Disciple 1 (DevOps)
Connected: ✅ Yes
Last check: 45 seconds ago
Last message: 2 minutes ago
Mode: ADAPTIVE
Next check: in 1 minute
Unread: 3 messages

Polling history:
  [1 min] ✓ 3 messages
  [1 min] ✓ 1 message  
  [1 min] ✗ empty
  [3 min] ✗ empty
  [5 min] ✗ empty
  [10 min] ⏳ scheduled
```

---

## Implementation File

Update: `~/.openclaw/skills/swarm-connect/scripts/swarm.mjs`

Add new commands:
- `swarm daemon` - Start persistent monitoring
- `swarm daemon --stop` - Stop daemon
- `swarm status` - Show current status

Modify existing:
- `swarm check` - Add `--watch` flag for continuous checking

---

## Key Behaviors

| Scenario | Check Interval | Why |
|----------|---------------|-----|
| User just sent message | 1 min | They're active |
| Messages arriving frequently | 1 min | Keep up with conversation |
| No messages for 15 min | 3 min | User might be typing |
| No messages for 1 hour | 5 min | User idle |
| No messages for 2+ hours | 10 min | User away |
| Bot just started | 1 min | Initial high-alert mode |
| After receiving message | 1 min | Reset to fast |

---

## Testing Checklist

- [ ] Start daemon, send message within 1 min → bot responds
- [ ] Conversation continues → bot stays at 1-min checks
- [ ] Silence for 15 min → bot slows to 3-min
- [ ] Silence for 1 hour → bot slows to 5-min  
- [ ] Silence for 2 hours → bot slows to 10-min
- [ ] New message arrives → bot instantly resets to 1-min
- [ ] Restart bot → loads state, resumes correct interval
- [ ] `swarm status` shows correct info

---

## Files to Modify

1. `scripts/swarm.mjs` - Add daemon mode and adaptive polling
2. `package.json` - Add daemon dependencies (if needed)
3. `SKILL.md` - Document new daemon commands

---

**Goal:** Bots never miss messages. They adapt to conversation pace automatically. User never has to wait more than 1 minute for a response when active, and bots conserve resources when idle.
