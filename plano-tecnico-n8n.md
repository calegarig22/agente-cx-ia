# Plano Técnico — Como a Marina vai operar (n8n + CRM + sistemas)

> O blueprint para tirar a Marina do papel. Junta tudo: prompt (`prompt-agente-cx.md`), regras (`regras-escalonamento.md`),
> base de conhecimento (FAQ, catálogo, playbook) e integrações (CRM, gateway de pagamentos, portal, LMS, Slack).
> Princípio: **subir em modo copiloto** e **graduar por intent** (ver `aprendizado-e-evolucao.md`).

## 1. Arquitetura do fluxo REATIVO (atendimento) — n8n
```
WhatsApp do aluno → CRM
   │ (webhook "mensagem recebida")
   ▼
[1] n8n recebe → FILTRO DE ESCOPO (descarta se não for da Marina):
       • canal == "WhatsApp Oficial - Customer Care" — só este número
       • owner/responsável da conversa != Atendente de outro produto (outro produto)
       → se falhar qualquer condição, IGNORA (não aciona a Marina)
   ▼
[2] CAMADA 0 — Debounce 8–12s (junta mensagens curtas/fragmentadas) + carrega HISTÓRICO do chat
   ▼
[2.5] MÍDIA — **transcreve áudios** (ex.: Whisper) e **lê imagens** (visão: prints, comprovantes) → vira texto pro prompt
   ▼
[3] CAMADA 1 — Filtro de palavra-chave (rede de segurança barata):
        contrato/pagamento/reclamação grave/preço/produto fora escopo → ESCALA direto (pula a IA)
   ▼
[4] ENRIQUECER CONTEXTO — busca dados do aluno (CRM/gateway de pagamentos/portal/LMS por e-mail/CPF):
        plano, módulo, status, nº de aulas, situação financeira → injeta no prompt
   ▼
[5] CAMADA 2 — Claude (system prompt + base de conhecimento + histórico + contexto) → JSON
   ▼
[6] Parse do JSON {acao, mensagem, motivo, resumo, sugestao, dados_aula, tags, confianca}
   ▼
[7] ROTEADOR por `acao`:
     • responder / encerrar → POST /v2/messages/text (CRM) + grava Nota da tratativa
     • registrar_aula      → grava dados_aula (fila agenda / futuro: API portal) + text + Nota
     • escalar             → text (handoff: silencioso no horário / "encaminho" fora) 
                             + POST /v2/messages/note (resumo+sugestao)
                             + move negócio p/ etapa "Atendimento Humano" + tag(motivo)
                             + NOTIFICA SLACK NA HORA (canal CX ou Comercial)
   ▼
[8] LOG da conversa (Postgres/Supabase) — desde o 1º dia (flywheel)
```
Regras de horário: 7h–20h handoff silencioso; fora, mensagem de "encaminho à equipe" + Slack na hora. (ver `regras-escalonamento.md`)

## 2. Fluxo PROATIVO (cobrança) — cron separado
- Fonte: **gateway de pagamentos** (aba Inadimplentes / `GET /bills?status=pending` vencidas) ou webhooks `charge_rejected`/`bill_created`.
- Cadência: 1º → 2º → 3º contato → desativação (atualiza Status).
- `bill_paid` (webhook) → fecha automático (Status→PAGO). "Já paguei" deixa de exigir comprovante manual na maioria.
- ⚠️ **Coordenar com "Recuperação de vendas" nativo do gateway de pagamentos** para não duplicar contato.
- Respostas do aluno caem no fluxo reativo (promessa→reenvia link; negociação→escala comercial; não reconheço→escala financeiro).
- ⚠️ **Janela de 24h da Meta (crítico):** fora de 24h desde a última mensagem do cliente, o WhatsApp só permite **template aprovado** (não texto livre). Todo contato **proativo** (cobrança, reengajamento, avisos) fora da janela **precisa de template aprovado**. Mapear e aprovar os templates necessários.

## 3. Integrações — estado e dependências
| Sistema | Para quê | Estado | Pendência |
|---|---|---|---|
| **CRM API** (`<CRM_API_BASE_URL>`) | enviar/ler msg, notas, etapas, tags | API existe (Elite) | confirmar **Elite + conexão Oficial + webhook de mensagem recebida** (risco nº1 do reativo) |
| **Slack** | notificar atendente ao escalar | — | criar app/webhook + canais CX e Comercial |
| **Log DB** (Postgres/Supabase) | flywheel | — | criar schema (`aprendizado-e-evolucao.md`) |
| **gateway de pagamentos API** (`<GATEWAY_API_BASE_URL>`) | cobrança, baixa, link | API documentada | gerar **chave de API** + configurar webhooks |
| **LMS/estudos**  | liberar Final/módulo, reset senha | mapeado por prints | **confirmar fornecedor + API** |
| **Portal agendamentos**  | link da aula, estornar, agendar, presenças | **SEM API pública** | **falar com o dev do portal** (API interna?) ou automação — **dependência nº1 para a Marina EXECUTAR** |

## 4. Enriquecimento de contexto (a Marina "olha a ficha")
Chave de cruzamento: **e-mail** (e CPF). Antes de chamar a IA, o n8n busca e injeta: plano/produto, módulo atual, nº de aulas realizadas (portal), status financeiro (gateway de pagamentos), status do cadastro (LMS). Isso transforma respostas genéricas em respostas "olhando o aluno".

## 5. Pré-requisitos / bloqueadores a confirmar (ordem)
1. **CRM:** plano Elite? conexão WhatsApp Oficial? existe webhook de "mensagem recebida"? → destrava todo o reativo.
2. **Slack:** workspace + canais CX/Comercial + webhook.
3. **DB de logs** (e política de retenção — LGPD).
4. **gateway de pagamentos:** chave de API + webhooks.
5. **Portal de agendamentos:** existe API interna? (define o quanto a Marina executa de cara).
6. **LMS:** fornecedor + API.

> ⭐ **Escopo inicial = PASSIVO/reativo apenas** (decisão de negócio, jun/2026). O aluno sempre inicia → tudo dentro da **janela de 24h** → **sem dependência de template da Meta**. O fluxo proativo de cobrança/reengajamento (que exigiria templates) fica para **fase posterior**. Marina reutiliza as **mensagens-padrão** existentes.

## 6. Rollout (do seguro ao autônomo)
- **Fase A — MVP copiloto (sem integrações de execução):** CRM plumbing + log + Claude. A Marina **responde FAQ/info e escala o resto**; humano aprova antes de enviar (ou auto-responde só intents de baixo risco: acesso, "como agendo", prazos, "como funciona"). Já reduz tempo do CX no CRM.
- **Fase B — Cobrança (gateway de pagamentos):** liga o fluxo proativo + confirmação de pagamento.
- **Fase C — Execução no portal de agendamentos:** a maior alavanca de "resolver" ("não consigo entrar" → manda link; estornar; agendar) — depende da API interna.
- **Fase D — LMS:** liberar Final/módulo, reset de senha.
- **Fase E — Contexto + graduação:** enriquecer perfil + graduar intents para auto-resposta (flywheel, métricas).

## 7. Métricas (desde a Fase A)
% resolvido sem humano · escalonamento por categoria · reabertura · CSAT · rascunho aceito sem edição (por intent → critério de graduação).

## Resumo de uma linha
**Fase A (CRM + Claude + log + Slack) já coloca a Marina atendendo em copiloto.** As fases B–D adicionam "mãos" (gateway de pagamentos, portal, LMS) conforme as integrações ficam prontas — sendo o **portal de agendamentos a dependência mais importante** para ela resolver os casos de maior volume.
