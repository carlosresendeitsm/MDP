# Validação de Códigos e Segurança — Template Integrado

> **Propósito:** padronizar o *security gate* aplicado a todo código-fonte antes de promoção para produção (ou *main*), consolidando exigências de governança (ISO/COBIT/ITIL), privacidade (LGPD), AppSec (OWASP), IA Generativa (OWASP LLM Top 10), Cloud (AWS SMM + CAF) e automação (GitHub Code Scanning / Actions).
> **Público:** squads de Engenharia, SRE, DevSecOps, CCoE, Arquitetura e Compliance.
> **Autor / Owner:** Carlos Resende — Head of IT (ênfase em Engenharia de Dados).
> **Versão:** v1.0 — 24/04/2026.

---

## 1. Arquitetura de 4 Camadas do Security Gate

| Camada | Domínio | Frameworks de referência | Objetivo de controle |
|---|---|---|---|
| **C1 — Governança e Conformidade** | Política, ciclo de vida, regulatório | ISO/IEC 27001, ISO/IEC 27002, ISO/IEC 20000-1, COBIT 2019, ITIL v4, LGPD (Lei 13.709/2018) | Garantir que **o que** está sendo entregue está autorizado, classificado, rastreável e conforme. |
| **C2 — Desenvolvimento Seguro (AppSec)** | Código, dependências, identidade | OWASP Top 10 (2021/atual), OWASP ASVS v4.0.3, MFA/FIDO2, GitHub CodeQL (SAST), Code Scanning Autofix | Garantir que **como** o código foi escrito não introduz vulnerabilidade nem credencial exposta. |
| **C3 — IA, Dados e Cloud (Contextual)** | LLMs, dados sensíveis, IaC, cloud posture | OWASP Top 10 for LLM & GenAI Apps (2025), AWS Security Maturity Model (SMM), AWS Cloud Adoption Framework (CAF — Security perspective) | Garantir que **onde** e **com o quê** o código opera (modelos, prompts, buckets, IAM) está endurecido. |
| **C4 — Verificação Dinâmica e Operacional** | Runtime, DAST, pipeline | OWASP ZAP (DAST), GitHub Actions (CI/CD), Code Review com IA, Navegação de Código (Code Search / Graph), observabilidade | Garantir que **quando** o código roda em ambiente simulado ou produção, o comportamento efetivo corresponde ao esperado. |

### Princípio norteador

> *"Um artefato só é promovido quando satisfaz **as quatro camadas simultaneamente**. Falha em qualquer uma bloqueia o *merge* ou o *deploy*."* — Política de *Security Gate* MDP/SAF (draft v1.0).

---

## 2. Camada C1 — Governança e Conformidade

### 2.1 Controles mínimos (checklist bloqueante)

- [ ] **Classificação de dados** (ISO 27001 A.5.12): artefato declara a classe de dados manipulada (Público / Interno / Confidencial / Restrito / Dado Pessoal Sensível-LGPD).
- [ ] **Base legal LGPD** (Art. 7º/11): se manipular dado pessoal, registrar base legal (consentimento, execução de contrato, legítimo interesse, tutela de menor, etc.) no cabeçalho do PR.
- [ ] **RoPA / Registro de Atividades de Tratamento** (LGPD Art. 37) atualizado quando há novo fluxo de dado pessoal.
- [ ] **DPIA / Relatório de Impacto** (LGPD Art. 38) exigido quando há dado sensível ou de menor (ex.: dados biométricos de atleta Sub 11).
- [ ] **Change Management** (ITIL v4 — *Change Enablement*): ticket de mudança aprovado com rollback plan e janela de deploy.
- [ ] **Service Management** (ISO 20000-1): SLA/SLO do serviço afetado documentado e não degradado pela mudança.
- [ ] **Governança de TI** (COBIT 2019 — EDM03/APO12/APO13/DSS05): risco identificado, avaliado, tratado e aceito por *business owner*.
- [ ] **Separação de ambientes**: Dev/UAT/Prod isolados; dado de produção nunca reside em ambiente inferior sem anonimização.

### 2.2 Matriz de rastreabilidade LGPD

| Requisito LGPD | Evidência exigida no PR |
|---|---|
| Finalidade específica (Art. 6º I) | Comentário no PR descrevendo a finalidade do tratamento |
| Adequação e Necessidade (Art. 6º II-III) | Justificativa de *data minimization* — nenhum campo extra coletado |
| Transparência (Art. 6º VI) | Atualização da Política de Privacidade se houver nova coleta |
| Segurança (Art. 6º VII) | Controles C2/C3/C4 deste template aplicados |
| Prevenção (Art. 6º VIII) | Registro de log de auditoria para operações críticas |
| Não discriminação (Art. 6º IX) | Modelo/regra não usa atributo sensível como feature discriminatória |
| Responsabilização (Art. 6º X) | Owner técnico e DPO cientes, assinaturas no PR |

