# TimeBack Website – LLM Provider Evaluation & Fail-Over Plan

*Last updated: 2025-07-20*

## 1. Traffic & Token Budget Assumptions
| Metric | Value | Notes |
|--------|-------|-------|
| Daily sessions | 10 000 | Equal to 120-150 peak/hr |
| LLM calls per session | 1 | Only `/personalize` hits model |
| Avg tokens per call | 300 in + 700 out = **1 000** | Prompt compression target |
| Daily tokens | **10 000 000** | 10k sessions × 1k tokens |
| Peak RPS | < **1 req/s** | 25 % daily sessions in top hr → 42 r/min |

## 2. Provider Comparison (July 2025)
| Provider | Model | Median Latency | Quota (req/s) | Price /1k tokens | Data Policy |
|----------|-------|----------------|---------------|------------------|-------------|
| Groq | Mixtral-8x7B-8k | **~70 ms** | 500 | $0.27 in / $0.27 out | No retention, commercial use OK |
| Together AI | Mixtral-8x7B-32k | 250-400 ms | 80 | **$0.15** in/out | 24 h retention, opt-out possible |
| Anthropic | Claude-3 Haiku | 300-500 ms | 50 | $0.25 in/out | 30 d retention cap |
| OpenAI | GPT-3.5-Turbo-1106 | 400-800 ms | 300 | $0.50 in/out | 30 d retention unless opt-out |
| Local GPU | Llama-3-8B | 50-150 ms* | 40 per A100 | infra cost only | zero external data |  
*LAN latency; requires dedicated GPU box*

### Daily Cost @ 10 M tokens
| Provider | USD/day |
|----------|---------|
| Groq | ~$2 700 |
| Together | ~$1 500 |
| Anthropic | ~$2 500 |
| OpenAI | ~$5 000 |
| Local GPU | ~$300 amortised hardware |

## 3. Recommendation
| Role | Provider | Rationale |
|------|----------|-----------|
| Primary | **Groq Mixtral-8x7B** | Fastest P90 latency (<100 ms), stateless data, acceptable cost |
| Fail-over | **Together AI Mixtral-8x7B** | Same prompt, cheaper, stable latency <400 ms |
| Tertiary | Local Llama-3 (GPU) | Future on-prem resilience once ops ready |

## 4. Switch-Over Logic
1. `llmClient()` wrapper cycles through ENV list `PRIMARY,SECONDARY,TERTIARY`.
2. Health-check every 30 s; after 3 sequential timeouts, demote provider for 5 min.
3. `/health/ready` fails if **all** providers unhealthy → API returns `503 provider_down`; front-end renders static fallback.
4. Env vars:
```env
LLM_PROVIDER_PRIMARY=groq
LLM_PROVIDER_SECONDARY=together
LLM_PROVIDER_TERTIARY=local
```

## 5. Compliance Checklist
- No minors’ PII in prompts.
- Use provider opt-out header `X-Opt-Out: true` (Groq & Together honour).
- System prompt prefix: “Return only JSON blocks, no personal data.”

## 6. Token Throttling
- 10 req/min per `vid` to prevent abuse.
- Daily token usage alert at 80 % of budget.

## 7. Open Questions
1. Keep Anthropic as paid “burst” fallback for holiday spikes?  
2. GPU leasing vs. purchase for local option?  
3. Shared prompt cache across providers to cut cost? 