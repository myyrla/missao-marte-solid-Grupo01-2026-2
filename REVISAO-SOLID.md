# Revisão crítica — Missão Marte Unifor (versão SOLID)

**Autoras/autores:** Antonio Cristiano Goncalves, Iandeyara Farias e Myrla Rodrigues.
**Data:** 22/09/2026
**Escopo revisado:** pacote `src/solidexercicio10`.

---

## 1. Observações por princípio SOLID

### 1.1 SRP — Single Responsibility Principle

**Local:** `JogoService` (pacote service)

**Princípio relacionado:** SRP (cada classe deve ter só um motivo para mudar).

**Observação:** o `JogoService` acumulou muitos papéis diferentes dentro dele. Quando abrimos o arquivo, encontramos no mesmo lugar: o menu principal do jogo, o que acontece dentro de cada rodada (mover, embarcar, colidir, perder vida), a criação do mapa e o sorteio de onde cada passageiro, asteroide e inimigo aparece, todas as mensagens impressas na tela, a leitura do que o jogador digita no teclado e ainda as chamadas para gravar o ranking. São áreas bem distintas do jogo convivendo numa classe só, que hoje tem cerca de 250 linhas.

**Impacto para manutenção, testes ou evolução:** o problema disso não aparece agora, aparece quando qualquer coisa precisa mudar. Se quisermos só trocar o texto do menu, mexemos nessa classe. Se quisermos rebalancear pontos, mexemos nessa classe. Se quisermos mudar como as entidades são sorteadas no mapa, mexemos nessa classe. Ou seja, coisas totalmente diferentes vão gerar alterações no mesmo arquivo, o que aumenta a chance de a gente quebrar uma parte sem querer enquanto conserta outra. Testar também fica ruim: para verificar uma regra pequena (por exemplo, "quando o score chega em zero, a partida acaba") precisamos simular uma partida inteira com teclado, porque a regra está embutida no meio do loop.

**Proposta:** dividir o `JogoService` em três pedaços menores e passá-los prontos no construtor. Um pedaço cuidaria de criar o mapa e sortear as posições das entidades (`MissaoFactory`). Outro cuidaria só de conversar com o usuário — imprimir menu, ler comandos, mostrar estatísticas (`ConsoleIO`). O `JogoService` ficaria enxuto, cuidando apenas do que acontece a cada turno da partida.

**Prioridade:** alta.

---

### 1.2 OCP — Open/Closed Principle

**Local:** `JogoService.criarNovaMissao` e `JogoService.definirPontuacaoInicial`

**Princípio relacionado:** OCP (uma classe deve estar aberta para receber novos comportamentos, mas fechada para precisar ser alterada toda vez que algo novo aparece).

**Observação:** hoje, os números que definem cada dificuldade (quantos passageiros, quantos asteroides, quantos inimigos, quanta pontuação inicial) estão dentro de blocos `if` e `switch` no meio do `JogoService`. É lá que está escrito, por exemplo, "se for difícil, coloca 6 passageiros, 3 asteroides e 3 inimigos" ou "se for fácil, começa com 30 pontos". O enum `Dificuldade` sabe apenas os nomes (FACIL, MEDIO, DIFICIL) — os valores que dão sentido a cada nome ficam do lado de fora.

**Impacto para manutenção, testes ou evolução:** o problema é que o ajuste do jogo mora no lugar errado. Sempre que quisermos criar uma dificuldade nova (por exemplo, EXTREMO), vamos ter que abrir o `JogoService` e adicionar mais uma condição em dois métodos diferentes. E se quisermos só rebalancear (mudar o médio de 5 para 6 passageiros), também mexemos no `JogoService`, que não deveria ser tocado por causa disso — ele cuida da partida, não do balanceamento.

**Proposta:** colocar os números dentro do próprio enum `Dificuldade`. Cada valor do enum (FACIL, MEDIO, DIFICIL) carregaria seus próprios números como campos: pontuação inicial, quantidade de passageiros, de asteroides e de inimigos. Assim, o `JogoService` só pergunta para a dificuldade "quantos passageiros você quer?" e recebe a resposta. Adicionar uma dificuldade nova vira uma única linha nova no enum, sem tocar no service.

**Prioridade:** média.

---

### 1.3 LSP — Liskov Substitution Principle

**Local:** `MapaRenderer.desenhar`

**Princípio relacionado:** LSP (se uma classe filha herda de uma classe mãe, ela deveria poder ser usada em qualquer lugar em que a mãe é usada, sem o código precisar saber qual filha é).