---

## 3. Camada C2 — Desenvolvimento Seguro (AppSec)

### 3.1 OWASP Top 10 (2021) — checklist

- [ ] **A01 — Broken Access Control**: ACL testada; *least privilege*; IDOR prevenido; CORS restrito.
- [ ] **A02 — Cryptographic Failures**: TLS 1.2+; AES-256; senhas com Argon2id/bcrypt; segredos fora do repositório.
- [ ] **A03 — Injection**: queries parametrizadas; ORM com *prepared statements*; input validation centralizado; *output encoding*.
- [ ] **A04 — Insecure Design**: *threat model* anexado para mudanças arquiteturais.
- [ ] **A05 — Security Misconfiguration**: *headers* de segurança (HSTS, CSP, X-Frame-Options); *hardening* de containers; *default deny*.
- [ ] **A06 — Vulnerable & Outdated Components**: `dependabot` ou equivalente verde; SBOM atualizado.
- [ ] **A07 — Identification & Authentication Failures**: MFA obrigatório em superfícies administrativas; sessões com *rotation* e *expiration*; rate-limit no login.
- [ ] **A08 — Software & Data Integrity Failures**: *signed commits*; *artifact signing* (Sigstore/cosign); *SLSA* nível alvo ≥ 2.
- [ ] **A09 — Security Logging & Monitoring Failures**: log estruturado; *alerting* em eventos críticos; retenção conforme LGPD.
- [ ] **A10 — Server-Side Request Forgery (SSRF)**: *allowlist* de egress; metadados de cloud (IMDSv2) protegidos.

### 3.2 OWASP ASVS v4.0.3 — nível alvo

- **Aplicações públicas / que tratam dado pessoal sensível:** **ASVS Nível 2** (mínimo).
- **Aplicações financeiras, autenticação, MDP Core:** **ASVS Nível 3**.
- Controles prioritários: V1 (Arquitetura), V2 (Auth), V3 (Session), V4 (Access Control), V5 (Validation), V6 (Crypto), V7 (Error/Logging), V8 (Data), V9 (Communication), V10 (Malicious), V11 (BusLogic), V12 (Files), V13 (API), V14 (Config).

### 3.3 Identidade e MFA

- [ ] MFA FIDO2/WebAuthn ou TOTP em 100% das contas humanas com acesso a console, repositório, painel cloud e banco.
- [ ] *Break-glass accounts* existentes, com MFA em hardware key isolada e monitoramento de uso.
- [ ] *Service accounts* não reutilizam credenciais humanas; rotação automática ≤ 90 dias.

### 3.4 GitHub Code Scanning (SAST com CodeQL)

