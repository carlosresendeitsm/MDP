# MDP — Maestrinho Data Platform · Security Gate & Governance Kit

**Owner:** Carlos Resende — Head of IT (ênfase em Engenharia de Dados).
**Versão:** v1.0 — 24/04/2026.
**Licença:** MIT (ver `LICENSE`).

Kit de Governança de TI e Cibersegurança da **Maestrinho Data Platform (MDP)** —
plataforma SaaS multi-tenant de performance esportiva com tratamento de dados sensíveis
de atletas menores de 18 anos (LGPD Art. 14).

Este repositório consolida o *security gate* bloqueante aplicado a todo código-fonte
antes da promoção para produção, integrando frameworks reconhecidos em **4 camadas**
(C1–C4).

---

## 📦 Conteúdo

```
.
├── .github/
│   ├── CODEOWNERS                       # Revisores obrigatórios por path
│   ├── ISSUE_TEMPLATE/                  # Bug / feature / documentação + links
│   │   ├── bug_report.yml
│   │   ├── feature_request.yml
│   │   ├── documentation.yml
│   │   └── config.yml
│   ├── PULL_REQUEST_TEMPLATE.md         # Checklist C1-C4 + seção MDP condicional
│   ├── dependabot.yml                   # Atualização automática de deps (Actions/npm/pip/docker)
│   └── workflows/
│       └── security.yml                 # Pipeline CI/CD bloqueante (10 jobs)
├── docs/
│   ├── architecture.md                          # Diagramas Mermaid das 4 camadas + pipeline
│   ├── example_audit_output.md                  # Exemplo JSON de saída do auditor
│   ├── prompt_auditoria_claude_code.md          # Prompt formal Claude Code
│   ├── validacao_codigos_seguranca.md           # Template-base (genérico)
│   └── validacao_codigos_seguranca_MDP.md       # Extensão LGPD Art. 14 (menor)
├── CLAUDE.md                            # System prompt oficial do Claude Code
├── CHANGELOG.md                         # Keep a Changelog (v1.0)
├── CONTRIBUTING.md                      # Fluxo de PR, commits, hooks, assinatura
├── SECURITY.md                          # Política de responsible disclosure
├── LICENSE                              # MIT © 2026 Carlos Resende
├── .gitignore                           # Cobre secrets, reports, IaC, Python, Node
└── README.md                            # este arquivo
```

---

## 🏛️ Arquitetura em 4 Camadas (C1-C4)

| Camada | Domínio | Frameworks |
|---|---|---|
| **C1** — Governança e Conformidade | Política, regulatório, ciclo de vida | ISO/IEC 27001, 20000-1, COBIT 2019, ITIL v4, **LGPD** |
| **C2** — Desenvolvimento Seguro | Código, deps, identidade | OWASP Top 10, ASVS v4.0.3, MFA/FIDO2, GitHub CodeQL + Autofix |
| **C3** — IA, Dados e Cloud | LLMs, dados sensíveis, IaC, cloud posture | OWASP Top 10 LLM (2025), AWS SMM, AWS CAF |
| **C4** — Verificação Dinâmica | Runtime, DAST, pipeline | OWASP ZAP, GitHub Actions, Code Review com IA |

> *Um artefato só é promovido quando satisfaz as quatro camadas simultaneamente.*

Detalhes completos em [`docs/validacao_codigos_seguranca.md`](docs/validacao_codigos_seguranca.md).
Diagrama visual (Mermaid, renderiza direto no GitHub) em
[`docs/architecture.md`](docs/architecture.md).
Exemplo de saída do auditor em
[`docs/example_audit_output.md`](docs/example_audit_output.md).

---

## 🧒 Extensão MDP — LGPD Art. 14 (dado de menor)

Aplicação obrigatória sempre que o artefato manipular dados biométricos, de saúde,
geolocalização, wellness, imagem ou comunicação de atleta menor de 18 anos.

Endurecimentos específicos:

- Classificação granular **L0–L4** (L4 = dado pessoal sensível de menor).
- **ASVS Nível 3** em todos os caminhos que manipulam L4.
- **CMK dedicada por tenant** (sem chave compartilhada).
- **Consentimento parental específico e em destaque** (co-titularidade).
- **Proibição absoluta** de L4 em logs, analytics sem pseudonimização, publicidade,
  fine-tune de modelo compartilhado ou integração cross-tenant.
- **Guardrails de LLM** obrigatórios para assistentes que conversam com menor.
- **Declaração proativa de conflito de interesse** quando há integração com clube no
  qual o responsável legal atua profissionalmente.
- **Janela de deploy** respeita agenda do atleta (treino, jogo, peneira, viagem).

Completo em [`docs/validacao_codigos_seguranca_MDP.md`](docs/validacao_codigos_seguranca_MDP.md).

---

## 🤖 Prompt Claude Code

O `CLAUDE.md` na raiz contém o *system prompt* oficial do agente auditor. Ao abrir
qualquer sessão do Claude Code neste repositório, o prompt é carregado automaticamente
e expõe os comandos:

