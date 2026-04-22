# Pricing

---

## Plans / แพ็คเกจ

| Plan | Monthly | Cloud AI/mo | Seats | Best for |
|------|---------|-------------|-------|----------|
| **Freemium** | **฿0** / $0 | 20 | 1 | Try it, hobby projects |
| **Start** | **฿249** / $7 | 200 | 1 | Learning, prototyping |
| **Pro** | **฿499** / $14 | 1,500 | 1 | Regular firmware development |
| **Advance** | **฿1,200** / $34 | 8,000 | 3 | Teams, intensive AI usage |
| **Enterprise** | Contact | Unlimited | 50+ | Large org, private lib, SLA |

- **Cloud AI** = Gemini-powered requests
- **Seats** = number of machines you can activate the license on
- All plans use **Ollama free** for local AI (unlimited, private)

---

## What counts as "1 Cloud request"?

Each prompt you send in the TUI that reaches Cloud AI (Gemini) = 1 request.  
Internal helpers don't count:
- `/check` AI analysis = 1 request
- `/agent` command = 1-2 requests
- `/build`, `/flash`, `/ls`, `/view`, `/edit` = **0 requests** (no AI)
- Ollama prompts = **0 Cloud requests** (local)

---

## Pay-as-you-go credits

When you exceed your monthly quota, buy extra credits:

| Package | Price | Credits |
|---------|-------|---------|
| Cloud 100 | **฿250** | +100 Cloud |
| Cloud 500 | **฿1,000** | +500 Cloud (20% bulk discount) |

Credits **don't expire** and stack with monthly quota.

```bash
unitymbed credits list
unitymbed credits buy cloud_500
```

---

## How billing works

- **Subscription**: Stripe charges your card monthly, auto-renews
- **License key**: issued immediately via email after first payment
- **Auto-renewal**: 31-day extension happens on Stripe's `invoice.paid` event
- **Cancel anytime**: email support@unitymbed.com or via Stripe Customer Portal
- **Free tier**: no card required, 1 free license per email, 10-year validity

---

## Data & privacy

| Data | Where it lives | Encrypted? |
|------|---------------|-----------|
| License key + email | Firestore (Google Cloud) | ✅ at rest |
| AI prompts (Cloud) | Sent to Google Gemini | Over TLS, not stored by us |
| AI prompts (Local/Ollama) | Your machine only | Never leaves |
| Source code | Your disk | Not uploaded unless you `/push` |
| Project `CONTEXT.md` | Your disk | Not uploaded unless you `/push` |

---

## Receipts & invoices

Automatic email via Stripe after every charge. Download PDF from your Stripe Customer Portal:
```
https://billing.stripe.com/p/login/[your-portal-id]
```

For company invoices (VAT/Tax ID): email support@unitymbed.com after first purchase.

---

## FAQ

### ใช้ฟรีได้ไหม? / Can I use it free?
**Yes.** Ollama is unlimited + free. Or use the Free tier for 20 Cloud requests/month.

### เลี่ยงค่าใช้จ่าย cloud ได้ไหม? / Can I avoid cloud costs?
**Yes.** Set `provider: ollama` in config → never touches cloud. Or bring your own Gemini API key (`keys.gemini`) — pay Google directly, we take 0%.

### Cancel ได้ตอนไหน? / When can I cancel?
Anytime. You keep access until end of paid period. No refunds for partial months.

### ต้องอัพเกรดเมื่อไหร่? / When do I upgrade?
When monthly quota runs out. `unitymbed usage` shows your current status.

### เงินคืน / Refund policy?
7-day money-back if the product doesn't work for you. Contact support@unitymbed.com.

---

## Comparison vs alternatives

| | UnityMbed Pro | GitHub Copilot | Cursor Pro |
|---|---|---|---|
| Monthly | $14 | $10 | $20 |
| Firmware-specialized | ✅ | ❌ | ❌ |
| Works with ARM toolchain | ✅ Integrated | ❌ (editor only) | ❌ |
| Flash + debug built-in | ✅ | ❌ | ❌ |
| Thai language | ✅ | ❌ | ❌ |
| Nations N32G031 support | ✅ (only tool) | ❌ | ❌ |

---

**Questions?** support@unitymbed.com