- [ ] Workflow `codeql.yml` ativo no repositório, gatilhos `push` + `pull_request` + `schedule` (semanal).
- [ ] Query packs: `security-and-quality` + `security-extended` para linguagens críticas (Python, JS/TS, Java, Go, C#, Ruby, Kotlin, Swift).
- [ ] **Code Scanning Autofix** habilitado para PRs — análise crítica humana antes do merge mesmo com *autofix* sugerido.
- [ ] **Zero *high*/*critical* aberto** no *default branch*; *medium* com plano de remediação formal.
- [ ] *Secret scanning* + *push protection* ativados; *historical scan* periódico.

---

## 4. Camada C3 — IA, Dados e Cloud (Contextual)

### 4.1 OWASP Top 10 for LLM & GenAI Applications (2025) — checklist

- [ ] **LLM01 — Prompt Injection**: *system prompts* isolados; *untrusted input* nunca concatenado direto com *tool calls*; detecção de *jailbreak patterns*.
- [ ] **LLM02 — Sensitive Information Disclosure**: *output filter* para PII/segredos; *redaction* antes de logs; *zero retention* quando contrato permitir.
- [ ] **LLM03 — Supply Chain**: modelos, *embeddings*, datasets e *fine-tunes* com procedência verificada (hash, assinatura, SBOM de modelo).
- [ ] **LLM04 — Data & Model Poisoning**: *training data* validado; *RAG corpora* com *integrity checks*; *guardrails* contra *data drift* malicioso.
- [ ] **LLM05 — Improper Output Handling**: *output* do LLM nunca executado direto (eval/exec/shell); *escaping* antes de renderizar em HTML/SQL/shell.
- [ ] **LLM06 — Excessive Agency**: *tool allowlist*; *scopes* mínimos; *human-in-the-loop* para ações destrutivas; *circuit breaker* de custo/volume.
- [ ] **LLM07 — System Prompt Leakage**: prompts sensíveis não embutem segredo; assume-se que prompt vaza → proteção no *backend*, não no prompt.
- [ ] **LLM08 — Vector & Embedding Weaknesses**: isolamento por tenant em índices vetoriais; *access control* em *retrieval*.
- [ ] **LLM09 — Misinformation / Overreliance**: *confidence scoring*; *disclaimers*; validação humana obrigatória para decisões de alto impacto.
- [ ] **LLM10 — Unbounded Consumption**: *rate limiting*, *cost caps*, *timeout*, *token budget* por sessão/usuário.

### 4.2 AWS Security Maturity Model (SMM) — nível alvo

- **Nível-alvo da organização:** **Phase 3 — Defined** (mínimo) com trilha para **Phase 4 — Managed** em 12 meses.
- Pilares: Identidade, Rede, Detecção, Proteção de Dados, Resposta a Incidentes, Governança e Automação.
- [ ] AWS Organizations com SCPs; Control Tower ou equivalente.
- [ ] GuardDuty + Security Hub + Config + CloudTrail (multi-region, *log validation*) ativos.
- [ ] Macie para S3 contendo dado pessoal.
- [ ] KMS com CMK por domínio; rotação automática.
- [ ] *IaC scanning* (Checkov / tfsec / cfn-nag) no pipeline.

### 4.3 AWS Cloud Adoption Framework (CAF) — Security Perspective

- [ ] **Directive capabilities:** política, *risk management*, *compliance* mapeados a ISO 27001 e LGPD.
- [ ] **Preventive capabilities:** IAM, *data protection*, *infrastructure protection*, *application security*.
- [ ] **Detective capabilities:** *logging & monitoring*, *vulnerability management*, *threat detection*.
- [ ] **Responsive capabilities:** IR plan, *automated response* (Lambda/EventBridge/Step Functions).

### 4.4 Dados sensíveis de menor (contexto SAF / MDP)

- [ ] Atleta menor de idade: tratamento sob Art. 14 LGPD (melhor interesse da criança, consentimento dos pais).
- [ ] Biometria, saúde, GPS em tempo real → criptografia em repouso + pseudonimização em analytics.
- [ ] Acesso restrito por *tenant* + *row-level security* no DW.
- [ ] Retenção mínima necessária; *hard delete* após término de contrato.

---

## 5. Camada C4 — Verificação Dinâmica e Operacional

### 5.1 OWASP ZAP (DAST)

- [ ] *Baseline scan* no pipeline de PR (duração ≤ 5 min).
- [ ] *Full scan* semanal em ambiente de *staging*.
- [ ] Regras customizadas para endpoints autenticados (auth context configurado).
- [ ] *False positive triage* documentado; *findings* com SLA de correção por severidade.

### 5.2 GitHub Actions — pipeline-padrão

Fluxo mínimo exigido em `/.github/workflows/security.yml`:

1. `checkout` → `setup-language` → `lint`
2. `secret-scan` (Gitleaks ou GitHub Secret Scanning)
3. `sast` (CodeQL + *language-specific linters*)
4. `dependency-scan` (Dependabot / OSV-Scanner / Snyk)
5. `iac-scan` (Checkov / tfsec) — se houver Terraform/CloudFormation
6. `license-scan` (FOSSA / ort) — bloqueia licenças copyleft sem aprovação jurídica
7. `build` + `sbom` (Syft / CycloneDX)
8. `sign` (cosign/Sigstore)
9. `dast` (OWASP ZAP baseline — *nightly build*)
10. `deploy-gate` (bloqueia merge/deploy se qualquer *step* de severidade `critical`/`high` falhar)

### 5.3 Code Review com IA + Code Navigation

- [ ] Toda PR recebe *first-pass* automatizado com assistente IA cobrindo as 4 camadas.
- [ ] Revisão humana (≥ 2 revisores sendo ao menos 1 sênior) **obrigatória e não-substituível** após o *first-pass*.
- [ ] *Code search* / *Code navigation* usado pelo revisor humano para verificar impacto em dependentes (*blast radius*).
- [ ] *Autofix* sugerido nunca aplicado sem entendimento semântico; suspeita de *false fix* = bloqueia merge.

---

## 6. Checklist Consolidado por Eixo Transversal

| Eixo | Perguntas-guia |
|---|---|
| **Identidade** | MFA em todos os acessos? *Least privilege* validado? *Break-glass* monitorado? |
| **Privacidade (LGPD)** | Base legal declarada? DPIA se dado sensível? Retenção mínima? Direitos do titular implementados (acesso, correção, eliminação, portabilidade)? |
| **Integridade** | *Signed commits*? *Artifact signing*? SBOM? Cadeia de suprimento auditável? |
| **Resiliência** | Backup testado? RTO/RPO definidos e validados? IR plan exercitado em *tabletop* nos últimos 12 meses? |
| **Cloud** | SCPs aplicadas? IaC escaneada? *Public resources* auditados? KMS por domínio? |
| **IA / LLM** | *Prompt injection* mitigada? *Tool scopes* mínimos? *Output handling* seguro? *Cost caps*? |
| **Observabilidade** | Logs estruturados, correlacionáveis, com retenção e proteção contra adulteração? |

---

## 7. Critério de Promoção (Deploy Gate)

```
APROVADO para produção quando:
  C1 ∧ C2 ∧ C3 ∧ C4 = TRUE

BLOQUEADO quando qualquer um:
  - CodeQL com alerta severidade ≥ HIGH aberto
  - Secret Scanning com hit não revogado
  - Dependency com CVE CVSS ≥ 7.0 sem exceção formal
  - ZAP com alerta HIGH confirmado em endpoint autenticado
  - DPIA exigida e ausente (dado sensível/menor)
  - MFA ausente em conta com acesso à mudança
  - Mudança sem ticket ITIL aprovado e plano de rollback
```

---

## 8. Matriz Resumida Framework → Camada → Controle

| Framework | C1 | C2 | C3 | C4 |
|---|---|---|---|---|
| ISO/IEC 27001 | ✅ Política, RoPA, classificação | ✅ A.8 Controles técnicos | ✅ A.8 Cloud | ✅ A.8 Monitoramento |
| ISO/IEC 20000-1 | ✅ Gestão de Serviço, SLA | — | — | ✅ Gestão de Eventos |
| COBIT 2019 | ✅ EDM/APO/DSS | ✅ BAI03/BAI10 | ✅ APO13 | ✅ DSS05 |
| ITIL v4 | ✅ Change Enablement | — | ✅ Service Value System | ✅ Monitor & Event Mgmt |
| LGPD | ✅ Art. 6º, 14, 37, 38 | ✅ Art. 6º VII (Segurança) | ✅ Art. 6º VII | ✅ Art. 48 (Incidentes) |
| MFA / FIDO2 | — | ✅ Autenticação forte | ✅ Cloud console | — |
| OWASP Top 10 (2021) | — | ✅ Integral | — | — |
| OWASP ASVS v4 | — | ✅ Níveis 2/3 | — | — |
| OWASP ZAP | — | — | — | ✅ DAST |
| OWASP Top 10 LLM 2025 | — | — | ✅ Integral | — |
| AWS SMM | — | — | ✅ Phase 3+ | — |
| AWS CAF (Security) | ✅ Directive | ✅ Preventive (AppSec) | ✅ Preventive (Data/Infra) | ✅ Detective/Responsive |
| GitHub Code Scanning (CodeQL) | — | ✅ SAST | — | — |
| Code Scanning Autofix | — | ✅ Remediação | — | — |
| GitHub Actions | — | ✅ CI | ✅ IaC scan | ✅ CD + DAST |
| Code Review com IA | ✅ Check de compliance | ✅ Check de padrão AppSec | ✅ Check de IA/Cloud | — |
| Code Navigation / Search | — | ✅ Blast radius | — | — |

---

## 9. Operacionalização

1. **Adoção:** incorporar este template como `PULL_REQUEST_TEMPLATE.md` ou `SECURITY_REVIEW.md` no repositório.
2. **Automação:** cada item marcado acima deve ter, preferencialmente, um *check* automatizado no CI.
3. **Governança:** desvios exigem *risk acceptance* formal assinado por *business owner* + *security owner*.
4. **Métrica:** publicar mensalmente no dashboard de governança (Grafana/Security Hub) a taxa de conformidade por repositório e por camada.
5. **Revisão do template:** semestral, ou a cada publicação relevante (ex.: nova versão do OWASP Top 10 ou do ASVS).

---

*Documento elaborado por Carlos Resende — Head of IT, em 24/04/2026, como padrão de Security Gate aplicável a ambientes corporativos de alta criticidade (SAF de elite, plataformas SaaS multi-tenant, MDP).*
