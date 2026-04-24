# Exemplo de saída do auditor — `audit:pr 42`

Este documento mostra um exemplo **realista** do que o Claude Code deve devolver ao
final de uma auditoria, no formato descrito em `CLAUDE.md`. Use como referência para
validar integrações (ex.: parser do job `security-gate`).

> ⚠️ **Ficção técnica.** O PR abaixo não existe; o exemplo foi construído para cobrir
> cada severidade e cada camada pelo menos uma vez.

---

## 1) Resumo Executivo

**Veredito:** ❌ **BLOQUEADO**

**Contagem:** 1 CRITICAL · 2 HIGH · 3 MEDIUM · 2 LOW · 1 INFO

**Top 3 riscos (ordem de impacto):**
1. Secret AWS (AKIA…) hardcoded em `infra/terraform/dev.tfvars:12` — credencial
   precisa ser rotacionada imediatamente no IAM, mesmo que o commit seja revertido.
2. SQL injection em `src/mdp/api/athlete_handler.py:77` via concatenação de
   `athlete_id` vindo de *query string* sem validação.
3. Falta de DPIA declarada para o novo endpoint `/v1/minor/performance` que retorna
   dado L4 (atleta menor) — violação de LGPD Art. 14 e da política MDP.

**Ação imediata:** revogar a chave IAM `AKIAEXAMPLE1234567890`, rotacionar a secret,
abrir ticket ITIL de incidente (CAT-1), e completar DPIA antes de reabrir a PR.

---

## 2) Achados Detalhados (JSON válido)

