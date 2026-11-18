# Regras de Escalonamento — Agente CX

Define **quando** e **como** o agente passa um atendimento para um humano do time de CX.
Funciona em camadas: agregar mensagens → filtro rápido no n8n → julgamento do agente com histórico.

---

## Camada −1 — ESCOPO: o que a Marina sequer abre (filtro no n8n, ANTES de tudo)

> ⭐ **A Marina atua em UM único canal.** O n8n só deve acionar a Marina se a mensagem satisfizer **as duas condições**:
> 1. **Canal = "WhatsApp Oficial - Customer Care" (número dedicado de atendimento)** — **só e exclusivamente** este número/canal. Mensagens de **qualquer outro telefone/canal** (ex.: Vendas) → a Marina **não abre, não lê, não responde**. Nada de misturar com a equipe de Vendas.
> 2. **Dono da conversa (atendente) ≠ Atendente de outro produto.** Dentro do próprio Customer Care, a **Atendente de outro produto** cuida de **outro produto** — conversas cujo responsável é ela **não** são da Marina (ignora).
>
> Se qualquer condição falhar → a Marina **não age** (silêncio total; a conversa segue com o time humano/Vendas). O n8n filtra isso pelo `channel-account` (Customer Care Oficial) e pelo **owner/responsável** da conversa no CRM.

## Camada 0 — Agregar mensagens e carregar histórico (antes de tudo)

No WhatsApp o cliente manda **várias mensagens curtas em sequência**. Se a IA responder à primeira
isolada, perde o contexto (ex.: *"não vou na aula de hoje"* + *"quero cancelar"* viram duas leituras erradas).

**Duas peças no n8n, sempre antes da IA:**

1. **Debounce / janela de espera (buffer):** ao receber uma mensagem, **espere «8–12s»** por novas mensagens
   do mesmo contato. Se chegar outra, reinicia o timer. Quando o cliente para de digitar, **concatena**
   tudo em um único texto e só então segue o fluxo. (No n8n: nó de espera + chave por `contato_id`,
   ou um nó de "Wait" com deduplicação.)
2. **Histórico da conversa:** guarde as últimas «N» trocas por contato (ex.: Redis, Postgres, Supabase, ou
   o próprio campo de anotações do CRM) e **injete o histórico no prompt** a cada chamada da IA.
   O agente decide com base na conversa inteira, não só na última linha.

> Sem a Camada 0, a regra de contexto do agente (§5.1 do prompt) não funciona — ela depende de receber o histórico.

---

## Camada 1 — Filtro de palavras-chave no n8n (sobre o texto já agregado)

Um nó de "Switch/IF" no n8n verifica a mensagem **já agregada** (Camada 0) **antes** de chamar a IA.
Se bater em qualquer gatilho abaixo, **pula a IA** e escala direto. É a rede de segurança barata.

> Cuidado: este filtro é **cego ao contexto** (olha só palavras). Por isso ele só contém expressões
> de **alta certeza** (contrato, financeiro, jurídico). Casos que dependem de contexto — como "cancelar"
> sozinho — **não** entram aqui; vão para a IA, que tem o histórico e decide certo.

| Categoria | Gatilhos (palavras/expressões) | Ação |
|---|---|---|
| **Cancelamento de CONTRATO** | cancelar contrato, cancelar matrícula, cancelar assinatura, encerrar contrato, rescindir, rescisão, quero sair do curso, desistir do curso, não quero mais o curso, dar baixa no contrato, trancar definitivo | Escala → **CX** |
| **Pagamento** | reembolso, estorno, cobrança, cobrado, duplicado, não reconheço, cartão, boleto, pix não caiu, paguei e não | Escala → **CX** |
| **Pedido humano** | falar com atendente, falar com humano, pessoa de verdade, atendente, sac | Escala → **CX** |
| **Pedido de ligação** (Marina NÃO liga) | pode me ligar, me liga, me liga por favor, prefiro por telefone, tem como ligar, ligação, me telefona | Escala → **CX** (um humano liga) |
| **Reclamação grave** | procon, advogado, processo, processar, reclame aqui, absurdo, golpe | Escala → **CX** (urgência crítica) |
| **Preço / valor** | quanto custa, qual o valor, qual o preço, mensalidade, quanto é, valor do pacote, tabela de preços | Escala → fila COMERCIAL/VENDAS |
| **Produto fora do escopo** | aula particular, particulares, premium, private (avulso), exame internacional, preparatório, kids, teens, infantil | Escala → fila COMERCIAL/VENDAS |

