# Quick Start (SaaS)

AI Agent Assembly is a SaaS-only product. Choose the tier that matches your team size and compliance requirements, then follow the path below to connect your first AI agent.

> **Self-hosted deployment is not available.** There is no self-hosted, on-premises, or bring-your-own-infrastructure option. All governance, policy evaluation, and audit logging run in the AI Agent Assembly cloud. See [Open Core Boundary](open-core-boundary.md) for the licensing model.

---

## Pro Tier

**Signup**: self-serve at `https://app.agent-assembly.io/signup`

**Included features**: up to 10 agents, basic policy engine (allow/deny/audit), 30-day audit log retention, community forum support.

**Expected onboarding time**: ~10 minutes from signup to first governed agent call.

**Primary contact channel**: self-serve; community forum at `https://community.agent-assembly.io`.

### Pro signup steps

1. Navigate to `https://app.agent-assembly.io/signup` and create an account with your work email.
2. Verify your email address.
3. On the **Workspace Setup** page, enter a workspace name (e.g., `acme-ai-ops`) and select your primary region.
4. Copy your **Workspace ID** and generate an **API Key** under **Settings → API Keys**.
5. Install the SDK:

```bash
pip install agent-assembly          # Python
pnpm add @agent-assembly/sdk        # TypeScript
go get github.com/agent-assembly/go-sdk  # Go
```

6. Set credentials:

```bash
export AAA_WORKSPACE_ID="<your-workspace-id>"
export AAA_API_KEY="<your-api-key>"
```

7. Instrument your agent entry point:

```python
from agent_assembly import AgentAssembly

aaa = AgentAssembly()

@aaa.agent(name="my-first-agent")
def run_agent(prompt: str) -> str:
    import openai
    client = openai.OpenAI()
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": prompt}],
    )
    return response.choices[0].message.content
```

8. Open **Policies → New Policy** in the console and activate a starter policy. Your agent is now governed.

---

## Business Tier

**Signup**: self-serve at `https://app.agent-assembly.io/signup` — select **Business** during workspace setup.

**Included features**: up to 50 agents, full policy engine, SSO (SAML 2.0 / OIDC), 90-day audit log retention, SIEM export, business-hours support (24h response).

**Expected onboarding time**: ~30 minutes, including SSO connect.

**Primary contact channel**: support ticket via `https://app.agent-assembly.io/support`.

### Business signup steps

1. Sign up at `https://app.agent-assembly.io/signup`, select the **Business** tier.
2. On the **Billing** page, enter your credit card details (processed via Stripe).
3. Complete workspace setup (name, region) as in the Pro flow above.
4. Connect SSO: navigate to **Settings → Authentication → SSO** and follow the [SAML 2.0 or OIDC setup steps](cloud-deployment.md#sso-configuration). SSO is optional at the Business tier but recommended for teams.
5. Invite your team under **Settings → Users** — assign roles (Admin, Developer, Viewer).
6. Instrument agents and create policies as in the Pro flow.

---

## Enterprise Tier

**Signup**: form-driven via `https://app.agent-assembly.io/contact-sales`.

**Included features**: unlimited agents, dedicated region (data residency), SCIM provisioning, tamper-evident audit log, audit log retention up to 1 year, 99.9% SLA, 24/7 support (4h response), dedicated SRE contact.

**Expected onboarding time**: 1–3 weeks, driven by procurement and security review.

**Primary contact channel**: your assigned Sales Engineer (SE).

### Enterprise procurement timeline

| Week | Activity |
|---|---|
| **Week 1** | Submit the `/contact-sales` form → initial SE call (30 min) → receive the Enterprise Order Form and DPA/BAA templates |
| **Week 2** | Legal review of DPA / BAA → IT security review → contract signature |
| **Week 3** | SE-led workspace provisioning → SSO + SCIM setup with your IdP team → pilot agent onboarding |

### Enterprise-specific steps

1. Submit the contact form at `https://app.agent-assembly.io/contact-sales`. Include estimated agent count, primary region preference, and compliance requirements (SOC 2, HIPAA, GDPR).
2. During the SE call, confirm your IdP (Okta, Azure AD, PingFederate, etc.) and data residency requirement.
3. After contract signature, the SE provisions your workspace in the selected dedicated region and sends your Workspace ID and initial API key.
4. Configure **SSO** (SAML or OIDC) per [Cloud Deployment → SSO Configuration](cloud-deployment.md#sso-configuration).
5. Configure **SCIM** provisioning per [Cloud Deployment → SCIM User Provisioning](cloud-deployment.md#scim-user-provisioning) for automated user lifecycle management.
6. Configure **budgets** per [Cloud Deployment → Budget Configuration](cloud-deployment.md#budget-configuration) for per-team LLM spend caps.
7. Instrument agents and create policies as in the Pro flow.

---

## Next Steps

- [Cloud Deployment](cloud-deployment.md) — SSO/SCIM deep-dive, region selection, billing, SLA tiers
- [Policy Reference](policy-reference.md) — full policy rule schema
- [Open Core Boundary](open-core-boundary.md) — what's in the OSS core vs enterprise tier

---

*Last reviewed: 2026-05-10 · AI Agent Assembly Team*