**Observação:** existe uma boa base montada — todas as entidades do jogo (Professor, Engenheiro, Astronauta, Asteroide, Inimigo, Nave) têm um método chamado `getSimbolo()` que devolve a letra usada no mapa. É esse método que deveria dizer para o renderer "eu sou um E", "eu sou um P", "eu sou um T". Só que o `MapaRenderer` não usa isso. Em vez de perguntar para cada passageiro qual é o símbolo dele, o renderer olha um outro campo (`getTipo()`, que devolve uma string tipo "Engenheiro") e usa `if` para decidir a letra: "se o tipo for Engenheiro, desenho E; se for Astronauta, desenho T; senão, P".

**Impacto para manutenção, testes ou evolução:** o problema é que o renderer só funciona corretamente para os tipos que ele já conhece pelo nome. Se amanhã criarmos um novo passageiro — digamos, `Medico` — mesmo que ele implemente `getSimbolo()` do jeito certo, ele vai continuar sendo desenhado como P porque o renderer não sabe comparar a string "Medico". Ou seja, a promessa da herança ("posso trocar uma classe filha por outra sem quebrar o código que usa a mãe") não vale aqui. Pior ainda: se alguém tentar "limpar" o código e remover o campo `tipo`, o programa continua compilando normal, mas o mapa passa a mostrar tudo como P — é um bug silencioso, sem aviso do compilador.

**Proposta:** confiar no método que já existe. Em vez de comparar strings, o renderer chamaria `passageiro.getSimbolo()` e usaria essa resposta direto. O mesmo vale para asteroide e inimigo. Assim, para adicionar um novo tipo de passageiro basta criar a classe nova com o seu próprio símbolo — o renderer nem precisa saber que ela existe.

**Prioridade:** alta.

---

### 1.4 ISP — Interface Segregation Principle

**Local:** `RankingRepository`

**Princípio relacionado:** ISP (uma interface — o "contrato" que uma classe assina — deve pedir só o que é realmente necessário, para ninguém ser obrigado a implementar métodos que não vai usar).

**Observação:** o contrato `RankingRepository` hoje exige que quem for guardar o ranking implemente dois métodos `salvar` diferentes: um simples, com só nome e pontuação, e outro completo, com nome, pontuação, dificuldade, quantidade de passageiros e tempo. Durante o jogo, ninguém chama o simples — só o completo é usado. O simples só existe por precaução, "vai que alguém precise".

**Impacto para manutenção, testes ou evolução:** o incômodo é que, se um dia quisermos criar outro tipo de repositório (por exemplo, um repositório de memória só para rodar testes), somos obrigados a implementar os dois métodos, mesmo sabendo que um deles nunca vai ser chamado. E fica ruim para quem lê a interface pela primeira vez: qual dos dois é o oficial? Por que tem os dois? Contrato inflado sem motivo.

**Proposta:** deixar na interface apenas o `salvar` completo. Se um dia surgir uma necessidade real do salvar simples, ele volta. Uma opção ainda melhor é receber um objeto `RankingEntry` pronto, evitando passar seis parâmetros soltos (essa parte está detalhada na melhoria 2.2).

**Prioridade:** média.

---

### 1.5 DIP — Dependency Inversion Principle

**Local:** `JogoService` (construtor) e `Missao.moverInimigos`

**Princípio relacionado:** DIP (uma classe não deve ela mesma escolher e criar as ferramentas que usa — as ferramentas devem chegar prontas por fora, para poderem ser trocadas).

**Observação:** o `JogoService` já faz uma parte disso certo: ele não cria o repositório de ranking sozinho, recebe pronto de fora (a `Main` entrega no construtor). O problema é que essa disciplina não vale para o resto. Dentro do próprio construtor, o `JogoService` chama `new MapaRenderer()` e `new Random()`, ou seja, escolhe e cria as ferramentas por conta própria, escondendo essa decisão. E na classe `Missao`, o método que sorteia o movimento dos inimigos usa `Math.random()` — um sorteador global que ninguém consegue controlar de fora.

**Impacto para manutenção, testes ou evolução:** o custo maior aparece na hora de testar. Como o sorteio é aleatório e ninguém controla a "semente" desse sorteio, cada execução de teste dá um resultado diferente. Não conseguimos escrever um teste que diga "quando o inimigo é sorteado nessa posição, a nave colide" — porque a posição muda toda hora. Também não dá para trocar o `MapaRenderer` por uma versão de teste que, em vez de imprimir na tela, guarde o desenho num texto para compararmos depois.

**Proposta:** também receber essas ferramentas de fora. O `MapaRenderer` e o `Random` entrariam pelo construtor do `JogoService`, e o mesmo `Random` seria passado para o `Missao` (ou o `Missao` receberia o dx/dy já sorteado). Assim, em produção o jogo continua igual, mas em teste conseguimos criar um `Random` com semente fixa e prever o resultado.

**Prioridade:** média.

---

## 2. Melhorias adicionais

