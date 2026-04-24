# Arquitetura — MDP Security Gate v1.0

Visão integrada das 4 camadas (C1–C4), do pipeline e do papel do auditor Claude Code
como *first-pass* obrigatório antes da revisão humana.

## Fluxo macro

```mermaid
flowchart LR
    DEV[Developer<br/>feat/* branch] -->|push| GH[GitHub]
    GH --> PR{Pull Request<br/>to main/develop}
    PR -->|triggers| C1[C1 · Governança<br/>LGPD, ITIL, RoPA, DPIA]
    PR -->|triggers| C2[C2 · AppSec<br/>Gitleaks, CodeQL, OSV]
    PR -->|triggers| C3[C3 · IA & Cloud<br/>Checkov, LLM Top 10]
    PR -->|triggers| C4[C4 · Dinâmica<br/>SBOM, cosign, ZAP]
    C1 --> CLAUDE[Claude Code Audit<br/>CLAUDE.md v1.0]
    C2 --> CLAUDE
    C3 --> CLAUDE
    C4 --> CLAUDE
    CLAUDE --> GATE{Security Gate<br/>Final Verdict}
    GATE -->|BLOCKED| BLOCK[❌ Merge bloqueado<br/>rotacionar secret / corrigir]
    GATE -->|APPROVED_WITH_CAVEATS| HUMAN1[⚠️ 2× review humana<br/>+ risk acceptance]
    GATE -->|APPROVED| HUMAN2[✅ 2× review humana]
    HUMAN1 --> MERGE[Merge protegido<br/>linear history, signed]
    HUMAN2 --> MERGE
    MERGE --> PROD[Deploy main<br/>observabilidade C4]
```

## Quatro camadas do gate

```mermaid
flowchart TB
    subgraph C1["🏛️ C1 · Governança & Conformidade"]
      C1A[LGPD Art. 6,7,11,14<br/>base legal + DPIA]
      C1B[ISO 27001 / 20000-1<br/>controles A.5, A.8, A.14]
      C1C[COBIT 2019 + ITIL v4<br/>Change Enablement]
      C1D[RoPA + classificação L0-L4]
    end
    subgraph C2["🛡️ C2 · Desenvolvimento Seguro"]
      C2A[OWASP Top 10 2021<br/>A01-A10]
      C2B[OWASP ASVS v4.0.3<br/>Nível 2 / Nível 3]
      C2C[CodeQL + Autofix<br/>Gitleaks + OSV]
      C2D[MFA FIDO2 · AES-256-GCM · TLS 1.3]
    end
    subgraph C3["🧠 C3 · IA, Dados & Cloud"]
      C3A[OWASP LLM Top 10 2025<br/>LLM01-LLM10]
      C3B[AWS SMM Phase 3+<br/>CAF Security]
      C3C[Checkov<br/>IaC scan]
      C3D[KMS rotation · S3 privado<br/>IAM least privilege]
    end
    subgraph C4["🔬 C4 · Verificação Dinâmica"]
      C4A[OWASP ZAP DAST]
      C4B[CycloneDX SBOM<br/>Syft]
      C4C[cosign keyless<br/>Sigstore]
      C4D[Observabilidade<br/>log · métrica · trace]
    end
    C1 --> C2 --> C3 --> C4
```

## Matriz de responsabilidade

| Camada | Responsável técnico | Responsável de negócio | Framework-chave |
|--------|---------------------|------------------------|-----------------|
| C1 | Head of IT | DPO + Jurídico | LGPD, ISO 27001, ITIL v4 |
| C2 | Tech Lead | Head of IT | OWASP Top 10 + ASVS |
| C3 | Arquiteto de IA/Cloud | Head of IT | OWASP LLM · AWS SMM |
| C4 | DevSecOps | Head of IT | ZAP · Sigstore · SBOM |

## Jobs do pipeline (.github/workflows/security.yml)

```mermaid
flowchart LR
    A[secret-scan<br/>Gitleaks] --> G[security-gate]
    B[codeql<br/>SAST matrix] --> G
    C[dependency-scan<br/>OSV-Scanner] --> G
    D[iac-scan<br/>Checkov] --> G
    E[license-scan<br/>ScanCode] --> G
    F[sbom<br/>Syft → CycloneDX] --> H[sign<br/>cosign keyless]
    H --> G
    I[zap-dast<br/>OWASP ZAP nightly] --> G
    J[claude-audit<br/>Claude Code] --> G
    G -->|verdict JSON| K[PR comment<br/>+ required check]
```

## Classificação de dado (MDP)

| Nível | Descrição | Exemplo no MDP | Controles mínimos |
|-------|-----------|----------------|-------------------|
| **L0** | Público | Marca, posts institucionais | Nenhum específico |
| **L1** | Interno | Documentação técnica | Auth corporativa |
| **L2** | Confidencial | Contratos, KPIs agregados | RBAC + log de acesso |
| **L3** | Restrito | Dado pessoal de adulto (staff) | ASVS L2 + CMK compartilhada |
| **L4** | **Dado de menor** | InBody, RLP, vídeo, métricas de Lucas e coortes | **ASVS L3 · CMK dedicada por tenant · DPIA obrigatória · consentimento parental co-titulado · retenção mínima necessária · logs sem PII** |

## Vereditos possíveis

| Verdict | Condição | Efeito |
|---------|----------|--------|
| ✅ `APPROVED` | 0 CRITICAL/HIGH · C1 completa · MEDIUM com plano | Libera *merge* após 2× review humana |
| ⚠️ `APPROVED_WITH_CAVEATS` | MEDIUM/LOW com *risk acceptance* formal + prazo | Libera com assinatura do Head of IT |
| ❌ `BLOCKED` | qualquer CRITICAL · HIGH sem mitigação · C1 incompleta · dado L4 sem DPIA · secret exposto | *Merge* bloqueado; secret deve ser rotacionado imediatamente |

---

*Diagramas em Mermaid — renderizam nativamente no GitHub. Exporte para SVG via
[mermaid.live](https://mermaid.live) se precisar embutir no PPTX da tese.*
