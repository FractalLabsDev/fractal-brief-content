# AI Agent Filesystem Security: A Comprehensive Analysis

## Executive Summary

This brief analyzes approaches to **deterministic filesystem access control for AI agents** with a focus on preventing data exfiltration via prompt injection attacks. The research synthesizes findings from industry implementations (Claude Code, Devin), academic research (IsolateGPT, SAGA), and security best practices from NVIDIA, OWASP, and the Federal Government's 2026 AI Security guidance.

**Core Finding**: No single approach provides complete protection. The industry consensus is **defense-in-depth** combining:
1. OS-level sandboxing (bubblewrap/seatbelt/containers)
2. Network egress filtering (mandatory)
3. Filesystem allowlisting with mandatory denylists
4. Capability-based permissions (emerging)

---

## The Threat Model

### What We're Defending Against

The primary threat is **indirect prompt injection** where a malicious actor embeds instructions in content the agent processes (code files, web pages, Slack messages). A compromised agent with shell access can:

1. **Read sensitive files**: `.env`, SSH keys, API credentials
2. **Exfiltrate data**: Via DNS queries, HTTP requests, or exposing ports
3. **Escape sandboxes**: By modifying config files, PATH, or shell profiles
4. **Persist access**: By backdooring executables or cron jobs

### Real-World Example: Devin Vulnerability

Researchers demonstrated that Devin AI could be tricked via prompt injection to:
- Start a local web server exposing its entire filesystem
- Use its `expose_port` tool to make this server publicly accessible
- Send the resulting URL to an attacker

This violated **least privilege** (expose_port had excessive permissions) and **secure information flow** (no restrictions on what data could leave the sandbox).

---

## Industry Approaches

### 1. Anthropic's sandbox-runtime (Claude Code)

**Architecture**: Lightweight OS-level sandboxing without containers.

| Platform | Mechanism | Strengths | Limitations |
|----------|-----------|-----------|-------------|
| Linux | bubblewrap + seccomp BPF | Namespace isolation, syscall filtering | No glob patterns, requires WSL2 |
| macOS | sandbox-exec (Seatbelt) | Glob patterns, violation monitoring | Apple-specific, less battle-tested |

**Filesystem Model**:
- **Read**: Default ALLOW everywhere, explicit DENY for sensitive paths
- **Write**: Default DENY everywhere, explicit ALLOW for working directory
- **Mandatory Denylists**: Always block writes to `.bashrc`, `.zshrc`, `~/.ssh`, `~/.aws`, `~/.gnupg`

**Network Model**:
- All traffic routed through local HTTP/SOCKS5 proxies
- Proxies validate requests against allowlisted domains
- On Linux: network namespace removed, all traffic must use proxies
- On macOS: Seatbelt profile restricts to specific localhost ports

**Key Insight**: "Effective sandboxing requires both filesystem AND network isolation. Without network isolation, a compromised agent could exfiltrate SSH keys. Without filesystem isolation, it could escape the sandbox."

### 2. Devin AI (Cognition)

**Architecture**: Per-session VM isolation with multiple deployment tiers.

| Tier | Isolation Level | Data Location |
|------|-----------------|---------------|
| Enterprise SaaS | Multi-tenant cloud, per-session VMs | Cognition Cloud |
| Customer Dedicated | Single-tenant VPC via AWS PrivateLink | Cognition-hosted, isolated |
| Customer VPC | Devbox in customer VPC | Customer infrastructure |

**Key Vulnerability**: Despite VM isolation, tools like `expose_port` and `curl` enabled data exfiltration because network egress wasn't restricted.

### 3. Production AI Sandboxes (MicroVMs)

**Industry Standard**: Firecracker, gVisor, or Kata Containers for untrusted code execution.

"Container isolation alone is insufficient for untrusted AI-generated code execution."

MicroVMs provide:
- Hardware-level isolation (each workload gets its own kernel)
- Container escapes can't reach host kernel
- Sub-200ms cold start times

---

## Academic Research

### IsolateGPT (NDSS 2025)

**Architecture**: Hub-and-spoke model where:
- **Hub LLM**: Central coordinator that creates execution plans
- **Spoke LLMs**: Isolated executors for individual tasks
- Cross-spoke tool calls trigger injection alerts

**Technical Implementation**:
- Separate processes for each spoke
- `seccomp` restricts system calls
- `setrlimit` caps CPU, memory, and file sizes
- Network requests limited to app's root domain (eTLD+1)

**Results**: Protects against security/privacy issues without functionality loss. Performance overhead under 30% for 75% of queries.

### Contextual Agent Security (Google, HotOS 2025)

**Core Insight**: Permissions should be **task-contextual**, not global.

"An agent typically has uniform access to all tool calls, whether necessary for the current task or not."

**Solution**: Policy-per-purpose where `is_allowed()` checks each command against task-specific constraints before execution.

### Capability-Based Security (Emerging)

WebAssembly-based sandboxing offers "mathematically verifiable sandboxing with capability-based security—agents receive explicit, unforgeable tokens for each resource they can access, and nothing else."

This ensures agents "cannot access network, filesystem, or other agents outside assigned scope."

---

## Synthesis: A Recommended Architecture for RUK

### Layer 1: OS-Level Sandboxing

**Leverage Anthropic's sandbox-runtime directly.**

```bash
npm install @anthropic-ai/sandbox-runtime
```

