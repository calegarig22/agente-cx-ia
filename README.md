# agente-cx-ia

Design e engenharia de um agente de atendimento 24/7 no WhatsApp (n8n + Claude) que resolve a maior parte dos atendimentos e escala para humanos com resumo pronto.

> Versão de portfólio anonimizada de uma ferramenta desenvolvida para uma escola de idiomas; dados e marcas reais removidos.

## O que faz
- Atende alunos e leads no WhatsApp como a persona **Marina**: dúvidas de FAQ, acesso/login, agendamento e cancelamento de aula, segunda via de pagamento, troubleshooting técnico.
- Escala para a fila certa (CX ou Comercial) o que exige humano: contrato, pagamento, preço, reclamação grave, LGPD, cálculos.
- Faz handoff silencioso (o humano assume a mesma conversa) e notifica o time no Slack na hora.

## Destaques técnicos
- **System prompt com saída JSON estruturada** (`acao`, `motivo_escalonamento`, `resumo_para_humano`, `sugestao_resposta`, `dados_aula`, `confianca`) que dirige um roteador no n8n.
- **Escalonamento em camadas:** filtro de escopo → debounce de mensagens fragmentadas (8–12s) + histórico → filtro de palavras-chave de alta certeza → julgamento da IA (com `confianca < 0.6` forçando escalonamento).
- **Guardrails:** anti-alucinação, desambiguação dos "3 tipos de cancelar", verificação de identidade em 2 fatores por nível de risco, lista negra (preço, promessas, dados de terceiros).
- **Golden set** de 25 casos como rede de regressão a cada mudança de prompt/base.
- **Flywheel de aprendizado:** log de conversas, par rascunho × resposta humana como gabarito, graduação de autonomia por intent (≥90% aceito sem edição).
- **Rollout em fases** do copiloto supervisionado à autonomia graduada; fluxo proativo de cobrança respeitando a janela de 24h do WhatsApp.

## Stack
n8n · Claude (Anthropic API) · WhatsApp Business via CRM · Slack · Postgres/Supabase (log) · integrações REST com gateway de pagamentos, portal de agendamentos e LMS.

## Como rodar
Este repositório é a especificação do agente (não contém o export do fluxo n8n):
1. Siga `build-n8n.md` para montar o fluxo da Fase A no n8n.
2. Cole `prompt-agente-cx.md` (saída JSON) ou `system-message-marina.md` (texto, para demo) no nó de IA.
3. Preencha os placeholders (`{{SENHA_TEMPORARIA}}`, `<CRM_API_BASE_URL>`, `<URL_PORTAL_AGENDAMENTOS>` etc.) com valores do seu ambiente, guardando tokens como credenciais do n8n.
4. Valide com os 25 casos de `golden-set.md` antes de liberar.

## Estrutura
- `00-INDICE.md` — índice da documentação
- `prompt-agente-cx.md`, `system-message-marina.md` — cérebro do agente
- `persona-seguranca-guardrails.md`, `habilidades-de-conversa.md`, `troubleshooting-tecnico.md` — persona e conduta
- `regras-escalonamento.md` — regras de decisão
- `plano-tecnico-n8n.md`, `build-n8n.md` — arquitetura e build
- `golden-set.md`, `checklist-go-live.md`, `aprendizado-e-evolucao.md` — qualidade e evolução

## Direitos autorais

© 2026 Gustavo Fiedler Calegari. Todos os direitos reservados.

Este código está publicado apenas para avaliação de portfólio. Não é permitido copiar, modificar, distribuir ou usar comercialmente, no todo ou em parte, sem autorização por escrito do autor.
