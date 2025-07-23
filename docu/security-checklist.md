# TimeBack Website – Security & Compliance Checklist (MVP)

_Scope: checklist applies to the public marketing site only; internal analytics and admin tools will have separate hardening docs._

*Last updated: 2025-07-20*

| Area | Requirement | MVP Action | Owner | Status |
|------|-------------|-----------|-------|--------|
| Transport | All endpoints over HTTPS | Deploy via Cloudflare cert | Infra | Pending |
| PII in Transit | Email hashed when sent to analytics | SHA-256 in client lib | FE | Pending |
| PII at Rest | Emails encrypted at DB layer | AES-256, RDS default | BE | Pending |
| Cookies | `tb_vid` flagged Secure; SameSite=Lax | Implement in `/session/start` | BE | Pending |
| Rate-Limiting | 5 req/min `/session/start` | Cloudflare WAF rule | Infra | Pending |
| DDoS | Universal WAF + bot detection | Cloudflare “Pro” tier | Infra | Pending |
| LLM Prompt Scrub | No minors’ PII allowed | Strip email before prompt | BE | Pending |
| Consent Banner | Cookie + analytics opt-out | Simple banner component | FE | Pending |
| Privacy Policy | GDPR + COPPA language | Draft page `/static/privacy.html` | Legal | Pending |
| Data Deletion | `/profile/delete` endpoint | Soft-delete row, purge events | BE | Future |
| Audit Logging | Access to PII tables logged | RDS IAM role audit | DevOps | Future |

**Notes**
- Session replay deferred until explicit consent UI.
- LocalStorage avoids PII; only caches personalization blocks.
- Further vendor security reviews (Groq, Together AI) to be completed before Aug 7. 

## 7. AI Safety Guardrails (Marketing Site)

| Area | Requirement | MVP Action | Owner | Status |
|------|-------------|-----------|-------|--------|
| Safe-completion | LLM must not output disallowed content (PII, self-harm, hate) | Enable provider `safe_completion` flag + app moderation pass | BE | Pending |
| Output Moderation | 2nd-pass model or regex fallback within 50 ms | Integrate OpenAI Moderation API; fallback regex | BE | Pending |
| Red-Team Testing | Monthly automated jailbreak sweep | Run Garak baseline corpus in CI | SecOps | Pending |
| Incident Response | <2 h mitigation, 24 h comms | PagerDuty rotation + flag `agent_stream_enabled=false` | SecOps | Pending |
| Audit Log | Store blocked outputs hashed & encrypted 2 y | `blocked_output_log` table | DevOps | Pending |

### Red-Team Playbook
1. Run Garak with jailbreak, prompt-injection suites.  
2. Evaluate moderation scores; file GH issue for >0.5 severity.  
3. Fix & retest before merging model upgrades.

--- 