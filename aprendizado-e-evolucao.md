# Aprendizado e Evolução do Agente — do "escala muito" ao "faz tudo sozinho"

> **Objetivo:** o agente aprende com o atendimento real e, com o tempo, resolve cada vez mais sozinho —
> até, no limite, fazer todo o atendimento. **Princípio honesto:** LLM **não aprende sozinho** ao conversar.
> Quem aprende é o **sistema em volta dele**: capturamos conversas, usamos a resposta humana como gabarito,
> minamos as lacunas e devolvemos pra base. Cada ciclo → o agente escala menos. Humano sempre no loop até a graduação.

---

## A roda (flywheel) em uma frase
**Capturar → Rotular (humano = gabarito) → Minerar lacunas → Curar na base → Medir → Graduar tema → repetir.**

```
   [Conversa real] → log
        │
        ▼
  agente responde OU escala (+ sugestao_resposta)
        │                         │
   resolveu                  humano responde  ←── GABARITO DE OURO
        │                         │
        └─────────► [Banco de conversas] ◄──────┘
                          │
              (semanal) minerar: temas + escalações + baixa confiança
                          │
                 curar Q→A novos na FAQ/base (revisão humana)
                          │
              medir métricas → graduar intents p/ auto-resposta
                          │
                     (volta ao topo, agente mais forte)
```

---

## Fase A — Capturar tudo (começa no dia 1, antes de "aprender" qualquer coisa)
Sem dado, não há aprendizado. Logar **cada turno** num banco (Postgres/Supabase/BigQuery — não na planilha).

**Schema mínimo do log de conversa:**
| Campo | Descrição |
|---|---|
| `contato_id`, `conversa_id`, `ts` | quem, qual conversa, quando |
| `mensagem_cliente` (agregada) | texto do cliente (após debounce) |
| `historico_usado` | o que foi injetado no prompt |
| `saida_agente_json` | o JSON completo (acao, mensagem, confianca, tags, motivo...) |
| `acao_final` | o que de fato aconteceu (enviado / escalado / corrigido) |
| `resposta_humana` | **se escalou: o que o humano respondeu** ← o gabarito |
| `sugestao_vs_final` | rascunho do agente × texto final humano (distância de edição) |
| `resolvido` / `reaberto` | desfecho |
| `csat` | nota do cliente, se houver |

> ⚠️ **LGPD:** conversas têm dado pessoal. Definir retenção, anonimização e acesso (já é tema da base de segurança).

## Fase B — A ponte que vocês já têm: `sugestao_resposta`
O campo `sugestao_resposta` (agente rascunha, humano edita e envia) é o **sinal de treino mais rico e de menor risco**.
- Fazer o agente **sempre rascunhar**, mesmo quando escala (não só em cálculo/interpretação).
- Guardar o par **{rascunho do agente, texto final do humano}**. A **diferença** mostra exatamente onde ele erra o tom/conteúdo.
- Risco zero: o humano continua no controle. Mas cada edição vira aprendizado.

## Fase C — Minerar e curar (ritual semanal — é aqui que ele "fica esperto")
1. **Agrupar escalações por tema** e ranquear por volume → mostra o que automatizar a seguir.
2. **Casos de `confianca` baixa** → lista de lacunas da base.
3. **Perguntas repetidas que o humano respondeu** → viram **novos itens de FAQ/base** (sempre revisados por humano antes de virar "verdade").
4. Atualizar `prompt`/`FAQ`/bases → versionar a mudança.
> Esse ritual é o motor. É literalmente "ensinar o que ele não sabia ontem", usando o que os humanos responderam.

## Fase D — Entender como humano (memória e contexto)
- **Histórico da conversa** (já previsto na Camada 0 das regras) — interpretar mensagens em conjunto.
- **Perfil do cliente no prompt:** plano, módulo atual, status de pagamento, aulas feitas — puxado dos sistemas
  (CRM/LMS/agendamentos/gateway de pagamentos). Aí ele responde "olhando a ficha", não no escuro.
- **Memória de longo prazo por contato:** o que já foi tratado, preferências, casos anteriores.

## Fase E — Medir (sem métrica, não há evolução)
- **% resolvido sem humano** (meta principal — sobe a cada ciclo).
- **% de escalação por categoria** (cai onde a gente curou).
- **Taxa de reabertura** (cliente volta com o mesmo problema = resposta ruim).
- **CSAT** pós-atendimento.
- **Golden set:** 30–50 casos reais com resposta esperada → roda a cada mudança de prompt/base (anti-regressão).

## Fase F — Graduar a autonomia, tema por tema (como vira "faz tudo sozinho")
Não se liga autonomia total de uma vez. Cada **intent** (ex.: "reenviar acesso", "como agendar", "prazo de cancelamento")
**graduada** de `escala` → `auto-responde` quando os dados provam que é seguro:

> **Critério de graduação (exemplo):** rascunho do agente aceito **sem edição em ≥90%** dos últimos **N** casos
> daquele intent **E** zero erro grave → o intent passa a auto-responder. Caso contrário, continua escalando.

A lista de "o que escala" encolhe a cada mês. Quando ela esvazia, o agente faz tudo. **É assim que se chega ao TUDO sozinho — medido, não no chute.**

## Fase G — (Depois, opcional) RAG e fine-tuning
- **RAG:** quando a base crescer muito, busca semântica injeta só o trecho relevante (em vez da base inteira).
- **Fine-tuning:** treinar um modelo nos pares {pergunta, resposta-humana-de-ouro} acumulados. **Só vale com volume alto** —
  na maioria dos casos, prompt forte + RAG + exemplos já resolvem. Não é o primeiro passo; é otimização tardia.

---

## Governança (inegociável durante todo o ciclo)
- Nada vira "verdade" do agente **sem revisão humana**. Curadoria é human-in-the-loop.
- **Termos** continuam supremos; aprendizado **nunca** sobrepõe regra dura (cancelamento/pagamento seguem escalando).
- Toda mudança de base/prompt é **versionada** (idealmente git) — dá pra reverter e auditar o que mudou e quando.

## Próximos passos concretos (ordem)
1. **Definir o banco de logs** (schema acima) e ligar no n8n — captura desde o primeiro atendimento.
2. **Sempre rascunhar `sugestao_resposta`** e guardar {rascunho × final humano}.
3. **Montar o golden set** inicial (puxa dos primeiros atendimentos reais).
4. **Definir as métricas** e um painel simples.
5. **Instaurar o ritual semanal** de mineração + curadoria.
6. **Escolher o 1º intent a graduar** (provável: reenvio de acesso / dúvida de agendamento — alto volume, baixo risco).
