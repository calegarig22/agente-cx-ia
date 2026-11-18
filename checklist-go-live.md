# Checklist Go-Live — o que falta pro agente começar a atender no CRM (e ficar super forte)

> Duas camadas: **🔴 BLOQUEADOR** = não pode tocar num cliente real sem isso · **🟢 FORÇA** = profundidade que
> ele ganha com o tempo (via o flywheel de `aprendizado-e-evolucao.md`), não precisa estar 100% no dia 1.
> Coluna "Quem": **Escola** = decisão/dado que só vocês têm · **Eu** = eu redijo/estruturo · **Dev/n8n** = engenharia.
> Princípio: dá pra começar pequeno e supervisionado. Não precisa saber tudo — precisa ser **seguro** no que faz.

---

## 🔴 BLOQUEADORES — sem isto, não atende cliente real

### 1. Persona do agente ✅ FEITO (em `persona-seguranca-guardrails.md` + prompt)
Nome **Marina** (feminino), voz consolidada, frases-modelo no prompt. *(Falta só Escola validar as frases.)*

### 2. Verificação de identidade ✅ FEITO (integrado no prompt §4)
2 fatores p/ ações sensíveis: número de WhatsApp bate com cadastro + confirmar nome completo e e-mail; senão escala (lgpd).

### 3. Lista negra / guardrails ✅ FEITO (integrado no prompt §4)
Lista dura no prompt (dados de terceiros, promessas, concorrentes, jurídico/financeiro, política/religião, etc). *(Escola revisa se quer somar algo específico.)*

### 4. Roteamento de escalonamento ✅ FEITO (definido — falta só ligar no n8n)
Duas filas: **CX** (cancelamento, pagamento, reclamação, lgpd, pedido_humano, fora_da_base, espanhol, cálculo) e **Comercial** (preço, particulares, Aulas Particulares Premium, exame internacional, Kids, negociação). **Notificação na hora via Slack.** Handoff **silencioso** (humano assume a mesma conversa). *(Dev: criar etapa "Atendimento Humano" no CRM + integração Slack.)*

### 5. Horário comercial ✅ FEITO
Atendimento humano **7h–20h**. Dentro: handoff silencioso. Fora: Marina contorna sem mentir (encaminha à equipe, retorno o quanto antes) e **notifica o atendente na hora via Slack** mesmo assim. Notificação imediata para **todos** os motivos.

### 6. Plumbing técnico mínimo (Dev/n8n)
- Confirmar **plano Elite + conexão WhatsApp Oficial + webhook de "mensagem recebida"**.
- **Debounce** (juntar mensagens curtas) + **histórico** injetado no prompt (Camada 0).
- Roteador por `acao` chamando os endpoints do CRM.
- **Log de conversa** ligado desde o 1º atendimento (Fase A do aprendizado).

---

## 🟢 FORÇA — profundidade que o deixa atender mais (cresce com o flywheel)

### 7. Fechar as lacunas ⚠️ da FAQ (Escola + Eu) — alto volume provável
- **Como entrar na aula ao vivo (Zoom):** link, navegador, problemas comuns.
- **Certificados:** quando, como emitir, validade.
- **Nivelamento na visão do aluno** (ferramenta de teste de fala): onde faz, quanto tempo, como recebe resultado.
- **E-mail de cadastro errado:** procedimento exato.

### 8. Público-alvo fino por produto (Escola + Eu)
Pra qualificar lead: "qual curso é melhor pra mim?" → indicar Plano A / Plano B / Plano C por perfil/objetivo.
(Sem dar preço — isso escala.)

### 9. Ciclo de vida do aluno (Escola + Eu)
- **Renovação:** gatilho, quando o saldo expira, o que o agente faz/oferece.
- **Reativação** de aluno inativo.
- **Sinais de churn:** o que o agente faz vs. quando passa pra retenção.

### 10. Conhecimento que só conversa real ensina (Escola fornece dados → Eu estruturo)
- **Transcrições reais** (bem atendidas = modelo de tom; difíceis = o que evitar).
- **Top 10–20 motivos de contato por volume** → prioriza o que graduar primeiro.

### 11. Memória/contexto do cliente no prompt (Dev/n8n)
Plano, módulo atual, status de pagamento, aulas feitas — puxados dos sistemas → ele responde "olhando a ficha".

---

## Como começar a atender no CRM (sem esperar tudo ficar pronto)
**Piloto supervisionado (modo copiloto):**
1. Liga os 🔴 bloqueadores 1–6.
2. O agente **rascunha toda resposta**, mas **um humano aprova antes de enviar** (ou ele só responde 1–2 intents seguros).
3. Mede acerto (rascunho aceito sem edição) por intent.
4. **Gradua** intent por intent para auto-resposta quando bate ≥90% (ver `aprendizado-e-evolucao.md`, Fase F).
5. A cobertura cresce sozinha com o ritual semanal de curadoria.

> Resumo: pra **começar**, foco nos 🔴 (a maioria é decisão de vocês + um pouco de engenharia).
> Pra ficar **super forte**, os 🟢 entram continuamente pelo flywheel. Não é um big-bang; é graduação.
