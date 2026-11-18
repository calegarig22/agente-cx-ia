# Índice — Base da Marina (agente de CX)

> Ponto de entrada da documentação da **Marina**, assistente de Customer Care da Escola de Idiomas Exemplo
> (atende WhatsApp via CRM, orquestrada em n8n com Claude).
> Versão de portfólio: documentos que eram essencialmente dados operacionais internos (FAQ, catálogo, termos,
> playbook de conversas reais, referências de API de fornecedores, planilhas) foram removidos.

## 1. Cérebro do agente
| Arquivo | O que é |
|---|---|
| `prompt-agente-cx.md` | **System prompt de produção**: identidade, tom, escopo, limites, 3 tipos de "cancelar", verificação de identidade, guardrails, contexto da conversa e **formato JSON de saída** consumido pelo n8n. |
| `system-message-marina.md` | Variante "rigorosa" do system message com saída em texto natural (demo/teste) e base de conhecimento de exemplo. |

## 2. Persona, segurança e conduta
| Arquivo | O que é |
|---|---|
| `persona-seguranca-guardrails.md` | Persona, verificação de identidade em 3 níveis de risco (2 fatores) e lista negra. |
| `habilidades-de-conversa.md` | Como a Marina conversa: mensagens fragmentadas, mídia, desescalar, objeções, idioma, microcopy. |
| `troubleshooting-tecnico.md` | Passo a passo dos erros técnicos mais comuns (navegador, cookies, Zoom, senha). |

## 3. Regras de decisão
| Arquivo | O que é |
|---|---|
| `regras-escalonamento.md` | Camadas (escopo → debounce → palavra-chave → IA), 2 filas (CX/Comercial), handoff silencioso, horário, fluxo de cobrança. |

## 4. Engenharia, qualidade e evolução
| Arquivo | O que é |
|---|---|
| `plano-tecnico-n8n.md` | Arquitetura do fluxo reativo e do proativo (cobrança), integrações e rollout em fases A–E. |
| `build-n8n.md` | Receita de construção nó a nó da Fase A (MVP copiloto). |
| `golden-set.md` | 25 casos de teste com comportamento esperado (regressão). |
| `checklist-go-live.md` | Bloqueadores vs. itens de "força" para começar a atender. |
| `aprendizado-e-evolucao.md` | Flywheel: capturar → gabarito humano → curar → medir → graduar intents. |

## Rollout (resumo)
- **Fase A — MVP copiloto** (CRM + Claude + log + Slack): responde FAQ/info e escala o resto; humano aprova.
- **Fase B — Gateway de pagamentos** (cobrança + baixa automática).
- **Fase C — Portal de agendamentos** (maior alavanca de "resolver").
- **Fase D — LMS** (liberar módulo, reset de senha).
- **Fase E — Contexto + graduação de intents** (flywheel + métricas).

**Métricas:** % resolvido sem humano · escalonamento por categoria · reabertura · CSAT · % de rascunho aceito sem edição por intent.
