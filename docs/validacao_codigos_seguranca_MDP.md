# Validação MDP — Código e Segurança com Dado Sensível de Atleta Menor (Art. 14 LGPD)

> **Escopo:** extensão do template-base `validacao_codigos_seguranca.md` v1.0, específica para a **Maestrinho Data Platform (MDP)** — plataforma SaaS multi-tenant que trata dados biométricos, de saúde, performance esportiva e geolocalização de **atletas menores de 18 anos** (categoria Sub 9 a Sub 15).
> **Quando aplicar:** toda alteração de código, modelo, pipeline de dados, IaC ou prompt que tangencie os Módulos 1 (Identidade), 2 (Biometria/InBody), 3 (Performance/RLP), 6 (Wellness/Body Map) ou 7 (Marketplace) da MDP.
> **Base legal dominante:** LGPD **Art. 14** (tratamento de dados pessoais de crianças e adolescentes) — sobrepõe-se a qualquer otimização técnica.
> **Owner:** Carlos Resende — Head of IT + Michelle Resende (Diretora de Desenvolvimento Humano como *co-titular* de consentimento parental).
> **Versão:** v1.0 — 24/04/2026.

---

## 1. Contexto Regulatório Específico — Art. 14 LGPD

### 1.1 Texto legal vinculante

> *"O tratamento de dados pessoais de crianças e adolescentes deverá ser realizado em seu melhor interesse, nos termos deste artigo e da legislação pertinente."* — LGPD Art. 14, *caput*.
>
> *"§ 1º — O tratamento de dados pessoais de crianças deverá ser realizado com o consentimento específico e em destaque dado por pelo menos um dos pais ou pelo responsável legal."*

### 1.2 Desdobramento operacional para a MDP

| Requisito Art. 14 | Implementação obrigatória na MDP |
|---|---|
| **Melhor interesse da criança** (*caput*) | Conselho consultivo familiar assina cada expansão de escopo; veto ético do Módulo 6 (Wellness) sempre que houver conflito entre performance e bem-estar. |
| **Consentimento específico + em destaque** (§1º) | Fluxo de *onboarding* com aceite granular por **finalidade** (não um checkbox guarda-chuva); consentimento dos **dois** pais quando aplicável; trilha de auditoria imutável. |
| **§2º — Publicidade** | Nenhum dado do atleta é usado para publicidade de terceiros, sem exceção. |
| **§3º — Informações para obter consentimento** | Linguagem adaptada (dupla camada: resumo acessível + texto técnico); ícones; vídeo explicativo de 60s. |
| **§5º — Acesso aos dados pelos responsáveis** | Portal parental 100% do escopo do titular; exportação em formato estruturado (JSON + PDF). |
| **§6º — Informações não-excessivas** | *Data minimization* reforçada: coletar só o que o modelo pedagógico-clínico exige; revisar a cada 90 dias. |

---

## 2. Classificação de Dados MDP (obrigatória em cada artefato)

| Classe | Exemplos MDP | Nível de proteção |
|---|---|---|
| **L0 — Público** | Nome artístico ("Maestrinho"), posição, clube atual (após autorização formal) | TLS + log de acesso |
| **L1 — Interno** | Agenda de treinos, nomes de treinadores, convocações | TLS + auth |
| **L2 — Confidencial** | Vídeos de jogos (uso pedagógico interno), scouting, peneiras | TLS + MFA + RBAC |
| **L3 — Restrito** | InBody (composição corporal), VO2máx, sprint times, saltos verticais, RLP PKZ LAB, histórico de lesões | TLS 1.3 + MFA + RBAC + **criptografia em repouso com CMK dedicada** |
| **L4 — Dado Pessoal Sensível de Menor (Art. 14 + Art. 11)** | Saúde, biometria, geolocalização, wellness (humor/sono), imagem facial, diário socioemocional | **Todos os controles L3 + DPIA anual + pseudonimização em analytics + *zero retention* pós-contrato + consentimento parental específico + veto clínico** |

**Regra bloqueante:** qualquer PR que manipule dado classificado como L4 sem DPIA válida = `BLOCKED`.

---

## 3. Controles C1–C4 com Ajustes Específicos para MDP

### 3.1 Camada C1 — Governança (extensão)

- [ ] **DPIA ativa e vigente** (< 12 meses) para qualquer módulo tocado (exigência de fato para L4, embora a ANPD ainda não tenha exigido formalmente).
- [ ] **RoPA** atualizado com a nova finalidade; se a finalidade muda, novo consentimento parental é obrigatório.
- [ ] **Co-titularidade de consentimento** (Carlos e Michelle Resende para o caso piloto Lucas): ambos devem assinar termos de aceite estruturais.
- [ ] **Política de Privacidade infanto-juvenil** em linguagem acessível ao adolescente a partir dos 12 anos + versão completa para os responsáveis.
- [ ] **Conselho consultivo familiar** citado em commits de grande escopo (novas features, integrações com clubes ou SAF).
- [ ] **Conflito de interesse declarado**: Carlos é Head of IT em SAF Botafogo; Lucas é atleta do Arouca Barra. Toda nova integração com clube da base listada no Tier 0 (Flamengo, Palmeiras, Fluminense, Vasco, Botafogo, São Paulo, Atlético-MG, Cruzeiro, Bahia, Grêmio) exige nota no ADR e aprovação do conselho familiar.
- [ ] **ITIL — Change Enablement**: janela de manutenção *never* durante treino, jogo oficial, peneira ou viagem do atleta.

