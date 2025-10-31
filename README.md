# advogado-inteligente
**Parte 1** (especificação funcional + tech stack + ferramentas que você precisa instalar no computador). Vou dividir o trabalho em fases (“por partes”), como pediu — já comecei pela primeira parte aqui, para irmos avançando passo a passo.

Citação da referência: a página que servirá de modelo apresenta o fluxo “escolha o modelo → responda perguntas → baixe Word/PDF” e ferramentas extras (calculadoras, artigos). ([99Contratos][1])

---

# Visão geral do produto (objetivo)

Aplicação web que permite a usuários (pessoas físicas e jurídicas) selecionar modelos de contratos revisados por advogados, responder um questionário dinâmico que preenche automaticamente os campos e gerar minutas em Word e PDF. A solução entregue será um **gerador automatizado de minutas** (modelos de referência), obedecendo às suas regras (seguir legislação brasileira, linguagem formal clara, inserir aviso de auto-geração, “[A PREENCHER]” se faltar, incluir cidade/data etc.).

---

# Principais funcionalidades (alto nível)

1. Catálogo de modelos (Prestação de Serviços, Locação, NDA, Compra e Venda, Contrato de Parceria, Contrato de Software, Política de Privacidade, Termo de Uso, etc.).
2. Para cada modelo: questionário dinâmico (pergunta → campo do template).
3. Motor de preenchimento: aplica respostas ao template e gera DOCX e PDF.
4. Preview do documento (HTML + download DOCX/PDF).
5. Histórico do usuário / versões / templates revisados por advogado.
6. Gestão administrativa (painel para advogados: criar/editar templates e questionários, aprovar versões).
7. Pagamento / planos (opcional): downloads pagos ou por assinatura.
8. LGPD & segurança: consentimento para tratamento de dados, logs mínimos, criptografia.
9. Localização de conteúdo: sempre incluir cidade e data fornecidas pelo usuário.
10. Disclaimer automático ao final do documento:
    “⚠️ Minuta gerada automaticamente com base em modelo revisado por advogado. Revisão profissional é recomendada.”

(Fluxo UX idêntico ao site referência: escolha do modelo → responder perguntas → gerar contrato Word/PDF). ([99Contratos][1])

---

# Requisitos legais e regras aplicáveis (resumo)

* Obedecer legislação brasileira conforme tipo de contrato (Código Civil, CLT, Lei do Inquilinato, LGPD, etc.).
* Não emitir parecer jurídico — o produto fornece **modelos de referência** e sinaliza recomendação de revisão por advogado.
* Inserir cláusulas obrigatórias por tipo de contrato — o motor NÃO deve remover cláusulas obrigatórias.
* Padrão de linguagem: clara e formal jurídica.
* Campo “[A PREENCHER]” quando dados faltarem.
  (Implementação: manter templates canônicos que só advogados do time podem alterar; o sistema valida presença de cláusulas obrigatórias ao gerar.)

---

# Arquitetura proposta (resumo técnico)

* Frontend: React (Next.js) — SSR/SSG para SEO do catálogo e fácil deploy (Vercel/Render).
* Backend: Node.js (Express / NestJS) ou Python (FastAPI) — API REST/GraphQL para templates, usuários, geração de documentos.
* Banco de dados: PostgreSQL (relacional) + Redis (cache / sessão opcional).
* Motor de templates: armazenar templates em formato DOCX com marcadores, ou usar templates em Markdown/Jinja e converter para DOCX/PDF. Biblioteca sugerida: **docxtemplater** (Node) ou **python-docx / docxtpl** (Python) + wkhtmltopdf / Puppeteer para PDF.
* Autenticação: JWT + refresh tokens; OAuth2 opcional.
* Storage: S3 (AWS) / DigitalOcean Spaces / Google Cloud Storage para arquivos (DOCX/PDF gerados).
* CI/CD: GitHub Actions / GitLab CI.
* Hospedagem: Vercel (frontend) + Render / Heroku / AWS ECS (backend) + managed Postgres (Heroku Postgres, RDS, Supabase).

---

# Segurança e LGPD (essenciais)

* Consentimento explícito na criação de conta/uso para tratamento de dados.
* Minimização de dados: armazenar apenas o necessário.
* Criptografia em trânsito (HTTPS/TLS) e em repouso para dados sensíveis.
* Logs de auditoria para alterações de templates por advogados.
* Política de retenção e processo de exclusão (direito ao esquecimento).
* Termos e Política de Privacidade acessíveis na UI.
  (Implementar checklist técnico e jurídico antes do lançamento.)

