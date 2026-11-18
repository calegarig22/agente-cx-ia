# Build n8n — receita de construção (Fase A: MVP copiloto)

> Passo a passo concreto para o dev montar o fluxo da Marina no n8n e **colocar pra rodar**. Escopo: **atendimento PASSIVO** (aluno inicia → dentro da janela 24h → sem template Meta). Referências: `prompt-agente-cx.md` (system prompt), `regras-escalonamento.md`.

## Pré-requisitos (sem isto não roda)
1. **n8n** ativo (cloud ou self-hosted com URL pública para receber webhook).
2. **CRM plano Elite** → **API Token** (Conta → API → Copiar). Base: `<CRM_API_BASE_URL>`, header `api-token` (guardar como credencial do n8n, nunca no fluxo).
3. **Conexão WhatsApp Oficial** no CRM (os endpoints `/v2/messages` são Beta e só funcionam nela).
4. **Webhook de "mensagem recebida"** do CRM apontando para o n8n (confirmar com o CRM que existe no módulo de atendimento Oficial). ← **risco nº1**; se não existir, falar com suporte CRM.
5. **Slack** (webhook/app) com canais **CX** e **Comercial**.
6. **Banco** (Postgres/Supabase) para o log de conversas.
7. Credencial de **IA**: Claude (API Anthropic) — modelo recomendado **claude-opus-4-8** (ou sonnet-4-6 para custo menor).

## Nós do fluxo (ordem)
**1) Webhook (Trigger)** — recebe a mensagem do CRM.
   - Extrai: `contato_id`, `chat_id`, `channel_account`, `owner` (responsável), `texto`, `tipo` (texto/áudio/imagem), `mídia_url`.

**2) Filtro de ESCOPO (IF/Switch)** — *só passa se as DUAS condições baterem:*
   - `channel_account` == canal **WhatsApp Oficial - Customer Care**.
   - `owner` **!=** "Atendente de outro produto".
   - Senão → **encerra** (não faz nada). *(ver `regras-escalonamento.md` Camada −1)*

**3) Debounce / buffer (8–12s por `contato_id`)** — junta mensagens em rajada num único texto. (Nó Wait + chave por contato, ou Redis/Supabase com janela.)

**4) Mídia (se houver)** — **áudio** → transcrição (Whisper/API); **imagem** → visão (descrição/OCR). Concatena o resultado ao texto.

**5) Carregar HISTÓRICO** — `GET /v2/messages/chat/{chat_id}` (CRM) → últimas N trocas.

**6) Enriquecer CONTEXTO** — busca o perfil do aluno por e-mail/telefone:
   - CRM (`GET /v2/chats/contact/{contact_id}` / contato), gateway de pagamentos (status financeiro), Portal/LMS (aula de hoje, módulo, nº de aulas) — *o que estiver integrado nesta fase; o resto entra nas Fases B–D.*

**7) Filtro de palavra-chave (Camada 1, opcional mas recomendado)** — gatilhos de alta certeza (contrato/pagamento/reclamação/preço/produto fora de escopo/ligação) → marca para escalar direto (economiza IA). *(ver `regras-escalonamento.md` Camada 1)*

**8) Nó de IA (Claude)** — system prompt = `prompt-agente-cx.md`; injeta **base de conhecimento** + **histórico** + **contexto do aluno** + a **mensagem agregada**. Saída = **JSON** (forçar JSON).
   - **Base a injetar** (concatenar/ou RAG): FAQ, termos e condições, catálogo, ciclo de vida, onboarding/acesso, mensagens-padrão, playbook de atendimento e `troubleshooting-tecnico.md`. *(Na versão de portfólio só o troubleshooting é incluído; os demais eram documentos internos da operação.)*
   - Dica de custo/latência: começar injetando a base inteira (simples); migrar para **RAG** (busca semântica) quando crescer.

**9) Parse do JSON** — `{acao, mensagem, motivo_escalonamento, resumo_para_humano, sugestao_resposta, dados_aula, tags, confianca}`.

**10) Roteador por `acao` (Switch):**
   - **responder / encerrar** → `POST /v2/messages/text` (CRM) com `mensagem` + grava Nota da tratativa (`POST /v2/messages/note`).
   - **registrar_aula** → grava `dados_aula` (fila da agenda / Nota) + `POST /v2/messages/text` (confirma que registrou).
   - **escalar** → `POST /v2/messages/text` (handoff: silencioso no horário 7h–20h / "encaminho à equipe" fora) + `POST /v2/messages/note` (resumo + sugestao) + mover negócio p/ etapa **"Atendimento Humano"** + aplicar **tag** do motivo + **notificar Slack NA HORA** (canal CX, ou Comercial se motivo=comercial/renovacao→dono do contrato).

**11) LOG** — grava no banco: entrada, JSON de saída, ação final, e (nas escaladas) a resposta humana depois = **gabarito** do flywheel. *(ver `aprendizado-e-evolucao.md`)*

## Modo COPILOTO (Fase A — segurança)
- Começar com **draft + aprovação humana** antes de enviar (ou auto-responder só intents de baixo risco: acesso, "como agendo", prazos, "como funciona", troubleshooting simples).
- Validar contra o **`golden-set.md`** (25 casos) antes de soltar.
- Graduar intent por intent quando o rascunho for aceito sem edição consistentemente.

## Endpoints CRM (resumo)
`POST /v2/messages/text|image|document|audio|template` · `POST /v2/messages/note` · `GET /v2/messages/chat/{id}` · `GET /v2/chats/contact/{id}` · `GET /v2/chats/{id}`. Auth: header `api-token`.

## Fluxo extra — Provisionamento de acesso (planilha de nivelamento → 2 plataformas)
**Gatilho:** novo aluno aprovado na **planilha de nivelamento** (Google Sheets) — linha com **Status (col G)=Agree** e **Level (col I)** preenchido.
**Passos n8n:**
1. **Trigger Google Sheets** (n8n "Google Sheets Trigger" — row added/updated) ou cron + leitura da aba, filtrando linhas novas com G=Agree e I≠vazio (ignorar "Refazer"/vazio).
2. **Montar payload do aluno:** Nome (col D), E-mail (col E), Level/módulo (col I), senha padrão {{SENHA_TEMPORARIA}}.
3. **Criar no LMS:** "Add User" (painel admin do LMS) com Group = Level — via API/import do LMS se houver, ou RPA (browser-use) se não.
4. **Criar no Portal de Agendamentos:** "Novo aluno" com Nome/E-mail/matrícula no módulo do Level — **Portal não tem API pública** → RPA/browser ou API interna (confirmar com dev).
5. **Enviar credenciais ao aluno** (CRM `POST /v2/messages/text`): 2 links + login (e-mail) + senha {{SENHA_TEMPORARIA}}; marcar **Envio** na planilha.
> Modo copiloto: gerar a **lista/rascunho** dos alunos a provisionar para um humano criar, até a escrita estar integrada e validada. Criar usuário é ação de escrita; **excluir é grave → nunca automatizar sem trava**.

## Teste antes de abrir pro público
1. Apontar o webhook para um **número/ambiente de teste** (ou um contato controlado).
2. Rodar os 25 casos do `golden-set.md`.
3. Conferir: filtro de escopo (ignora outros canais e o atendente de outro produto), handoff silencioso, Slack, log gravando.
4. Subir em copiloto com volume limitado → graduar.