```json
{
  "audit_version": "1.0",
  "timestamp": "2026-04-24T14:32:11Z",
  "artifact": "PR#42 — feat(mdp): expose minor performance endpoint",
  "verdict": "BLOCKED",
  "counts": { "critical": 1, "high": 2, "medium": 3, "low": 2, "info": 1 },
  "findings": [
    {
      "id": "F-001",
      "severity": "CRITICAL",
      "layer": "C2",
      "framework_refs": ["OWASP A07", "ASVS V2.10.4", "CWE-798", "LGPD Art. 46"],
      "file": "infra/terraform/dev.tfvars",
      "line_range": "12-12",
      "finding": "AWS access key hardcoded em arquivo versionado",
      "evidence": "aws_access_key_id = \"AKIAEXAMPLE1234567890\"",
      "remediation": "Remover a chave do arquivo; mover para AWS Secrets Manager e referenciar via 'data \"aws_secretsmanager_secret_version\"'. Adicionar 'dev.tfvars' ao .gitignore se contiver dados de ambiente.",
      "autofix_safe": false,
      "rationale": "Credencial de longo prazo (AKIA*) exposta no histórico. Mesmo removendo no próximo commit, git preserva blob; rotacionar é mandatório."
    },
    {
      "id": "F-002",
      "severity": "HIGH",
      "layer": "C2",
      "framework_refs": ["OWASP A03", "ASVS V5.3.4", "CWE-89"],
      "file": "src/mdp/api/athlete_handler.py",
      "line_range": "74-82",
      "finding": "SQL injection via f-string em query com parâmetro do request",
      "evidence": "query = f\"SELECT * FROM athletes WHERE id = {athlete_id}\"",
      "remediation": "Usar parametrização: cursor.execute(\"SELECT * FROM athletes WHERE id = %s\", (athlete_id,))",
      "autofix_safe": true,
      "rationale": "athlete_id vem de request.args sem validação de tipo. Mesmo sendo lido (SELECT), habilita DoS e leitura cruzada entre tenants."
    },
    {
      "id": "F-003",
      "severity": "HIGH",
      "layer": "C1",
      "framework_refs": ["LGPD Art. 14", "LGPD Art. 38", "ANPD Res. 2/2022"],
      "file": "src/mdp/api/athlete_handler.py",
      "line_range": "1-1",
      "finding": "Endpoint de dado L4 (atleta menor) sem DPIA/RIPD referenciado",
      "evidence": "# POST /v1/minor/performance  (nenhum link para DPIA no PR)",
      "remediation": "Anexar RIPD aprovada à PR (campo 'DPIA' do template); referenciar ticket ITIL de Change Enablement; atualizar RoPA.",
      "autofix_safe": false,
      "rationale": "LGPD Art. 14 exige DPIA para tratamento de dado de criança/adolescente. Ausência bloqueia a promoção por não-conformidade."
    },
    {
      "id": "F-004",
      "severity": "MEDIUM",
      "layer": "C3",
      "framework_refs": ["OWASP LLM02", "OWASP LLM05", "ASVS V8.3.4"],
      "file": "src/mdp/ai/coach_bot.py",
      "line_range": "45-58",
      "finding": "Saída do LLM inserida em HTML sem sanitização; log inclui transcript com PII",
      "evidence": "return f\"<div>{llm_response}</div>\"  # ... logger.info(session_transcript)",
      "remediation": "Sanitizar HTML (bleach/html.escape); remover transcript do log ou reduzir a hash/ID opaco.",
      "autofix_safe": false,
      "rationale": "Improper Output Handling (LLM05) permite XSS refletido; Sensitive Information Disclosure (LLM02) expõe PII L4 em SIEM."
    },
    {
      "id": "F-005",
      "severity": "MEDIUM",
      "layer": "C3",
      "framework_refs": ["AWS CAF Security", "ASVS V1.6.2", "ISO 27001 A.8.24"],
      "file": "infra/terraform/kms.tf",
      "line_range": "8-15",
      "finding": "KMS CMK sem rotação automática habilitada",
      "evidence": "resource \"aws_kms_key\" \"mdp\" { enable_key_rotation = false }",
      "remediation": "Alterar para 'enable_key_rotation = true'. Validar em ambientes não-prod primeiro.",
      "autofix_safe": true,
      "rationale": "Chave criptográfica de dado L3/L4 sem rotação fere baseline AWS SMM Phase 3."
    },
    {
      "id": "F-006",
      "severity": "MEDIUM",
      "layer": "C4",
      "framework_refs": ["NIST SSDF PS.2.1", "Sigstore Policy"],
      "file": ".github/workflows/release.yml",
      "line_range": "N/A",
      "finding": "Pipeline de release não assina artefato gerado (ausência de cosign)",
      "evidence": "workflow contém 'docker push' sem step subsequente de 'cosign sign'.",
      "remediation": "Adicionar step cosign keyless após push; publicar assinatura como artefato.",
      "autofix_safe": false,
      "rationale": "Sem signing, consumidores downstream não conseguem verificar proveniência — fere cadeia de suprimentos."
    },
    {
      "id": "F-007",
      "severity": "LOW",
      "layer": "C2",
      "framework_refs": ["OWASP A05", "ASVS V14.4.5"],
      "file": "src/mdp/api/app.py",
      "line_range": "22-26",
      "finding": "Headers de segurança ausentes (CSP, HSTS, X-Content-Type-Options)",
      "evidence": "app.add_middleware(CORSMiddleware, allow_origins=[\"*\"])",
      "remediation": "Adicionar SecurityHeadersMiddleware com CSP restritiva, HSTS max-age≥31536000, XCTO nosniff. Remover allow_origins=\"*\" em produção.",
      "autofix_safe": false,
      "rationale": "CORS aberto e ausência de HSTS em serviço que manipula dado L4 amplia superfície XSS/CSRF."
    },
    {
      "id": "F-008",
      "severity": "LOW",
      "layer": "C2",
      "framework_refs": ["OWASP A09", "ASVS V7.1.3"],
      "file": "src/mdp/api/athlete_handler.py",
      "line_range": "92-92",
      "finding": "Log sem correlation-id nem structured logging",
      "evidence": "print(f\"request for {athlete_id} ok\")",
      "remediation": "Substituir por logger estruturado (structlog/loguru) com correlation_id, sem PII.",
      "autofix_safe": true,
      "rationale": "Dificulta investigação de incidente e correlação no SIEM."
    },
    {
      "id": "F-009",
      "severity": "INFO",
      "layer": "C2",
      "framework_refs": ["Dependabot"],
      "file": "requirements.txt",
      "line_range": "5-5",
      "finding": "Pacote 'requests' em versão 2.30.0 — há 2.32.3 disponível",
      "evidence": "requests==2.30.0",
      "remediation": "Subir para 2.32.3 (sem CVE conhecida, apenas higiene).",
      "autofix_safe": true,
      "rationale": "Sem CVE ativa, mas alinhamento com política de atualização contínua."
    }
  ],
  "out_of_scope": [
    {
      "path": "vendor/third_party_lib/",
      "reason": "Biblioteca vendorizada sem lockfile próprio; reportar ao mantenedor upstream."
    }
  ],
  "next_actions": [
    "Rotacionar IAM key AKIAEXAMPLE1234567890 imediatamente (AWS Console → IAM → Deactivate + Delete).",
    "Abrir CHG-2026-xxxx no ITSM com categoria 'Security Incident' referenciando F-001.",
    "Corrigir F-002 com query parametrizada antes de reabrir a PR.",
    "Anexar DPIA aprovada ao PR (F-003) e registrar em RoPA atualizada.",
    "Sanitizar saída LLM e higienizar logs (F-004).",
    "Ativar key_rotation do KMS (F-005) via PR separada, ambiente dev primeiro.",
    "Adicionar cosign ao release workflow (F-006)."
  ]
}
```

---

## Como o `security-gate` consome esta saída

```bash
# Pseudo-código do job final:
VERDICT=$(jq -r .verdict audit.json)
if [ "$VERDICT" = "BLOCKED" ]; then
  echo "::error::Security Gate BLOCKED — veja comentário da PR."
  exit 1
fi
```

O comentário do Claude Code na PR deve incluir:
- Resumo executivo em PT-BR (máx. 10 linhas).
- JSON válido em code-fence dentro de `<details>` colapsável.
- Links para os frameworks citados (quando disponíveis em docs internas).

---

*Exemplo v1.0 — 2026-04-24. Atualizar junto com CLAUDE.md quando o schema evoluir.*