> ⚠️ **NÃO** coloque "cancelar" sozinho neste filtro. "Cancelar aula" / "desmarcar" é um caso que o **agente resolve**
> (ver seção abaixo). Só expressões ligadas a **contrato/matrícula** escalam direto. Se vier "cancelar" sem contexto,
> deixe passar pra IA — ela vai **perguntar** qual dos dois é antes de qualquer coisa.

> Mantenha essa lista versionada aqui. Toda palavra nova que causar problema vira uma linha nova.

## ⚠️ Os DOIS tipos de cancelamento (o agente precisa diferenciar)

Esta é a distinção mais importante do atendimento. São tópicos **completamente diferentes**.

### A) Cancelamento de AULA → o agente REGISTRA (não escala, não executa)
Cliente quer desmarcar/remarcar **uma aula específica** (não sair do curso). É rotina, e o objetivo é
exatamente tirar isso do humano. O agente:
1. Identifica a **modalidade** do aluno (define o prazo: **Plano A e aulas em grupo do Plano B = 3h**; **aulas private do Plano B = 48h**).
2. Verifica se está **dentro do prazo** de antecedência e do **limite de 25%** das aulas do mês.
3. **Registra a solicitação** (`acao: "registrar_aula"`, com `dados_aula` preenchido) e confirma ao cliente
   que o pedido **foi registrado** — nunca diz "já está cancelada".
4. Se está **fora do prazo** → explica a regra com gentileza (a aula é perdida / sem reposição, salvo exceções da base)
   e registra mesmo assim com `dentro_do_prazo: false` para a agenda decidir.
5. Sinais típicos: *"cancelar minha aula de amanhã"*, *"não vou poder na aula"*, *"desmarcar"*, *"remarcar a aula"*, *"reagendar"*.

> **Como funciona hoje:** o agente **registra** o pedido (grava `dados_aula` no CRM / fila da agenda) e a equipe
> de agenda processa. **Futuro:** será linkado ao portal e o cancelamento da aula acontecerá **automaticamente**.
> Quando essa integração existir, basta o n8n chamar a API do portal no ramo `registrar_aula` — o resto do fluxo não muda.

### B) Cancelamento de CONTRATO → SEMPRE escala (fila RETENÇÃO)
Cliente quer encerrar a **matrícula/contrato/assinatura** — sair do curso de vez. Envolve multa contratual,
direito de arrependimento, reembolso. **O agente nunca decide nem executa isso.**
1. Acolhe sem fricção, **não** tenta convencer nem segurar (isso é trabalho do time de retenção).
2. Pode **explicar a política** se perguntado (números exatos dos termos), mas **não confirma** valores nem processa.
3. `acao: "escalar"`, `motivo_escalonamento: "cancelamento"`.
4. Sinais típicos: *"cancelar meu contrato"*, *"cancelar a matrícula"*, *"quero sair do curso"*, *"encerrar minha assinatura"*.

### C) Ambíguo ("quero cancelar", sem dizer o quê) → o agente PERGUNTA primeiro
Nunca assuma. Antes de resolver ou escalar, o agente faz **uma** pergunta para diferenciar:
> *"Claro, posso te ajudar! Só pra eu te direcionar certo: você quer desmarcar **uma aula específica**, ou encerrar **sua matrícula/contrato** com a gente?"*
- Resposta = aula → fluxo A. Resposta = contrato/matrícula → fluxo B.

## Camada 2 — Julgamento do agente (IA)

