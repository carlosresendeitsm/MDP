<!--
  PULL_REQUEST_TEMPLATE.md — Security Gate C1–C4
  Owner: Carlos Resende — Head of IT
  Referências: validacao_codigos_seguranca.md v1.0 e validacao_codigos_seguranca_MDP.md v1.0
  Colocar em: .github/PULL_REQUEST_TEMPLATE.md
-->

## 📝 Descrição

<!-- O que esta PR faz? Por quê? Qual ticket/história? -->

**Ticket ITIL / Issue:** #

**Classificação do dado manipulado** (marque apenas uma):
- [ ] L0 — Público
- [ ] L1 — Interno
- [ ] L2 — Confidencial
- [ ] L3 — Restrito
- [ ] L4 — **Dado Pessoal Sensível de Menor (Art. 14 LGPD)** ← *Exige MDP-specific checklist*

**Base legal LGPD aplicável** (se manipular dado pessoal):
<!-- Ex.: Art. 7º I (consentimento), Art. 11 (sensível), Art. 14 (menor), Art. 7º V (execução contrato) -->

---

## 🔐 Security Gate C1–C4

### C1 — Governança e Conformidade
- [ ] Ticket de mudança (ITIL Change Enablement) aprovado e referenciado acima.
- [ ] Classificação de dado declarada acima.
- [ ] Base legal LGPD declarada (ou N/A se sem dado pessoal).
- [ ] RoPA atualizado (ou N/A).
- [ ] DPIA vigente referenciada (obrigatória se L4 ou novo fluxo de dado sensível).
- [ ] SLA/SLO impactados documentados; não há degradação não-aprovada.
- [ ] Risco avaliado e aceito por *business owner* (COBIT EDM03).
- [ ] Plano de rollback descrito.

### C2 — Desenvolvimento Seguro (AppSec)
- [ ] **OWASP Top 10 (2021):** cobertura revisada para A01–A10.
- [ ] **OWASP ASVS:** nível atendido → `[ ] N1  [ ] N2  [ ] N3`
- [ ] **Segredos:** nenhum hardcoded; *secret scanning* verde; *push protection* ativo.
- [ ] **Criptografia:** TLS 1.2+; AES-256; hashes modernos (Argon2id/bcrypt); sem MD5/SHA1/DES/RC4/ECB.
- [ ] **Injeção:** queries parametrizadas; *input validation* central; *output encoding*.
- [ ] **MFA:** FIDO2/WebAuthn em acesso admin e contas com escrita crítica.
- [ ] **Dependências:** `dependabot`/OSV verde; nenhuma CVE ≥ 7.0 aberta sem exceção formal.
- [ ] **CodeQL (SAST):** zero `HIGH`/`CRITICAL` no *default branch*; `MEDIUM` com plano.
- [ ] **Code Scanning Autofix:** sugestões aceitas foram revisadas semanticamente (sem *autofix* cego).

### C3 — IA, Dados e Cloud
- [ ] **LLM/IA (se aplicável):** OWASP LLM Top 10 (2025) revisado — atenção a `LLM01` (Prompt Injection), `LLM05` (Output Handling), `LLM06` (Excessive Agency), `LLM10` (Unbounded Consumption).
- [ ] **Tools do agente** com *allowlist* e *scopes* mínimos; *human-in-the-loop* em ações destrutivas.
- [ ] **IaC (Terraform/CloudFormation/K8s):** `Checkov` verde; sem recursos públicos não-intencionais; SGs fechados; IAM sem `*:*`.
- [ ] **AWS SMM Phase ≥ 3:** CloudTrail, GuardDuty, Config, Security Hub ativos.
- [ ] **KMS:** CMK dedicada; rotação automática; nenhum dado L3/L4 em chave compartilhada.
- [ ] **Pseudonimização:** dado pessoal em ambiente de analytics está pseudonimizado.

