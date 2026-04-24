# Changelog

Todos os lançamentos notáveis deste projeto são documentados aqui.

O formato segue [Keep a Changelog 1.1](https://keepachangelog.com/pt-BR/1.1.0/) e este
projeto adere ao [Semantic Versioning 2.0](https://semver.org/lang/pt-BR/).

## [Unreleased]
### Planejado
- Integração com ServiceNow para abertura automática de ticket ITIL em veredito BLOQUEADO.
- Exportador CSV de findings para relatório trimestral ao Comitê de Governança.
- Suporte a SARIF unificado para upload automático em Code Scanning.

## [1.0.0] — 2026-04-24
### Adicionado
- **CLAUDE.md v1.0** — política ativa do Auditor de Segurança Sênior (Claude Code)
  com fluxo obrigatório de 7 etapas, formato JSON de saída e 10 Regras de Ouro.
- **`docs/validacao_codigos_seguranca.md`** — template base (SAF/geral) com 4 camadas
  C1-C4, matriz de frameworks (ISO 27001/20000, COBIT 2019, ITIL v4, LGPD, OWASP Top 10,
  ASVS v4.0.3, OWASP LLM Top 10 2025, AWS SMM/CAF) e critérios de *deployment gate*.
- **`docs/validacao_codigos_seguranca_MDP.md`** — overlay específico para dado sensível
  de atleta menor (LGPD Art. 14), com classificação L0-L4, DPIA, CMK por tenant,
  consentimento parental (co-titularidade), *guardrails* LLM para conversa com menor e
  declaração proativa de conflito de interesse (SAF × clube de origem).
- **`docs/prompt_auditoria_claude_code.md`** — prompt formal completo com 8 comandos
  de invocação (`audit:pr`, `audit:file`, `audit:dir`, `audit:diff`, `audit:llm`,
  `audit:iac`, `audit:quick`, `audit:full`).
- **`.github/workflows/security.yml`** — pipeline com 10 jobs:
  `secret-scan` (Gitleaks), `codeql` (matrix py/js-ts/java-kotlin/go/csharp),
  `dependency-scan` (OSV-Scanner), `iac-scan` (Checkov), `license-scan` (ScanCode),
  `sbom` (Syft/CycloneDX), `sign` (cosign keyless — Sigstore), `zap-dast` (OWASP ZAP,
  noturno + manual), `claude-audit` (comentário na PR), `security-gate` (veredito final).
- **`.github/PULL_REQUEST_TEMPLATE.md`** — checklist C1-C4 com campo obrigatório de
  classificação de dado (L0-L4) e seção condicional MDP.
- **`.github/CODEOWNERS`** — revisores obrigatórios por path.
- **`.github/dependabot.yml`** — atualização automática de Actions, npm e pip.
- **`.github/ISSUE_TEMPLATE/`** — bug report, feature request, security vulnerability e
  LGPD data request.
- **`SECURITY.md`** — política de *responsible disclosure* com SLA, escopo e *safe harbor*.
- **`CONTRIBUTING.md`** — fluxo de PR, Conventional Commits, assinatura GPG/SSH,
  hooks pre-commit.
- **`docs/architecture.md`** — diagrama Mermaid das 4 camadas C1-C4 e fluxo de gate.
- **`docs/example_audit_output.md`** — exemplo de saída JSON de auditoria válida.
- **`.gitignore`** — cobre segredos, relatórios gerados pelo pipeline, IaC, Python,
  Node, IDE, OS e temporários.
- **`LICENSE`** — MIT, © 2026 Carlos Resende.
- **`README.md`** — visão geral, arquitetura, setup, referências a todos os artefatos.

### Frameworks cobertos (v1.0)
LGPD (13.709/2018) · ISO/IEC 27001 · ISO/IEC 20000-1 · COBIT 2019 · ITIL v4 ·
OWASP Top 10 (2021) · OWASP ASVS v4.0.3 · OWASP Top 10 for LLM & GenAI Apps (2025) ·
OWASP ZAP · AWS Security Maturity Model (Phase 3+) · AWS Cloud Adoption Framework
(Security) · GitHub CodeQL · Code Scanning Autofix · Secret Scanning · Dependabot ·
CycloneDX SBOM · Sigstore/cosign · ECA · Marco Civil da Internet · ANPD Res. 2/2022.

### Segurança
- Nenhum segredo hardcoded no repositório (verificado via Gitleaks no commit inicial).
- Pipeline configurado para *fail-closed*: qualquer job obrigatório em falha bloqueia o
  merge através do job `security-gate`.

---

[Unreleased]: https://github.com/carlosresendeitsm/MDP/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/carlosresendeitsm/MDP/releases/tag/v1.0.0