Quando a mensagem passa pelo filtro, a IA decide. Ela escala (`acao: "escalar"`) quando:
- Detecta intenção dos casos acima mesmo sem a palavra exata (ex.: *"não quero mais isso"* = cancelamento)
- `confianca < 0.6` (resposta não está clara na base de conhecimento)
- Cliente demonstra irritação / frustração crescente
- Pedido de desconto, exceção ou negociação comercial
- **Preço/valor de qualquer produto** ("quanto custa", mensalidade, parcelas) → o agente **nunca informa valor**, escala `comercial`
- **Produto fora do escopo** (Aulas Particulares, Aulas Particulares Premium/Private, preparatório para exame internacional, Kids/Teens) → escala `comercial`
- Qualquer tema de dados sensíveis / LGPD
- **Cálculo ou interpretação** (`motivo: calculo_interpretacao`): a pergunta exige calcular valor (multa contratual,
  residual, proporcional) ou interpretar se o caso se enquadra (arrependimento, reembolso, limite de 25%).
  O agente **não responde ao cliente** — escala e preenche **`sugestao_resposta`** com o cálculo/resposta provável
  para o **atendente revisar e enviar**. Dúvida informativa puramente factual (ex.: "qual o prazo de arrependimento?")
  o agente responde direto, sem escalar.
- **Espanhol** (`motivo: espanhol`): o foco atual é o **inglês**. **Qualquer dúvida/atendimento relacionado ao
  espanhol** (cursos de espanhol, módulos em espanhol, liberações em espanhol) → o agente **não resolve, escala para humano**.

## Roteamento no CRM (para onde vai cada escalonamento)

Quando `acao = "escalar"`, o n8n atualiza o negócio no CRM:
1. Move o negócio para a **etapa "Atendimento Humano"** (criar essa etapa no funil)
2. Aplica a **tag** do `motivo_escalonamento` (`cancelamento`, `pagamento`, etc.)
3. Cola o `resumo_para_humano` (e a `sugestao_resposta`, quando houver) como **nota/anotação** no contato
4. **Notifica o time NA HORA via Slack** — no canal da equipe certa (CX ou Comercial) — para o atendente assumir o quanto antes.

> **Handoff silencioso:** o humano assume a **mesma conversa** no CRM, **sem anunciar troca de atendente**. A Marina
> não diz "vou te passar para um especialista"; ela usa uma frase natural de "ganhar tempo" (dentro do horário) ou
> a frase de "encaminhar à equipe" (fora do horário). Para o cliente, é um atendimento contínuo. (ver `prompt-agente-cx.md` §6)

> Quando vier `sugestao_resposta`, o atendente **revisa e envia** (ou edita). É a IA adiantando o trabalho,
> sem assumir a decisão. Ótimo para os casos de `calculo_interpretacao`.

> **Duas filas reais (jun/2026):** **CX / Customer Care / Atendimento Pós-venda** e **Comercial / Vendas**.
> Por isso o roteamento é **para a equipe**, não para uma pessoa específica.

| motivo_escalonamento | Fila / equipe | Urgência |
|---|---|---|
| cancelamento (de contrato) | **CX** | Alta |
| pagamento (pós-venda) | **CX** | Alta |
| reclamacao_grave | **CX** | Crítica |
| calculo_interpretacao | **CX** (revisa a sugestão da IA) | Média |
| espanhol | **CX** | Média |
| pedido_humano | **CX** | Média |
| lgpd | **CX** | Alta |
| fora_da_base | **CX** | Baixa |
| **comercial** (preço, particulares, Aulas Particulares Premium, exame internacional, Kids, negociação) | **Comercial / Vendas** | Média |
| **renovacao** (pacote vencendo) | **Dono do contrato** (vendedor que fechou — lookup na planilha de vendas ou no CRM) | Média |
| **trancamento** (quer +90 dias ou sem acordo) | **CX** | Média |

E quando `acao = "registrar_aula"` (não é escalonamento): o n8n grava `dados_aula` na fila/etapa da **agenda**
(não na de atendimento humano) e confirma ao cliente. Futuramente, esse ramo chama a API do portal e cancela automático.

## Horário e SLA

