# Política de Segurança — MDP Security Gate Kit

## Versões suportadas

| Versão | Suporte |
|--------|---------|
| 1.0.x  | ✅ Suporte ativo com patches de segurança |
| < 1.0  | ❌ Não suportado |

## Como reportar uma vulnerabilidade

**Não abra issue pública** para vulnerabilidades de segurança. Use um dos canais privados
abaixo, na ordem de preferência:

1. **GitHub Private Vulnerability Reporting** (preferencial)
   - Acesse: `https://github.com/carlosresendeitsm/MDP/security/advisories/new`
   - Anexe PoC, impacto estimado e sugestão de remediação.

2. **E-mail cifrado**
   - Destinatário: `Carlos.resende@safbfr.com.br`
   - Use PGP se tiver — chave disponível sob demanda.
   - Assunto: `[MDP-SECURITY] <resumo curto>`.

3. **Formulário de Titular (LGPD)** (apenas quando o reporte envolver dado pessoal do
   titular ou do seu dependente menor)
   - Portal: `https://safbfr.com.br/lgpd` (canal oficial do Controlador).

## O que esperar

| Etapa | SLA |
|-------|-----|
| Primeiro aceite do reporte | ≤ 2 dias úteis |
| Triagem e classificação (CVSS + LGPD impact) | ≤ 5 dias úteis |
| Plano de remediação | ≤ 10 dias úteis para HIGH/CRITICAL |
| Patch publicado | ≤ 30 dias corridos para HIGH/CRITICAL; ≤ 90 dias para MEDIUM |
| Comunicação ao reporter | contínua, com *CVE/GHSA ID* quando aplicável |

## Escopo

### Dentro do escopo
- Pipeline `.github/workflows/security.yml`
- Prompt e política do auditor (`CLAUDE.md`, `docs/prompt_auditoria_claude_code.md`)
- Templates de PR e *issue*
- Exemplos e documentação que possam induzir uso inseguro

### Fora do escopo
- Dependências *upstream* (GitHub Actions, LibreOffice, Python libs) — reporte
  diretamente aos mantenedores e, em paralelo, abra *issue* aqui se afetar a configuração
  fornecida.
- Engenharia social contra mantenedores.
- Ataques físicos ou negação de serviço volumétrica contra infraestrutura da SAF
  Botafogo ou do Controlador.
- Achados baseados apenas em *version banner* sem PoC.

## Classificação de severidade

Usamos CVSS v3.1 como base e aplicamos um *multiplicador LGPD*:

| Severidade | CVSS base | Ajuste LGPD Art. 14 (menor) |
|------------|-----------|-----------------------------|
| CRITICAL   | ≥ 9.0     | sempre CRITICAL se afeta dado L4 |
| HIGH       | 7.0–8.9   | promove para CRITICAL se afeta L4 |
| MEDIUM     | 4.0–6.9   | promove para HIGH se afeta L3/L4 |
| LOW        | < 4.0     | promove para MEDIUM se afeta L2+ |

## Safe Harbor

Pesquisadores que seguirem esta política **não sofrerão ação legal** por atividade de
pesquisa em segurança de boa-fé, desde que:
- Não acessem, modifiquem ou exfiltrem dados de terceiros (especialmente dados de
  atletas menores — L4).
- Não degradem o serviço (sem DoS, *brute force* em produção, *data exfiltration*).
- Reportem em canal privado antes de divulgar publicamente.
- Concedam prazo razoável de remediação (90 dias padrão).

## Reconhecimento

Mantemos um *Hall of Fame* em `docs/security_hall_of_fame.md` (criado no primeiro
reporte válido). A inclusão é opcional e depende do consentimento expresso do reporter.

---

*Política v1.0 — vigente desde 2026-04-24. Responsável: Head of IT SAF Botafogo.*