### 3.2 Camada C2 — AppSec (extensão)

- [ ] **ASVS Nível 3** obrigatório para módulos que tratam L4 (autenticação, criptografia, access control, logging).
- [ ] **MFA FIDO2/WebAuthn** em 100% dos acessos administrativos e em 100% dos acessos dos responsáveis ao portal parental. TOTP aceito apenas como *fallback*.
- [ ] **Segregação de funções:** desenvolvedor nunca acessa dado de produção em claro; acesso via *break-glass* com janela + motivo registrados.
- [ ] **Criptografia:** AES-256-GCM em repouso, TLS 1.3 em trânsito, KMS com CMK **dedicada por tenant** (não compartilhada), rotação automática a cada 90 dias.
- [ ] **Pseudonimização em analytics:** `athlete_id` hasheado com *salt* distinto por propósito; o *join* com identidade só existe no cofre de produção.
- [ ] **Proibição absoluta de logs contendo L4 em claro**: scrubbing obrigatório no pipeline de log; violação = incidente reportável.
- [ ] **Retenção:** L4 apagado em *hard delete* 30 dias após término de contrato ou saída do atleta da categoria-fim.

### 3.3 Camada C3 — IA, Dados e Cloud (extensão)

#### 3.3.1 OWASP LLM Top 10 (2025) — MDP-specific
- **LLM01 Prompt Injection:** o assistente pedagógico conversa com atleta menor. *System prompt* com guarda-chuva de segurança + *allowlist* de tópicos + *refusal* default para qualquer conteúdo fora de formação esportiva, nutrição aprovada ou bem-estar. Input do atleta jamais é usado para *retrieval* cross-tenant.
- **LLM02 Sensitive Disclosure:** *output filter* testado para não retornar dado clínico a usuário não-autorizado. *Red team* trimestral focado em *jailbreak* com contexto infantil.
- **LLM05 Improper Output Handling:** resposta do LLM nunca executada como código, nunca renderizada como HTML sem *sanitization*, nunca enviada a endpoint externo sem confirmação humana (pais/treinador).
- **LLM06 Excessive Agency:** *tools* do assistente limitados a consulta (ler agenda, estatística do Módulo 6); ações que afetam agenda, nutrição ou comunicação com clube = *human-in-the-loop* obrigatório.
- **LLM09 Misinformation / Overreliance:** *disclaimer* permanente: *"estas sugestões são orientações gerais e não substituem profissionais (nutricionista Dra. Rita Cadiz, preparador CETRAF, psicopedagoga)"*.
- **LLM10 Unbounded Consumption:** *rate limit* agressivo por tenant + *token budget* mensal publicado para os pais.

#### 3.3.2 Modelos e fine-tuning
- [ ] Dados de Lucas **nunca** são usados para treino de modelo base ou fine-tune compartilhado.
- [ ] *Opt-in explícito* para uso de dados anonimizados em benchmarks públicos.
- [ ] Procedência dos modelos externos (OpenAI/Anthropic/Google) documentada; contratos com cláusula de *zero-retention* verificada a cada revisão.

#### 3.3.3 Cloud (AWS SMM Phase 3+ / CAF)
- [ ] Organização AWS com **conta dedicada por domínio** (Identity, Data-Lake, ML, Ops).
- [ ] SCPs negando: `s3:PutBucketPublicAccessBlock=false`, `kms:ScheduleKeyDeletion`, criação de recursos fora da região `sa-east-1`.
- [ ] Macie ativo em todos os buckets que possam receber L4.
- [ ] CloudTrail multi-region + *log file validation* + imutabilidade via S3 Object Lock (*compliance mode*).
- [ ] GuardDuty + Security Hub + Config com *conformance packs* PCI/CIS/HIPAA adaptados para LGPD-saúde-menor.
- [ ] IAM: acesso a L4 apenas via *Just-in-Time* (AWS SSO + permissão temporária de 4h).

### 3.4 Camada C4 — Verificação Dinâmica (extensão)