### 2.1 Remover a duplicação entre `Passageiro.tipo` e as subclasses

**Local:** `Passageiro` e subclasses (`Professor`, `Engenheiro`, `Astronauta`)
**Princípio relacionado:** DRY (não repetir a mesma informação em dois lugares).
**Observação:** cada subclasse passa uma string (`"Professor"`, `"Engenheiro"`, `"Astronauta"`) para o construtor da classe mãe. Só que essa informação já é o próprio nome da classe — não precisava ser dita de novo por escrito. O único lugar que usa esse campo é o renderer, na comparação por string que a melhoria 1.3 já elimina.
**Impacto para manutenção, testes ou evolução:** é uma fonte fácil de bug. Se um dia renomearmos a classe `Engenheiro` para `Cientista` e esquecermos de trocar a string, a persistência começa a gravar `"Engenheiro"` para um objeto que na verdade é `Cientista`. Nada quebra na compilação, o erro só aparece depois.
**Proposta:** apagar o campo `tipo` e o getter. Onde for preciso mostrar o nome do tipo, usar `getClass().getSimpleName()`.
**Prioridade:** média.

### 2.2 Encapsular `RankingEntry` como `record` e simplificar `salvar`

**Local:** `RankingEntry` e `RankingRepository.salvar`
**Princípio relacionado:** encapsulamento (não deixar os dados de dentro de uma classe expostos direto).
**Observação:** hoje o `RankingEntry` deixa todos os campos como `public final`, dá para acessar por fora sem passar por método nenhum. Funciona, mas expõe a estrutura interna. E como não existe um objeto pronto para representar uma entrada de ranking, o método `salvar` precisa receber seis parâmetros soltos em sequência.
**Impacto para manutenção, testes ou evolução:** baixo agora, mas cresce se o ranking ganhar novos campos.
**Proposta:** transformar `RankingEntry` em um `record` Java, que já vem imutável e com métodos de acesso automáticos. E mudar a interface para `salvar(RankingEntry entry)`, passando o objeto inteiro em vez de seis parâmetros.
**Prioridade:** baixa.

### 2.3 Renomear `RankingService` para `FileRankingRepository`

**Local:** `RankingService`
**Princípio relacionado:** clareza de nomes.
**Observação:** o nome "Service" dá a entender que a classe tem regra de negócio dentro, mas na verdade ela só grava e lê o arquivo do ranking. É um "repositório em arquivo".
**Impacto para manutenção, testes ou evolução:** confunde a leitura e ocupa o nome. Se um dia surgir um serviço de verdade de ranking (por exemplo, regra anti-fraude), não vai ter mais como chamá-lo de `RankingService`.
**Proposta:** renomear para `FileRankingRepository`. A `Main` muda uma linha, o resto do código não sente.
**Prioridade:** baixa.

### 2.4 Remover ou usar de fato a classe `Astronauta`

**Local:** `Astronauta` e `JogoService.posicionarPassageiros`
**Princípio relacionado:** YAGNI (não deixar código pronto para uma necessidade que não existe).
**Observação:** `Astronauta` foi criada como subclasse de `Passageiro`, mas o método que espalha os passageiros pelo mapa só instancia `Professor` e `Engenheiro`. O símbolo `T` (Astronauta) existe no renderer e na legenda, mas nunca aparece durante o jogo.
**Impacto para manutenção, testes ou evolução:** é código morto — engorda o projeto e confunde quem lê.
**Proposta:** ou incluir `Astronauta` na rotação de sorteio dos passageiros, ou apagar a classe.
**Prioridade:** baixa.

---

## 3. Decisão do tutorial com a qual concordamos

**Local:** `RankingRepository` (interface) e composição feita na `Main`
**Princípio relacionado:** DIP
**Observação:** o tutorial fez o `JogoService` depender da interface `RankingRepository`, não da classe concreta. Só a `Main` conhece a implementação real (`RankingService` gravando em arquivo).
**Benefício:** isso deixa a persistência solta do resto do jogo. Se um dia quisermos trocar o arquivo texto por SQLite, ou usar um repositório em memória para rodar testes, o `JogoService` não precisa saber — a `Main` entrega a versão que faz sentido. E o tutorial não trouxe framework de injeção de dependência nenhum, ficou na composição manual dentro da `Main`, que é o tamanho certo para esse projeto.
**Prioridade:** informativa (não é achado a corrigir).

---

## 4. Decisão do tutorial com a qual discordamos