---

# Ferramentas e apps que você precisa ter no computador (instalar agora)

Essenciais (desenvolvimento local):

1. Git (controle de versão).
2. Node.js (LTS) e npm / yarn.
3. VS Code (IDE) + extensões (Prettier, ESLint).
4. Docker (opcional, muito recomendado para replicar ambiente).
5. Postgres local (ou usar um container).
6. wkhtmltopdf / Puppeteer (para geração PDF via HTML) — dependendo da abordagem.
7. Ferramenta de design: Figma (para UI/UX).
8. Conta em provedor cloud (AWS / DigitalOcean / Render / Vercel).
9. Conta em S3-compatible storage (AWS S3 ou DigitalOcean Spaces) para arquivos.
10. OpenSSL (certificados locais) — opcional para testes HTTPS locais.

Ferramentas administrativas / de apoio:

* Google Workspace ou similar (e-mail transacional).
* Stripe / PayPal (pagamentos).
* Sentry (monitoramento de erros).
* Postman / Insomnia (teste de API).

---

# Bibliotecas / stacks recomendadas (detalhado)

Frontend:

* Next.js (React)
* Tailwind CSS (rápido para UI) ou Chakra UI
* react-hook-form (forms dinâmicos)
* i18next (se precisar internacionalização)

Backend:

* Node.js + NestJS (bom para APIs e estrutura modular) **ou** FastAPI (Python)
* TypeORM / Prisma (ORM para Postgres)
* docxtemplater + pizzip (preenchimento DOCX) — Node
* html-pdf / Puppeteer para conversão HTML → PDF (caso o preview seja HTML)
* bcrypt / argon2 para senhas

Document handling:

* Armazenar templates DOCX com placeholders ({{nome}}, {{endereço}}).
* Um “schema de perguntas” JSON por template que define campos, tipos, validações e cláusulas condicionais.

Exemplo de esquema JSON (modelo de pergunta — use como base):

```json
{
  "template_id": "locacao_residencial_v1",
  "title": "Contrato de Locação Residencial",
  "fields": [
    {"id":"cidade","label":"Cidade","type":"text","required":true},
    {"id":"data","label":"Data","type":"date","required":true},
    {"id":"locador_nome","label":"Nome do Locador","type":"text","required":true},
    {"id":"locatario_nome","label":"Nome do Locatário","type":"text","required":true},
    {"id":"valor_aluguel","label":"Valor do Aluguel (R$)","type":"currency","required":true}
  ],
  "clauses_required": ["identificacao","objeto","prazo","valor","rescisao"]
}
```

---

# UX/UI — páginas principais da aplicação

1. Home / catálogo de modelos (SEO) — lista com busca/filtragem.
2. Página do modelo — descrição + botão “Criar contrato”.
3. Questionário dinâmico — etapas com validação (salvar rascunho).
4. Preview (HTML) — mostra minuta preenchida, com botão “Gerar DOCX / PDF”.
5. Downloads / Histórico — lista de documentos gerados.
6. Painel Admin (advogados): criar/editar templates, revisar versões.
7. Área de cobranças / assinaturas.
8. Página de ajuda / artigos / calculadoras (como no site referência). ([99Contratos][1])

---

# Conteúdo obrigatório do documento gerado (por minuta)

* Título centralizado.
* Identificação das partes (com campos completos).
* Cláusulas numeradas.
* Disposições finais.
* Local e data (cidade + data fornecida).
* Assinaturas (linhas para assinaturas e campos de testemunhas, se aplicável).
* Aviso final obrigatório (conforme sua regra):
  “⚠️ Minuta gerada automaticamente com base em modelo revisado por advogado. Revisão profissional é recomendada.”
* Em caso de falta de dado: inserir “[A PREENCHER]” no campo correspondente.

---

# Faseamento (como vamos fazer “por partes”)

Vou propor 6 fases — iremos implementar e validar cada fase antes de passar à próxima.

**Fase 0 — Planejamento / requisitos detalhados** (já iniciada aqui)
**Fase 1 — MVP funcional**