A Marina opera 24x7. O atendimento humano funciona **das 7h às 20h**.
- **Dentro do horário (7h–20h):** handoff silencioso — Marina usa frase natural de "ganhar tempo" e o atendente assume a mesma conversa. Notificação no Slack **na hora**.
- **Fora do horário (antes das 7h / depois das 20h):** a Marina **contorna sem mentir** — informa que vai encaminhar para a equipe responsável, que retornará o mais breve possível. Ex.: *"Vou encaminhar isso pra equipe responsável e te retornamos o mais breve possível, tá? 💙"*
  **Mesmo fora do horário, o atendente é notificado NA HORA via Slack** e contata o cliente assim que possível.
- Vale para **todos** os motivos (inclusive reclamação grave): notificação imediata sempre.

## Fluxo de Cobrança (proativo / outbound) — separado do atendimento reativo

Cobrança **não** começa por uma mensagem do aluno; é o agente quem inicia. Por isso é um **segundo fluxo no n8n**,
agendado (cron), que lê a fonte de verdade (planilha de cobranças / gateway de pagamentos)
e executa a cadência.

**Cadência (3 contatos + desativação):**
1. Seleciona linhas com `Status = EM CONTATO` e parcela vencida.
2. Envia o lembrete (template aprovado) avançando o passo:
   `Primeiro contato` → (após «3 dias») `Segundo contato` → (após «3 dias») `Terceiro contato` → `Desativação`.
3. Marca a coluna TRUE correspondente e registra no CRM.

> ⚠️ **Coordenar com a régua nativa do gateway de pagamentos (não duplicar) + checar validade**:
> - O gateway de pagamentos já contata automaticamente na **emissão**, na **rejeição** e em **D+1 e D+5** do vencimento. A cobrança da Marina deve atuar **após D+5** ou no que a régua não cobre (negociação, troca de meio de pagamento, dúvida); antes de contatar, olhar o Histórico da fatura para não sobrepor mensagem.
> - **Antes de cobrar, confirmar que a parcela é VÁLIDA:** assinatura **Ativa** + fatura valor>0 Vencido. Fatura **"Cancelado"/R$0/"Não faturado"** = **INDEVIDA, não cobrar**. Fatura com valor vinculada a assinatura **cancelada** = **suspeita → escala financeiro** (não cobrar direto).
> - Enviar **sempre a 1ª parcela vencida** (mais antiga); se houver várias, **avisar o aluno** antes.

**O que o agente faz vs escala, quando o aluno RESPONDE à cobrança** (a resposta cai no fluxo reativo):
| Resposta do aluno | Ação |
|---|---|
| "Vou pagar amanhã" / promete data | Reenvia link, Status → **PROMESSA DE PAGTO**. `acao: responder` |
| Envia comprovante / "já paguei" | **Hoje:** pede comprovante e **escala FINANCEIRO** p/ dar baixa. **Com o gateway de pagamentos integrado:** o webhook `bill_paid` confirma sozinho (Status→PAGO) — só escala se houver divergência real. `acao: escalar` (pagamento) |
| "Não tenho dinheiro" / quer parcelar / desconto | **Negociação → escala** (sempre humano). `acao: escalar` (comercial) |
| "Não reconheço" / cobrança errada | **Escala FINANCEIRO** (possível INDEVIDO). `acao: escalar` (pagamento) |
| "Quero cancelar por causa disso" | **Escala RETENÇÃO** (contrato). `acao: escalar` (cancelamento) |

> O agente **nunca** dá baixa de pagamento, **nunca** desativa acesso sozinho e **nunca** negocia.
> Confirmação de pagamento depende de comprovante + conferência humana/gateway de pagamentos.

## Princípio-guia

> Na dúvida, **escale**. Um humano pegando um caso simples custa pouco.
> Uma IA dando resposta errada sobre cancelamento ou pagamento custa caro.

## Métricas para acompanhar (depois)

- % de atendimentos resolvidos sem humano (meta principal de redução)
- % de escalonamentos por categoria
- Taxa de reabertura (cliente volta com o mesmo problema)
- CSAT pós-atendimento