| Comando | Efeito |
|---|---|
| `audit:pr <n>` | Audita a PR completa |
| `audit:file <path>` | Audita um arquivo |
| `audit:dir <path>` | Audita um diretório |
| `audit:diff <a..b>` | Audita um range de commits |
| `audit:llm <path>` | Foco em OWASP LLM Top 10 (2025) |
| `audit:iac <path>` | Foco em AWS SMM + CAF (IaC) |
| `audit:quick` | Smoke test (C1 + secrets + CVE crítica) |
| `audit:full` | Varredura completa C1-C4 |

Detalhes e opções de implantação em
[`docs/prompt_auditoria_claude_code.md`](docs/prompt_auditoria_claude_code.md).

---

## 🔁 CI — Pipeline `security.yml`

Gatilhos: `push` (main/develop/release), `pull_request`, `schedule` (03:00 UTC, varredura
noturna) e `workflow_dispatch`.

Jobs encadeados (falha em qualquer um bloqueia o merge):

1. `secret-scan` — Gitleaks
2. `codeql` — SAST com query packs `security-and-quality` + `security-extended`
3. `dependency-scan` — OSV-Scanner
4. `iac-scan` — Checkov (Terraform/CloudFormation/K8s/Helm/Docker/Actions)
5. `license-scan` — ScanCode Toolkit (bloqueia copyleft forte)
6. `sbom` — Syft (CycloneDX)
7. `sign` — cosign keyless (Sigstore) — apenas em push
8. `zap-dast` — OWASP ZAP Baseline — nightly/manual
9. `claude-audit` — Claude Code com prompt C1-C4 — comenta no PR
10. `security-gate` — veredito final (bloqueia merge se qualquer upstream falhou)

**Secrets requeridos:**

- `ANTHROPIC_API_KEY` — para o job `claude-audit`.
- `GITLEAKS_LICENSE` — se conta corporativa Gitleaks.

**Variáveis:**

- `STAGING_URL` — alvo do ZAP DAST.

---

## 🚀 Como usar este repositório como template

```bash
# 1. Clone o template
gh repo create meu-servico --template carlosresendeitsm/MDP --private

# 2. Configure secrets
gh secret set ANTHROPIC_API_KEY

# 3. Configure variáveis (opcional, para DAST)
gh variable set STAGING_URL --body "https://staging.meudominio.com"

# 4. Abra uma PR — o Security Gate dispara automaticamente
```

---

## 📋 Matriz de Controles (resumo)

| Framework | C1 | C2 | C3 | C4 |
|---|---|---|---|---|
| ISO 27001 / 27002 | ✅ | ✅ | ✅ | ✅ |
| ISO 20000-1 | ✅ | — | — | ✅ |
| COBIT 2019 | ✅ | ✅ | ✅ | ✅ |
| ITIL v4 | ✅ | — | ✅ | ✅ |
| LGPD | ✅ | ✅ | ✅ | ✅ |
| MFA / FIDO2 | — | ✅ | ✅ | — |
| OWASP Top 10 (2021) | — | ✅ | — | — |
| OWASP ASVS v4 | — | ✅ | — | — |
| OWASP ZAP | — | — | — | ✅ |
| OWASP LLM Top 10 (2025) | — | — | ✅ | — |
| AWS SMM (Phase 3+) | — | — | ✅ | — |
| AWS CAF — Security | ✅ | ✅ | ✅ | ✅ |
| GitHub CodeQL + Autofix | — | ✅ | — | — |
| GitHub Actions | — | ✅ | ✅ | ✅ |

---

## 🧭 Roadmap

| Trimestre | Entregável |
|---|---|
| **Q2/2026** | v1.0 publicada; piloto em 2 repositórios internos |
| **Q3/2026** | Métricas: *findings*/PR, taxa de falso-positivo, MTTR |
| **Q4/2026** | v1.1 com ajustes baseados em dados |
| **Q1/2027** | Revisão semestral contra novas versões OWASP/ASVS/LLM Top 10 |

---

## 🧱 Governança do repositório

| Arquivo | Função |
|---|---|
| [`CODEOWNERS`](.github/CODEOWNERS) | Revisores obrigatórios por path |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Fluxo de PR, Conventional Commits, hooks, assinatura |
| [`SECURITY.md`](SECURITY.md) | Política de *responsible disclosure* com SLA e *safe harbor* |
| [`CHANGELOG.md`](CHANGELOG.md) | Histórico de versões (Keep a Changelog) |
| [`.github/dependabot.yml`](.github/dependabot.yml) | Atualização automática de Actions/npm/pip/docker |
| [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE) | Bug / feature / documentação + links (LGPD, security) |

## 📞 Contato

- **Owner:** Carlos Resende — `Carlos.resende@safbfr.com.br` / `linkedin.com/in/carlosresendeitsm`
- **Issues abertas:** use o tracker deste repositório (ver templates em `.github/ISSUE_TEMPLATE/`).
- **Security disclosures:** ver [`SECURITY.md`](SECURITY.md) — **nunca** abrir *issue* pública.
- **Solicitações de titular LGPD:** canal do Controlador em `https://safbfr.com.br/lgpd`.

---

## 📜 Licença

MIT — ver [`LICENSE`](LICENSE).

---

*MDP Security Gate Kit v1.0 — construído para ambientes de alta criticidade (SAF de
elite, SaaS multi-tenant, plataformas esportivas com dado de menor). Uso livre,
contribuições bem-vindas.*
