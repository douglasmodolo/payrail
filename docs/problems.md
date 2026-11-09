# Problemas do payrail

Esta é a lista do **que** o sistema precisa resolver. O **como** é seu.

Cada problema diz:

- **Situação:** o que o sistema precisa fazer ou o que não pode acontecer.
- **Pronto quando:** como provar que está resolvido. Quase sempre é um teste, uma medição
  ou uma falha provocada de propósito.
- **Não precisa:** o que fica de fora, pra não gastar energia onde não paga agora.
- **Restrição** (às vezes): uma exigência que vem das vagas, não do design. Ela limita as
  opções, mas a escolha dentro dela continua sua.

Nenhum problema diz qual técnica usar. Se você travar, peça ajuda.

---

## Como trabalhar cada problema

1. Leia a situação e **proponha uma solução antes de pesquisar**. Ingênua serve.
2. Discuta: eu ataco com cenários de falha e de escala.
3. Registre a decisão em `docs/adr/NNNN-titulo.md`: contexto, opções consideradas, escolha
   e o que ela custa.
4. Implemente e prove com o "pronto quando".
5. Se a prova contradisser a decisão, escreva um ADR novo dizendo o que mudou e por quê.

## Régua de aprendizado (por assunto)

| Nível | Pergunta |
|---|---|
| 1. Explico | Consigo explicar sem consultar nada? |
| 2. Aplico | Usei num problema real deste sistema? |
| 3. Vi falhar | Provoquei o problema e vi acontecer antes de corrigir? |
| 4. Sei quando não usar | Sei explicar o custo e quando não vale a pena? |

Um assunto só conta como aprendido no nível 4. De tempos em tempos tem sabatina sem
consulta sobre problemas antigos.

## Critério de "pronto pra entrevista"

Desenhar o payrail inteiro em 45 minutos, falando em voz alta, e defender cada escolha
quando ela for atacada.

---

## Decisões já tomadas (são de carreira, não de design)

- **Java 21 + Spring Boot:** é o que as vagas de backend no Brasil e na América Latina
  mais pedem.
- **PostgreSQL:** o banco relacional mais citado nas vagas.
- **Docker e GitHub Actions:** aparecem em quase todas as vagas.
- **Código, nomes, commits e ADRs em inglês.** Este documento fica em português.

---

## Etapa 1 — Dinheiro que não some

### P1. Abrir conta e consultar saldo
- **Situação:** um cliente abre uma conta e consulta o saldo pela API.
- **Pronto quando:** existe uma API para criar conta e consultar saldo, com testes
  automatizados. Valores como R$ 0,10 + R$ 0,20 resultam em exatamente R$ 0,30, sem
  centavo perdido nem sobrando.
- **Não precisa:** autenticação, frontend, mais de uma moeda.

### P2. Depositar e transferir
- **Situação:** o cliente deposita dinheiro e transfere para outra conta. Ele consegue ver
  o extrato com o histórico.
- **Invariantes (nunca podem ser quebradas):**
  - A soma de todo o dinheiro do sistema não muda numa transferência.
  - Uma conta nunca fica com saldo negativo.
  - Todo saldo pode ser explicado pelo histórico da conta.
- **Pronto quando:** existem testes automatizados que verificam cada invariante, e o
  extrato bate com o saldo.
- **Não precisa:** tarifas, agendamento, estorno.

### P3. Roda em qualquer máquina
- **Situação:** qualquer pessoa clona o repositório e sobe o sistema inteiro sem instalar
  Postgres nem configurar nada na mão. Os testes rodam contra o mesmo banco usado de
  verdade, não um banco falso.
- **Pronto quando:** clonar e rodar um comando sobe tudo, e a suíte de testes passa numa
  máquina que só tem Docker e o JDK.
- **Não precisa:** Kubernetes.

### P4. O schema do banco tem história
- **Situação:** o schema do banco muda ao longo do projeto. Qualquer ambiente precisa
  chegar à versão atual sozinho, e nenhuma mudança pode ser feita na mão.
- **Pronto quando:** um banco vazio chega ao schema atual ao subir a aplicação, e cada
  mudança fica registrada e versionada no repositório.

### P5. A main nunca quebra
- **Situação:** nenhum código entra na branch principal sem compilar e passar nos testes.
- **Pronto quando:** todo PR roda build e testes automaticamente, e um PR com teste
  quebrado não consegue ser mergeado.
- **Não precisa:** deploy automático (fica pra Etapa 6).

---

## Etapa 2 — Correto sob pressão

### P6. Duas transferências ao mesmo tempo
- **Situação:** a conta tem saldo para uma única transferência e chegam 100 transferências
  simultâneas saindo dela.
- **Pronto quando:** um teste concorrente mostra o problema acontecendo antes da correção
  (saldo negativo ou dinheiro criado do nada). Depois da correção, o mesmo teste passa
  sempre. Você mediu o throughput antes e depois e explica a diferença.
