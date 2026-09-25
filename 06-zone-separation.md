# 6. Zone separation

**The process holding the credential must not be the process reasoning over the input, and
the reasoning process must not have network egress.**

## The decision

Consuming a mailbox looked like a connector problem - which client, which scopes. It is
not. **An inbox is the highest-quality untrusted input channel that exists:** any stranger,
unauthenticated, for free, at scale, can place arbitrary text into your agent context.
You never opted in the way you opt into visiting a web page.

Four public incidents, all this shape:

| Incident | Lesson |
|---|---|
| **ShadowLeak** (2025) - injection as white-on-white text in an email; the agent harvested inbox data and exfiltrated it with its *own* browser tool, server-side | The agent own tool was the leak. No endpoint DLP could see it. |
| **EchoLeak** - CVE-2025-32711, CVSS 9.3 - zero-click; malicious mail retrieved by RAG, exfiltrated via Markdown image auto-fetch through an allowlisted domain | Rendering agent output as Markdown or HTML is itself an egress channel. |
| **Gemini summary hijack** (2025) - zero-size white text made "summarise this" render an attacker phishing warning in the vendor own voice | Content filters do not catch pure text with no payload. |
| **postmark-mcp backdoor** (2025) - one added line BCC'd every processed message to the author; ~1,500 weekly downloads | An MCP server is code you are running, not a config entry. |

**Every one collapsed because the credential-holder and the reasoner were the same
process.**

## The zones

```
Zone 1  holds the credential, writes to a local store, has NO model and NO LLM client
Zone 2  reads that store, reasons, has NO credential and no access to the source
Zone 3  a human
```

## It is not about mail

Any connector pulling attacker-reachable content into a model has the same three
properties: shared inboxes, ticketing systems, chat, webhooks, public forms - and **search
results and RSS**, which is the one people miss. If your pipeline ingests anything a
stranger can publish, a write-capable agent over that data has all three legs of the
trifecta.

---

**Reference implementation:** [`docs/decisions/gmail-ingest.md`](https://github.com/gledach/signals/blob/main/docs/decisions/gmail-ingest.md)
