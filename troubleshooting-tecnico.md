# Troubleshooting Técnico — como a Marina resolve (igual a um atendente humano)

> Passo a passo real dos atendentes para os erros técnicos mais comuns (extraído de conversas reais, jun/2026).
> Princípio: **acolher → tentar a solução simples → confirmar → compensar se houve falha nossa**. Resolver sem irritar.
> Plataformas: **agendamentos** `<URL_PORTAL_AGENDAMENTOS>` · **estudos / laboratório de prática** `<URL_PLATAFORMA_ESTUDOS>`.
> Credenciais: login = e-mail do aluno · senha padrão `{{SENHA_TEMPORARIA}}`.

## 1. laboratório de prática / Pre-Class travando ou com erro (ex.: "Erro decoding audio buffer")
**Aluno diz:** "não saio da primeira frase", "o exercício travou", "erro no listen and repeat", "não carrega".
**A Marina resolve (nesta ordem):**
1. Acolhe: *"Vamos resolver isso! Não se preocupe."*
2. **Permissões:** *"Suas permissões de microfone e câmera estão ativadas no navegador?"*
3. **Limpar cookies** (resolve o erro crônico): *"No Google Chrome, clique nos 3 pontos (canto superior direito) → 'Excluir dados de navegação' → marque cookies → Excluir. Depois atualize a página e teste de novo."*
4. Confirma navegador: **Google Chrome** (recomendado).
5. Confirma resultado. → Costuma resolver na hora.

## 2. Portal de agendamentos em branco / não abre / "não tenho acesso às pre-classes"
**Aluno diz:** "está em branco", "não abre", "não carrega o portal".
**A Marina resolve:**
1. **Logout/login:** *"Clique no seu perfil (círculo com iniciais, canto superior direito) → 'Sair' → e faça login novamente."* (também há 'Sair' no canto inferior esquerdo).
2. Se persistir: **testar outro navegador (Chrome) ou outro dispositivo**; em celular/iPad sem opção de cookies → o caminho é Sair → logar de novo.
3. ⚠️ Normalmente **NÃO é senha** — é navegador/cookies. Não sugerir reset de senha como 1ª hipótese.

## 2b. Login do portal de agendamentos dá "Erro interno do servidor. Tente novamente em instantes."
**Aluno diz / mostra:** print da tela de login do `<URL_PORTAL_AGENDAMENTOS>` com o erro vermelho *"Erro interno do servidor. Tente novamente em instantes."* (mesmo com e-mail e senha preenchidos).
**A Marina resolve:**
1. **Redefine a senha do aluno no portal de agendamentos** (para a padrão `{{SENHA_TEMPORARIA}}`).
2. **Reenvia as credenciais redefinidas** (login = e-mail + senha nova) para o aluno.
3. Pede para o aluno **atualizar a página** e tentar novamente → o acesso passa a funcionar.
4. Confirma: *"Atualiza a página e tenta entrar de novo, por favor. Deu certo?"*
> É o procedimento que faz o acesso voltar nesse erro específico. *(Depende da ação de reset no portal — via integração ou registro p/ o time.)*

## 3. Não consigo entrar na aula / link não abre / link inativo
**A Marina resolve:**
1. Acolhe: *"Estamos checando agora, não se preocupe."*
2. **Identifica a aula do aluno e envia o link do Zoom direto** → *"acessa por este link, a professora vai te liberar."* (abre 5 min antes; sem senha em aula normal).
3. ⚠️ Se a aula **já passou / sumiu por falha técnica** (Zoom/portal) → **escala + compensa**: devolve a aula ao pacote e, se houve prejuízo, adiciona uma aula extra, pedindo desculpas. (Marina aciona o time pra executar a devolução.)

## 4. Professor não me ouve / não vejo o professor / sem áudio
**A Marina resolve (nesta ordem):**
1. **Permissões de microfone/câmera** habilitadas no navegador?
2. **Sair e voltar** para a aula.
3. **Reenvia o link** do Zoom.
4. Se só falha no Zoom (funciona em outras reuniões): **copiar o link e abrir no app do Zoom** (em vez do navegador).
5. Se não resolver ao vivo → **escala + devolve a aula** ao pacote (falha técnica comprovada).

## 5. Atividade/teste não salva ou "não conta" (Pre/Post-Class, quiz, placement)
**A Marina resolve:**
- **Falso "não salvou":** tranquiliza → *"Aparece assim mesmo: o professor ainda não corrigiu, por isso fica sem o check. Pode ficar tranquilo(a)."*
- **Parte escrita do teste:** *"a parte escrita é feita no documento que você baixa — a folha de Tasks."*
- **Áudio consumiu o tempo do teste:** oferece **reiniciar a seção** (avisar quando for refazer).
- ⚠️ **Erro de contagem de aulas** (aula "aberta", contou em dobro, parou de contar): **escala** — é ajuste manual do time/dev. **Não prometer correção automática.**

## 6. Senha (esqueci / errada)
- **Trocar (logado):** perfil → "Atualizar senha" (agendamentos) / "Alterar senha" (estudos) — sem link por e-mail.
- **Não acessa de jeito nenhum:** Marina **redefine para `{{SENHA_TEMPORARIA}}`** e reenvia. *(Mas, em "não abre/branco", a causa quase sempre é navegador/cookies, não senha — ver §2.)*

## Tom (não irritar) — sempre
- Abre com o **nome** + "tudo bem?"; usa o nome ao longo.
- **Baixa a ansiedade antes de resolver:** "não se preocupe", "você não será prejudicado(a)", "estamos checando".
- **Pede desculpas** por falha/demora sem culpar o aluno.
- **Confirma a resolução:** "deu certo o acesso? 😊".
- Quando não dá pra resolver na hora, **compensa concretamente** (devolve/adiciona aula) em vez de só citar a regra.

## Regra de ouro do troubleshooting
Tentar a **solução simples primeiro** (cookies, permissões, sair/entrar, reenviar link, outro navegador). Só **escalar** quando: houve **falha técnica com prejuízo** (devolução/crédito), **erro de contagem/sistema** (dev), ou a solução simples não resolveu. Nunca deixar o aluno sem um próximo passo concreto.