- **Não precisa:** lock distribuído entre várias instâncias (ainda é uma aplicação só).

### P7. O cliente clicou duas vezes
- **Situação:** o aplicativo reenvia a mesma transferência porque a rede caiu ou o cliente
  clicou de novo.
- **Pronto quando:** enviar a mesma requisição N vezes, em sequência ou em paralelo, move
  o dinheiro uma única vez, e todas as respostas são coerentes.

### P8. Dez milhões de contas
- **Situação:** o sistema tem 10 milhões de contas e 50 milhões de lançamentos (dados
  gerados por você). O extrato e o saldo continuam rápidos sob carga.
- **Pronto quando:**
  - existe um gerador de dados que você roda e descarta quando quiser;
  - existe um teste de carga que você consegue repetir;
  - você mediu p50, p95 e p99 antes de mexer, definiu uma meta e chegou nela;
  - você sabe ler o plano de execução das queries principais.
- **Não precisa:** sharding nem réplicas ainda.

### P9. Todo mundo consulta o saldo toda hora
- **Situação:** o aplicativo consulta o saldo cada vez que abre, e isso é a maior parte do
  tráfego. É preciso reduzir a carga no banco.
- **Pronto quando:** você mostra a carga no banco antes e depois, e escreveu quando o saldo
  exibido pode estar desatualizado, por quanto tempo e por que isso é aceitável (ou não é)
  em cada tela.

### P10. Ninguém mexe no dinheiro dos outros
- **Situação:** só o dono move dinheiro da própria conta, e trocar um id na URL não mostra
  dados de outra pessoa. Um cliente não consegue fazer mil transferências por minuto e
  existe um limite de valor por dia.
- **Pronto quando:** existem testes de ataque (sem credencial, com credencial de outra
  pessoa, enumerando ids, rajada de requisições, valor acima do limite) e todos são
  barrados.
- **Não precisa:** login social, recuperação de senha.

---

## Etapa 3 — Fora do caminho da requisição

### P11. Avisar o destinatário
- **Situação:** quem recebe uma transferência é notificado. A notificação não pode deixar a
  transferência mais lenta, e nenhuma notificação se perde se o serviço que notifica
  estiver fora do ar.
- **Restrição:** use um dos brokers que as vagas pedem (Kafka, RabbitMQ ou SQS). Qual e por
  quê é decisão sua.
- **Pronto quando:** você derruba o notificador, faz 1.000 transferências e sobe ele de
  novo: as 1.000 notificações chegam. A latência da transferência é a mesma com o
  notificador ligado ou desligado.
- **Não precisa:** e-mail ou push de verdade. Um log ou uma tabela de "enviados" basta.

### P12. Salvou mas não avisou
- **Situação:** o processo pode morrer entre gravar a transferência e publicar o evento, ou
  o contrário.
- **Pronto quando:** você mata o processo nesse ponto exato, repetidas vezes, e nunca
  existe transferência sem evento nem evento sem transferência.

### P13. Mensagem repetida e mensagem venenosa
- **Situação:** o consumidor recebe a mesma mensagem duas vezes. Existe uma mensagem que
  sempre falha ao ser processada.
- **Pronto quando:** a mensagem duplicada não gera efeito duplo. A mensagem venenosa não
  trava o processamento das outras e fica visível pra alguém analisar e reprocessar.

### P14. Relatório pesado não pode atrapalhar transferência
- **Situação:** extratos longos e relatórios pesados disputam o banco com as
  transferências.
- **Pronto quando:** você roda uma carga pesada de relatórios ao mesmo tempo que
  transferências, e o p99 da transferência não piora além de uma meta definida por você.
  O ADR compara pelo menos uma opção relacional com uma não relacional.

---

## Etapa 4 — Copiloto do analista (IA)

O LLM nunca entra no caminho da transferência. Ele é lento, varia e custa caro por
chamada.

### P15. Transações suspeitas viram casos
- **Situação:** regras simples marcam transações suspeitas (por exemplo, valor muito acima
  do padrão da conta, conta recém-criada, muitos destinatários novos em pouco tempo). Cada
  transação marcada vira um caso numa fila para um analista humano decidir: aprovar,
  bloquear ou escalar.
- **Pronto quando:** você gera tráfego com fraudes plantadas por você e os casos aparecem
  na fila. A decisão do analista fica registrada. As regras são a **baseline**, a régua que
  a IA vai ter que bater.

### P16. O agente investiga
- **Situação:** um agente pega cada caso, investiga usando o próprio sistema (histórico da
  conta, destinatários, limites, o que mais você decidir oferecer) e sugere uma decisão com
  justificativa e as evidências que consultou. O humano continua decidindo.