* Catálogo com 6–10 modelos iniciais
* Questionário dinâmico + preenchimento DOCX + geração PDF
* Autenticação básica e armazenamento de arquivos
  **Fase 2 — Painel de advogados (edição/versão de templates)**
  **Fase 3 — Pagamentos / assinaturas / planos**
  **Fase 4 — LGPD completo (auditoria, DPO flow, política de retenção)**
  **Fase 5 — Escala (caching, fila para geração de documentos, alta disponibilidade)**

Você pediu “vamos fazer por parte” — **neste momento eu entreguei a Fase 0 e o roteiro para a Fase 1**. A seguir eu já preparo a **entrega da Parte 1 executável** (o que você deve ter pronto e um checklist de arquivos e estruturas) — veja abaixo.

---

# Parte 1 — Entregável imediato (o que você precisa preparar / estrutura inicial)

## 1) Repositório Git (estrutura básica)

* `/frontend` — Next.js app
* `/backend` — NestJS/Express API
* `/templates` — pasta com templates DOCX e schema JSON por template
* `/docs` — especificações legais, checklist LGPD, políticas

## 2) Modelo de template (DOCX) e esquema JSON

* Crie um DOCX base com placeholders no formato `{{campo_id}}`.
* Para cada template, crie `template_id.json` com o array `fields` (tipo, validação, hint) e `clauses_required`.

## 3) Exemplo de template mínimo (contrato de prestação de serviços)

* `templates/servicos_v1.docx` (com placeholders)
* `templates/servicos_v1.json` (schema perguntas) — usar o JSON de exemplo já mostrado.

## 4) Ambiente local (instalar)

* Git, Node.js LTS, Docker (recomendado), Postgres (container).
* VS Code com extensões.

## 5) Primeiras rotas API (backend)

* `GET /templates` → lista templates (titulo, descrição, id).
* `GET /templates/:id/schema` → retorna JSON schema de perguntas.
* `POST /generate` → recebe `template_id` + respostas → retorna arquivo gerado (DOCX/PDF) ou link.

## 6) Primeiras telas (frontend)

* Página catálogo (lista templates).
* Formulário dinâmico gerado a partir do schema JSON.
* Botão “Gerar” que chama `POST /generate` e oferece download.

## 7) Política de auditoria e revisão

* Todo template novo: campo `approved_by` (advogado) e `approved_at` antes de ser publicado no catálogo.
* Histórico de versões: manter templates antigos para compliance.

---

# Checklist mínimo de conformidade (LGPD + contratos)

* Checkbox de consentimento antes de salvar dados.
* Explicitar finalidade do tratamento.
* Mecanismo para exclusão de conta/dados.
* Política de retenção e backups seguros.
* Revisão das cláusulas obrigatórias por área do direito (advogado do projeto deve validar).

---

# Próximo passo (o que eu vou gerar agora se quiser)

Você pediu “vamos fazer por parte” — já entreguei o planejamento + Parte 1 (roteiro e checklist). Se concordar, eu posso agora **gerar** (nesta conversa, imediatamente):

A. O *esqueleto do JSON schema* e um *exemplo de template DOCX* (em formato textual; para DOCX eu posso gerar o conteúdo em Markdown/Word-ready que você poderá copiar para um .docx).
B. O *end-to-end endpoint* `POST /generate` (pseudocódigo) e o código inicial (Node.js) para preencher DOCX usando `docxtemplater`.
C. Um *modelo de questionário* para 3 contratos iniciais (Prestação de Serviços, Locação Residencial, NDA).

Como você pediu para prosseguirmos por partes, **vou executar o item B e C agora** (fornecer o pseudocódigo + JSON schema para os 3 modelos). Se preferir que eu gere o conteúdo do template DOCX (texto do contrato em linguagem jurídica), diga e eu já produzo a minuta de um dos contratos seguindo suas regras (lembrando que ao gerar o texto do contrato, o texto final deve conter o aviso e usar “[A PREENCHER]” onde faltar).

---

Se quiser que eu já produza **uma minuta de exemplo** (por exemplo, **Contrato de Prestação de Serviços**) com todos os requisitos (título centralizado, identificação, cláusulas numeradas, cidade/data, aviso final), responda apenas com qual modelo você quer primeiro: **Prestação de Serviços**, **Locação**, **NDA**, **Compra e Venda**, **Desenvolvimento de Software**, ou outro da lista. Eu então **gero o texto da minuta imediatamente** seguindo todas as regras que você impôs.

[1]: https://www.99contratos.com.br/index.html "99Contratos - Contratos Personalizados Simples | Modelos Word e PDF para baixar"