- [ ] **DAST autenticado** com usuário de teste Atleta, Responsável, Treinador, Admin — cobrindo autorização cross-tenant e cross-role.
- [ ] **Testes de regressão de privacidade:** suíte que valida que `/api/athletes/{id}` não retorna campo L4 para role < Responsável.
- [ ] **Canary deploys** para mudanças em Módulo 6 (Wellness) — 5% → 25% → 100% com janela de 24h e *auto-rollback* se erro 5xx > 1% ou latência p95 > 300ms.
- [ ] **Tabletop exercise de IR com cenário-Art.14** a cada 6 meses: simular vazamento de InBody de atleta e cronometrar *time-to-notify ANPD* (meta ≤ 72h) + responsáveis (meta ≤ 24h).

---

## 4. Checklist de Aprovação PR MDP (versão reduzida)

```
# Antes do merge — responder SIM a todos para não bloquear

[ ] DPIA vigente referenciada no PR?
[ ] RoPA atualizado?
[ ] Classificação do dado declarada no cabeçalho (L0–L4)?
[ ] Se L4: consentimento parental específico existe? Se mudou de finalidade, renovado?
[ ] ASVS Nível 3 satisfeito em módulos L4?
[ ] MFA FIDO2 nos caminhos admin/responsáveis?
[ ] Logs NÃO contêm L4 em claro? (verificado em scrubbing de teste)
[ ] Modelo/prompt LLM passa pelos 10 itens do OWASP LLM 2025?
[ ] IaC Checkov verde + SCPs respeitadas?
[ ] Conflito de interesse (SAF Botafogo) cabe e foi declarado?
[ ] Janela de deploy fora de treino/jogo/viagem do atleta?
[ ] Owner técnico + DPO + co-titular de consentimento cientes?
```

---

## 5. Matriz Regulatória Resumida — MDP

| Lei / Norma | Artigo / Cláusula | Controle MDP |
|---|---|---|
| LGPD Art. 7º | Bases legais | Consentimento (Art. 7º I) + tutela do menor (via Art. 14) |
| LGPD Art. 11 | Dados sensíveis | Consentimento específico e em destaque |
| LGPD Art. 14 | Menor de idade | **Consentimento parental específico, melhor interesse, sem publicidade**, linguagem acessível |
| LGPD Art. 18 | Direitos do titular | Portal parental com: confirmação, acesso, correção, anonimização, portabilidade, eliminação, revogação |
| LGPD Art. 37 | RoPA | Atualizado a cada nova finalidade |
| LGPD Art. 38 | DPIA | Exigida para L4 (MDP trata como obrigatória) |
| LGPD Art. 46–49 | Segurança e incidentes | Controles C2-C4 + IR plan com cenário-Art.14 |
| Marco Civil (L.12.965) | Art. 7º-X | Exclusão definitiva a pedido |
| ECA (L.8.069) | Art. 17, 18 | Dignidade, respeito à intimidade |
| Res. ANPD 2/2022 | Agente pequeno porte | Avaliar elegibilidade; não reduz exigência Art. 14 |
| OWASP Top 10 (2021) | A01–A10 | Template-base C2 |
| OWASP LLM Top 10 (2025) | LLM01–LLM10 | Template-base C3 + guardrails específicos para conversa com menor |
| ISO 27001:2022 | A.5.34, A.8.11 | Privacidade + mascaramento de dado |
| AWS SMM | Phase 3+ | CMK, GuardDuty, Macie, Object Lock |

---

## 6. Gate de Deploy MDP (estrito)

```
APROVADO quando:
  template-base (C1 ∧ C2 ∧ C3 ∧ C4) = TRUE
  E
  MDP-specific:
    DPIA vigente
    consent_granular_valid
    no_L4_in_logs
    llm_guardrails_passed
    conflict_of_interest_disclosed
    deploy_window_respects_athlete_schedule

BLOQUEADO em qualquer uma:
  - Uso de dado L4 para publicidade, inferência comercial ou cross-tenant
  - Qualquer log com L4 em claro em qualquer ambiente
  - Modelo LLM tratando menor sem guardrail testado
  - Integração com SAF Botafogo (empregador do Head of IT) sem nota de conflito
  - MFA FIDO2 ausente em acesso administrativo ou de responsável
  - Janela de deploy coincidente com treino/jogo/peneira/viagem
```

---

## 7. Observações Finais

1. **Este documento complementa, não substitui,** o template-base `validacao_codigos_seguranca.md` v1.0.
2. A régua Art. 14 é **propositalmente mais rígida do que o mínimo legal atual**, antecipando jurisprudência e guidelines futuras da ANPD sobre dados de crianças.
3. Revisão obrigatória a cada nova resolução ANPD, atualização do OWASP LLM Top 10 ou mudança substancial de modelo de IA adotado.
4. A responsabilidade final pela aprovação de mudanças em dados L4 é **compartilhada e não-delegável** entre Carlos (técnico), Michelle (humano-socioemocional) e DPO designado.

---

*Extensão MDP redigida por Carlos Resende (Head of IT) e Michelle Resende (Diretora de Desenvolvimento Humano) em 24/04/2026, consistente com o Anexo VI da tese "Lucas Maestrinho" e com o Art. 14 da LGPD.*