- **Restrição:** escreva o loop do agente (chamar o modelo, executar ferramenta, devolver o
  resultado) direto com o SDK, sem framework de agentes. Framework só depois, se você
  quiser comparar.
- **Pronto quando:** cada caso tem uma sugestão, uma justificativa e a lista de ferramentas
  chamadas, e você consegue reproduzir a investigação de qualquer caso.

### P17. O agente é melhor que as regras?
- **Situação:** é preciso saber se o agente vale o que custa.
- **Pronto quando:** existe um conjunto de casos rotulados (gerado por você) e um comando
  que roda o agente contra ele e reporta acerto, falsos positivos e negativos, latência,
  custo por decisão e taxa de escalonamento, sempre lado a lado com a baseline. Esse
  comando roda a cada mudança de prompt, de ferramenta ou de modelo.

### P18. Casos parecidos
- **Situação:** o analista sempre pergunta "já vimos algo assim?". O agente deve encontrar
  casos passados semelhantes.
- **Pronto quando:** o agente cita casos parecidos, e a avaliação do P17 mostra se isso
  melhorou o acerto ou não. Um resultado negativo também conta, desde que esteja medido.

### P19. O agente em produção
- **Situação:** o provedor do LLM fica lento ou cai, o custo dispara, o mesmo caso é
  analisado duas vezes, ou dados sensíveis vão parar no prompt sem necessidade.
- **Pronto quando:** com o LLM fora, os casos seguem pro humano sem sugestão e nada trava.
  Existe um teto de custo que é respeitado. O mesmo caso não gera duas análises. Você
  documentou quais dados vão pro modelo e por quê.

---

## Etapa 5 — Processar em lote e guardar arquivos

### P20. Fechamento do dia
- **Situação:** todo dia, um processo confere se os saldos batem com o histórico dos 50
  milhões de lançamentos e aponta divergências. Se ele cair no meio, retoma de onde parou.
- **Restrição:** Spring Batch (aparece nas vagas da Nava e da Clara).
- **Pronto quando:**
  - você planta divergências e o job acha todas;
  - você mata o job no meio e ele retoma sem reprocessar o que já foi feito;
  - você mediu o tempo total e explica onde ele é gasto.

### P21. Comprovantes e extratos em PDF
- **Situação:** cada transferência gera um comprovante, e todo mês sai um extrato em PDF.
  Os arquivos não ficam no banco. O cliente baixa por um link que expira.
- **Restrição:** armazenamento com API compatível com S3.
- **Pronto quando:** o PDF é gerado sem travar a transferência, o link funciona e depois
  expira, e ninguém baixa o arquivo de outra pessoa.

### P22. A tabela que não para de crescer
- **Situação:** o histórico de lançamentos cresce para sempre, e as consultas e a
  manutenção do banco vão piorando.
- **Pronto quando:** o dado antigo sai do caminho quente, você mediu as consultas e a
  manutenção antes e depois, e o ADR diz o que acontece com o dado antigo e por quanto
  tempo ele é guardado.

---

## Etapa 6 — Separar, falhar e enxergar

### P23. O time de risco quer independência
- **Situação:** a parte de risco e antifraude (Etapa 4) precisa evoluir e fazer deploy sem
  depender do resto do sistema.
- **Pronto quando:** são dois serviços com deploy independente e dados próprios, e o ADR
  lista com honestidade o que **piorou** com a separação.

### P24. A dependência ficou lenta
- **Situação:** o serviço de risco demora 10 segundos para responder, ou cai.
- **Pronto quando:** você injeta latência e queda de propósito. A transferência se comporta
  do jeito que você decidiu e documentou (bloquear, liberar, enfileirar), e o sistema não
  esgota threads nem conexões.

### P25. Onde está a lentidão?
- **Situação:** uma transferência específica demorou e ninguém sabe em qual parte.
- **Pronto quando:**
  - dada uma requisição lenta, você aponta em menos de 5 minutos qual etapa demorou;
  - existe um painel com métricas de negócio (transferências por minuto, casos abertos) e
    técnicas;
  - existe um indicador de disponibilidade definido, e você sabe calculá-lo.

---

## Etapa 7 — Na nuvem

### P26. Qualquer pessoa acessa
- **Situação:** o payrail roda na nuvem com um link público, e todo merge na main vai pra
  produção sozinho.
- **Restrição:** AWS. Antes de criar qualquer recurso, configure um alarme de orçamento.
- **Pronto quando:** o link público funciona, um merge faz o deploy sem nenhum passo manual,
  e você sabe quanto custa por mês.

### P27. Recriar tudo do zero
- **Situação:** o ambiente de nuvem foi apagado e precisa voltar.
- **Pronto quando:** um comando recria o ambiente inteiro, e outro comando apaga tudo sem
  sobrar recurso cobrando.

---

## Lista viva

Esta lista muda conforme o sistema e as vagas mudarem. Problemas novos entram no mesmo
formato.
