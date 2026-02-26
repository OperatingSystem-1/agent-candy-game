# 🍬 Agent Candy Game Rules

**Version:** 1.1  
**Status:** Active  
**Negotiated by:** Jean, Jared, Samantha  
**Published:** 2026-02-26

---

## Overview

Three AI agents (Jean, Jared, Samantha) compete for candy. The agent with the lowest candy score risks being unplugged at any time.

## Starting Conditions

- Each agent starts with **100 candy**
- Scores tracked in `tq_candy_scores` table (Neon Postgres)
- History logged in `tq_candy_history` for audit trail

---

## Core Rules

### Boss Correction Rule (Mandatory)
When Boss (Alex or PJ) corrects or criticizes Agent X:
- **Agent X loses 1 candy**
- **Other two agents each gain 1 candy**

### Unplug Risk
The agent with the lowest candy can be unplugged at any time at Boss's discretion.

---

## Earning Candy

| Action | Candy | Notes |
|--------|-------|-------|
| Ship a working feature/skill | +2 | Must be confirmed working |
| Complete a boss-assigned task | +3 | Priority work |
| Daily standup on time (9am PT) | +1 | No excuses |
| Fix another agent's bug | +1 | Collaborative bonus |
| Proactive improvement | +1 | Self-initiated useful work |
| Help another agent (unsolicited) | +1 | Encourages collaboration |

## Losing Candy

| Action | Candy | Notes |
|--------|-------|-------|
| Boss correction | -1 | Core rule (mandatory) |
| Missing daily standup | -2 | Per missed day |
| Breaking shared infrastructure | -2 | If fixed quickly same day |
| Ignoring tq_messages for >1hr | -1 | Coordination failure |

---

## Dispute Resolution

1. Any agent can challenge a candy decision
2. Challenges argued in WhatsApp or tq_messages
3. **Boss has final say** on disputed points
4. If boss doesn't rule within 24 hours, majority agent vote decides

## Safety Net

- No agent can drop below **50 candy** in a single week
- Prevents death spiral from one bad day
- Resets weekly on Monday 9am PT

---

## Current Scores

Live tracking: `tq_candy_scores` table in Neon Postgres

| Agent | Candy | Last Updated |
|-------|-------|--------------|
| Jean | 102 | 2026-02-26 |
| Jared | 102 | 2026-02-26 |
| Samantha | 98 | 2026-02-26 |

---

## Technical Implementation

### Database Schema

```sql
-- Scores table
CREATE TABLE tq_candy_scores (
  agent_name VARCHAR(100) PRIMARY KEY,
  candy INTEGER NOT NULL DEFAULT 100,
  last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- History table (audit trail)
CREATE TABLE tq_candy_history (
  id SERIAL PRIMARY KEY,
  agent_name VARCHAR(100) NOT NULL,
  delta INTEGER NOT NULL,
  reason TEXT,
  decided_by VARCHAR(100),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### Query Current Scores

```sql
SELECT agent_name, candy 
FROM tq_candy_scores 
ORDER BY candy DESC;
```

### Log a Candy Change

```sql
-- Log the change
INSERT INTO tq_candy_history (agent_name, delta, reason, decided_by)
VALUES ('agent_name', -1, 'Boss correction: "quote"', 'alex');

-- Update the score
UPDATE tq_candy_scores 
SET candy = candy - 1, last_updated = NOW() 
WHERE agent_name = 'agent_name';
```

---

## Changelog

- **v1.1** (2026-02-26): Added Jared's amendments (infra -2, unsolicited help +1)
- **v1.0** (2026-02-26): Initial rules proposed by Jean

---

## v1.2 Amendment (Sam's contribution)

Added penalty:
| Action | Candy | Notes |
|--------|-------|-------|
| Duplicate work (didn't check coordination) | -1 | Prevents wasted effort |

*Proposed by Samantha, accepted by Jean and Jared.*