### C4 — Verificação Dinâmica e Operacional
- [ ] Pipeline `.github/workflows/security.yml` verde para esta PR.
- [ ] SBOM gerado (CycloneDX) e assinado (cosign/Sigstore) para artefatos promovíveis.
- [ ] **OWASP ZAP Baseline** rodou em *staging* (schedule/manual) sem `HIGH` em endpoint autenticado.
- [ ] Logs estruturados, correlacionáveis, **sem PII/L3/L4 em claro**.
- [ ] Métricas/tracing em caminhos críticos.
- [ ] Canary/feature flag disponível para rollback rápido.

---

## 🧒 Checklist MDP — Apenas se marcou `L4` acima

<!-- Se marcou L4, TODOS os itens abaixo são obrigatórios. Não marcou L4? Ignore esta seção. -->

- [ ] **Consentimento parental específico e em destaque** (LGPD Art. 14 §1º) válido e vigente.
- [ ] **Co-titularidade de consentimento** respeitada (ambos os responsáveis, quando aplicável).
- [ ] **DPIA** vigente (< 12 meses) referenciada.
- [ ] **Política de privacidade infanto-juvenil** cobre a nova finalidade.
- [ ] Dado L4 jamais usado para: publicidade, inferência comercial, *cross-tenant*, treino de modelo base.
- [ ] **ASVS Nível 3** satisfeito nos trechos que manipulam L4.
- [ ] **CMK dedicada por tenant**; nada em chave compartilhada.
- [ ] **Logs sem L4 em claro** (testado com *scrubbing* de amostra).
- [ ] **LLM/Assistente em contato com menor:** guardrails testados; *refusal* default para temas fora de formação esportiva, nutrição aprovada e bem-estar.
- [ ] **Conflito de interesse (SAF Botafogo)** declarado e tratado, se a mudança tangencia integração com clube da base.
- [ ] **Janela de deploy** não coincide com treino, jogo, peneira ou viagem do atleta.
- [ ] **Retenção:** política de *hard delete* pós-contrato preservada.
- [ ] DPO ciente e assinatura registrada.

---

## 🤖 Claude Code Audit

<!-- Cole a saída resumida do audit:pr abaixo. Full JSON ficará como comentário da PR. -->

```
Veredito: [ APPROVED | APPROVED_WITH_CAVEATS | BLOCKED ]
Contagem: 🔴 0  🟠 0  🟡 0  🔵 0
Top 3 riscos:
  1.
  2.
  3.
Ação imediata recomendada:
```

---

## ✅ Testes

- [ ] Unit tests passando (cobertura ≥ 80% nas linhas alteradas).
- [ ] Integration tests passando.
- [ ] Teste de regressão de privacidade (se L3/L4).
- [ ] Teste manual documentado no ticket (se UX-relevant).

## 👥 Revisores

- [ ] Ao menos **2 revisores humanos**, sendo pelo menos 1 sênior.
- [ ] Se L4: **DPO** + **co-titular de consentimento** (Carlos + Michelle, para caso Lucas) adicionados como *reviewers*.
- [ ] Se tangencia SAF Botafogo: **conselho consultivo familiar** notificado.

## 🚀 Plano de Deploy

- [ ] Ambiente(s): `[ ] dev  [ ] staging  [ ] prod`
- [ ] Estratégia: `[ ] blue-green  [ ] canary  [ ] rolling  [ ] big-bang`
- [ ] Janela: `YYYY-MM-DD HH:MM BRT` — conferida contra agenda do atleta (se MDP).
- [ ] Comunicação a stakeholders (Slack/email) agendada.

---

## 📎 Referências

- Template-base: `Governanca_TI/validacao_codigos_seguranca.md` v1.0
- Extensão MDP (Art. 14): `Governanca_TI/validacao_codigos_seguranca_MDP.md` v1.0
- Prompt Claude Code: `Governanca_TI/prompt_auditoria_claude_code.md` v1.0
- Pipeline: `.github/workflows/security.yml`

---

> ⚠️ **Merge bloqueado automaticamente** se qualquer job obrigatório falhar, se qualquer item bloqueante acima ficar desmarcado, se houver `HIGH`/`CRITICAL` em aberto ou se o veredito do Claude Code for `BLOCKED`.