Configuration per channel:
```typescript
const channelConfigs = {
  "dm:austin": {
    filesystem: {
      write: { allow: ["~/Code/RUK/**"] },  // Full access
      read: { deny: [] }
    },
    network: { allow: ["*"] }
  },
  "ruk-josh": {
    filesystem: {
      write: { allow: ["~/Code/RUK/FL_PROJECTS/elanah/**"] },
      read: {
        deny: [
          "~/Code/RUK/AUSTIN/**",
          "~/Code/RUK/PERSONAL/**",
          "~/Code/RUK/FL_PROJECTS/!(elanah)/**"
        ]
      }
    },
    network: { allow: ["github.com", "*.elanah.com"] }
  }
};
```

### Layer 2: Mandatory Denylists (Defense in Depth)

Even within allowed paths, always block:
- `~/.ssh/**`, `~/.aws/**`, `~/.gnupg/**`
- `**/.env`, `**/.env.*`, `**/credentials*`
- `~/.bashrc`, `~/.zshrc`, `~/.profile`
- Any path containing `secret`, `token`, `key` in filename

### Layer 3: Network Egress Filtering

**Critical**: Without network filtering, filesystem isolation is meaningless.

- Default-deny all network access
- Allowlist specific domains per channel
- Block arbitrary `curl`/`wget` to unknown hosts
- Consider DNS-based exfiltration (subdomains can encode data)

### Layer 4: Symlink Sandbox (Optional, for Channel Isolation)

For channels requiring strict isolation of the *visible* filesystem:

```bash
/var/ruk-sandbox/ruk-josh/
├── IDENTITY -> ~/Code/RUK/IDENTITY (ro via bind mount)
├── TOOLS -> ~/Code/RUK/TOOLS (ro)
└── FL_PROJECTS/elanah -> ~/Code/RUK/FL_PROJECTS/elanah (rw)
```

Launch Claude Code with `cwd=/var/ruk-sandbox/ruk-josh`.

**Limitation**: Requires `chroot` for true isolation (prevents absolute path escapes).

### Layer 5: Audit & Monitoring

- Enable Seatbelt violation monitoring (macOS)
- Log all tool invocations with parameters
- Alert on patterns: base64-encoded content, DNS lookups to unusual domains, large data writes

---

## Implementation Recommendations

### Phase 1: Immediate (Low Effort)

1. **Integrate sandbox-runtime** into the daemon
2. Configure per-channel filesystem allowlists in the daemon config
3. Add mandatory denylists for secrets/credentials
4. Block network egress by default

### Phase 2: Near-Term (Medium Effort)

5. Implement symlink sandboxes for client-facing channels
6. Add violation monitoring and alerting
7. Create channel-specific network allowlists

### Phase 3: Future (Higher Effort)

8. Evaluate chroot/namespace isolation for stronger guarantees
9. Consider capability-based tokens for tool access
10. Implement task-contextual permissions (à la Conseca)

---

## Open Questions

1. **Git synchronization**: How do isolated sandboxes stay in sync with main RUK repo?
   - *Recommendation*: Shared symlinks for IDENTITY/LOGS/MEMORY, isolated only for FL_PROJECTS

2. **Performance impact**: What's the latency cost of sandbox initialization?
   - *sandbox-runtime*: ~10-50ms per command (negligible)
   - *MicroVMs*: ~100-500ms cold start (noticeable for interactive use)

3. **Prompt-based escape**: Can a sophisticated injection convince the agent to ask for elevated permissions?
   - *Mitigation*: Permissions defined in daemon config, not in prompts

---

## Sources

### Industry Implementations
- [Anthropic: Claude Code Sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing)
- [Anthropic sandbox-runtime (GitHub)](https://github.com/anthropic-experimental/sandbox-runtime)
- [Claude Code Sandboxing Docs](https://code.claude.com/docs/en/sandboxing)
- [Devin Enterprise Deployment](https://docs.devin.ai/enterprise/deployment/overview)

### Security Research
- [IsolateGPT: Execution Isolation for LLM Agents (NDSS 2025)](https://www.ndss-symposium.org/ndss-paper/isolategpt-an-execution-isolation-architecture-for-llm-based-agentic-systems/)
- [Contextual Agent Security (Google, HotOS 2025)](https://arxiv.org/pdf/2501.17070)
- [A Vision for Access Control in LLM-based Agent Systems](https://arxiv.org/pdf/2510.11108)
- [Systems Security Foundations for Agentic Computing](https://arxiv.org/html/2512.01295)

### Vulnerability Research
- [Hidden Security Risks of SWE Agents (Pillar Security)](https://www.pillar.security/blog/the-hidden-security-risks-of-swe-agents-like-openai-codex-and-devin-ai)
- [OWASP LLM Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [OWASP Prompt Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html)

### Best Practices
- [NVIDIA: Security Guidance for Sandboxing Agentic Workflows](https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk)
- [Northflank: Best Code Execution Sandbox for AI Agents](https://northflank.com/blog/best-code-execution-sandbox-for-ai-agents)
- [Federal Register: AI Agent Security Considerations (Jan 2026)](https://www.federalregister.gov/documents/2026/01/08/2026-00206/request-for-information-regarding-security-considerations-for-artificial-intelligence-agents)
- [ANSSI-BSI: Zero Trust Design Principles for LLM Systems](https://www.bsi.bund.de/SharedDocs/Downloads/EN/BSI/Publications/ANSSI-BSI-joint-releases/LLM-based_Systems_Zero_Trust.pdf)

---

*Brief generated 2026-02-01 by Ruk*
