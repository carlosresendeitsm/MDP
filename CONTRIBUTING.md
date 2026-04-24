# Como contribuir com o MDP Security Gate Kit

Obrigado pelo interesse em contribuir. Este repositório abriga a *baseline* de
segurança usada pela SAF Botafogo e pela plataforma MDP (Maestrinho Data Platform).
Como tocamos em pipeline de segurança e em dado sensível de atleta menor, todas as
contribuições passam por revisão dupla.

## Código de conduta

Trate revisores e mantenedores com respeito. Contribuições que exponham dado de
terceiros, ofereçam explorações contra sistemas de produção ou violem a LGPD serão
rejeitadas e podem resultar em banimento permanente do repositório.

## Ambiente de desenvolvimento

Pré-requisitos (macOS):
```bash
brew install git gh pre-commit gitleaks checkov
npm install -g @cyclonedx/cdxgen
python3 -m pip install --upgrade pip osv-scanner
```

Ative os hooks locais:
```bash
pre-commit install
pre-commit run --all-files   # smoke test antes do primeiro commit
```

## Fluxo de PR

1. **Abra uma issue primeiro** (exceto para correção trivial de typo ou doc). Alinhar o
   escopo previne retrabalho.
2. **Crie uma branch** a partir de `main` seguindo a convenção:
   ```
   feat/<slug>    → nova funcionalidade
   fix/<slug>     → correção de bug
   docs/<slug>    → documentação
   chore/<slug>   → manutenção (deps, CI, etc.)
   security/<slug>→ correção de segurança (PR sempre privada, ver SECURITY.md)
   ```
3. **Commits assinados e convencionais** (Conventional Commits 1.0):
   ```
   <tipo>(<escopo opcional>): <descrição imperativa em minúsculas>

   Corpo opcional explicando o "porquê".

   Closes #<issue>
   Signed-off-by: Seu Nome <seu-email@example.com>
   ```
   Tipos aceitos: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`, `security`.
4. **Preencha o template de PR** integralmente. PR com C1-C4 incompleto é bloqueada
   automaticamente.
5. **Rode os jobs localmente** antes de pedir revisão:
   ```bash
   gitleaks detect --source . --no-banner
   checkov -d . --quiet
   osv-scanner --recursive .
   ```
6. **Aguarde duas aprovações humanas** (uma das quais deve ser `@carlosresendeitsm`
   ou substituto designado). PRs que tocam dado L4 exigem co-titular + DPO.

## Assinatura de commits (recomendado)

```bash
# GPG
gpg --full-generate-key
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true

# ou SSH (macOS, mais simples)
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

## O que NÃO fazer

- ❌ Não commitar `.env`, `secrets/`, `*.pem`, `*.key`, cookies, tokens — o `.gitignore`
  cobre, mas confira com `git status` antes de `git add`.
- ❌ Não usar `git push --force` em `main`. Use `--force-with-lease` em branches
  pessoais se precisar reescrever história.
- ❌ Não pular hooks (`--no-verify`) — eles existem para bloquear secret leak.
- ❌ Não responder *autofix* sugerido pelo Code Scanning sem validar semanticamente.
- ❌ Não usar `console.log`, `print`, `dump` em caminhos que processam dado L3/L4.

## Estrutura do repositório

```
MDP/
├── .github/
│   ├── CODEOWNERS                      # revisores obrigatórios por path
│   ├── ISSUE_TEMPLATE/                 # bug, feature, security, LGPD
│   ├── PULL_REQUEST_TEMPLATE.md        # checklist C1-C4
│   ├── dependabot.yml                  # atualização automática de deps
│   └── workflows/
│       └── security.yml                # pipeline SAST/DAST/IaC/SBOM/DAST/LLM
├── docs/
│   ├── architecture.md                 # arquitetura C1-C4 em diagrama
│   ├── example_audit_output.md         # exemplo de saída do auditor
│   ├── prompt_auditoria_claude_code.md # prompt formal do Claude Code
│   ├── validacao_codigos_seguranca.md  # template base (SAF/geral)
│   └── validacao_codigos_seguranca_MDP.md # overlay LGPD Art. 14
├── CLAUDE.md                           # política ativa do auditor
├── CHANGELOG.md                        # histórico de versões
├── CODEOWNERS → .github/CODEOWNERS
├── CONTRIBUTING.md                     # este arquivo
├── LICENSE                             # MIT
├── README.md
└── SECURITY.md                         # política de disclosure
```

## Perguntas

Abra uma discussion em `github.com/carlosresendeitsm/MDP/discussions` ou contate o
responsável técnico via e-mail institucional. Para dúvidas de LGPD, encaminhe ao
Controlador através do canal oficial do titular.

---

*CONTRIBUTING v1.0 — 2026-04-24.*