**Local:** `MapaRenderer.desenhar` e campo `Passageiro.tipo`
**Princípio relacionado:** LSP e DRY
**Observação:** o tutorial escolheu guardar em `Passageiro` um campo `String tipo` e resolver o desenho do mapa comparando essa string, mesmo com `getSimbolo()` já disponível em toda a hierarquia. Também deixou três versões públicas do método `desenhar` no renderer, embora o `JogoService` só use uma delas.
**Justificativa (tamanho do projeto, complexidade da alternativa, testabilidade e custo de manutenção):** a alternativa (usar o `getSimbolo()` polimórfico) é mais simples do que a solução do tutorial, não mais complexa. Apaga um campo, apaga um getter e troca um bloco de if por uma linha só. O custo de manter dois caminhos que decidem a mesma coisa (comparar string versus perguntar direto para o objeto) já é alto agora e vai crescer a cada tipo novo de passageiro. As três versões públicas do `desenhar` são código antecipado sem motivo — introduzem API que ninguém usa e ainda deixam quem lê na dúvida sobre qual chamar.
**Proposta:** aplicar 1.3 e 2.1 juntos e deixar um único método público no `MapaRenderer`.
**Prioridade:** alta (parte LSP) e baixa (parte dos overloads).

---

## 5. Testes realizados e resultados

Compilação e execução manuais com `javac` e `java` a partir de `src/`. Ainda não há testes automatizados no projeto.

| # | O que foi testado | Resultado |
|---|---|---|
| 1 | Compilar todos os arquivos do pacote `solidexercicio10` | OK, sem warnings |
| 2 | Rodar a `Main` e escolher opção inválida no menu | Mensagem "Opção inválida" e menu volta a aparecer |
| 3 | Iniciar partida na dificuldade fácil, tamanho 3 | Mapa 7×7 desenhado, pontuação inicial 30 |
| 4 | Mover a nave com w/s/a/d dentro dos limites | Nave move e score cai 1 por movimento |
| 5 | Tentar mover além do limite do mapa | Nave não sai do lugar, mas o score cai mesmo assim (achado colateral) |
| 6 | Embarcar passageiro com `c` em cima do símbolo | +15 (Professor) ou +20 (Engenheiro), contador "A bordo" incrementa |
| 7 | Tentar embarcar com a nave cheia | Mensagem "Nave cheia!" |
| 8 | Colidir com asteroide | "Colisão detectada!" e vidas caem |
| 9 | Perder as 3 vidas | GAME OVER e volta ao menu |
| 10 | Coletar todos os passageiros e voltar a (0,0) | Estatísticas exibidas e entrada gravada no ranking |
| 11 | Consultar Top 5 (opção 2) | Linha da partida aparece no ranking |
| 12 | Resetar ranking (opção 3) e consultar de novo | "Nenhum registro de ranking ainda" |
| 13 | Fechar e reabrir o jogo, consultar ranking | Entradas anteriores continuam lá (persistência OK) |
| 14 | Digitar uma dificuldade inexistente | Cai no padrão MEDIO (comportamento intencional) |
| 15 | Rodar várias partidas observando o símbolo `T` (Astronauta) | Nunca aparece — confirma o dead code (2.4) |

**Achado colateral registrado nos testes**

Movimento bloqueado ainda desconta pontuação (teste 5). Quando a nave está na borda e o comando é rejeitado por `Nave.moverComLimites`, o `JogoService` executa `score--` mesmo assim. É regra de jogo, não SOLID, mas fica anotado. Prioridade: **média**.

---

## 6. Sumário de prioridades

| Achado | Princípio | Prioridade |
|---|---|---|
| 1.1 Muitos papéis no `JogoService` | SRP | Alta |
| 1.3 Renderer decide símbolo por string | LSP | Alta |
| 4. Discordância: `tipo` + versões extras de `desenhar` | LSP + DRY | Alta (parte LSP) |
| 1.2 if/switch de `Dificuldade` no service | OCP | Média |
| 1.4 `salvar` sem uso na interface | ISP | Média |
| 1.5 `Random` e `MapaRenderer` presos por dentro | DIP | Média |
| 2.1 `Passageiro.tipo` duplicado | DRY | Média |
| 5. Movimento bloqueado desconta score | Regra de jogo | Média |
| 2.2 `RankingEntry` como record | Encapsulamento | Baixa |
| 2.3 Renomear `RankingService` | Clareza | Baixa |
| 2.4 `Astronauta` código morto | YAGNI | Baixa |

**Ordem sugerida para aplicar**

1. Fazer 1.3 e 2.1 juntos — refatoração pequena que elimina um caminho de bug inteiro.
2. Aplicar 1.1 (extrair `MissaoFactory` e `ConsoleIO`) — destrava a possibilidade de escrever testes.
3. Aplicar 1.5 (passar `Random` e `MapaRenderer` de fora) — permite JUnit determinístico.
4. Aplicar 1.2 e 1.4 (números no enum e interface enxuta).
5. Polimento: 2.2, 2.3, 2.4 e o ajuste do score em movimento bloqueado, em um único commit.
