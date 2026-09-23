# Missão Marte Unifor — refatoração SOLID

Reescrita do jogo de console **Missão Marte Unifor** aplicando os cinco princípios SOLID. A versão original (procedural, com tudo dentro da `Main`) fica preservada em `src/exercicio10/` para comparação; a versão nova vive em `src/solidexercicio10/` organizada em camadas.

---

## Equipe

- **Myrla Rodrigues**
- **Cristiano Gonçalves**
- **Iandeyara Farias**

Disciplina de Proj. Arquitetura de Sistemas — UNIFOR.

---

## Como compilar

O projeto usa **Java 17** (por causa dos `switch expressions` no `JogoService` e nos `arrow-cases` da `Nave`). Não usa Maven nem Gradle — a compilação é direta com `javac`.

A partir da raiz do repositório:

```bash
# Compila a versão SOLID em uma pasta out/
javac -d out $(find src/solidexercicio10 -name "*.java")

# Se quiser compilar também a versão original (procedural), para comparação:
javac -d out $(find src/exercicio10 -name "*.java")
```

No **IntelliJ IDEA**, basta abrir a pasta raiz como projeto — o `.iml` já vem configurado. `Build → Build Project` compila tudo.

---

## Como executar

Depois de compilar:

```bash
# Versão SOLID (é a entrega principal)
java -cp out solidexercicio10.Main

# Versão original, procedural
java -cp out exercicio10.Main
```

O jogo abre no terminal com um menu:

1. **Iniciar Nova Missão** — pergunta nome do piloto, dificuldade (`facil`, `medio`, `dificil`) e tamanho do mapa.
2. **Visualizar Ranking Top 5** — mostra as melhores pontuações salvas.
3. **Resetar Ranking** — apaga o arquivo de ranking.
4. **Sair**.

**Comandos dentro da partida:** `w`, `a`, `s`, `d` para mover a nave, `c` para embarcar o passageiro que está na mesma casa e `q` para abortar a missão. O objetivo é resgatar todos os passageiros e voltar à plataforma de pouso `L` em (0,0) sem perder as três vidas.

O ranking é gravado no arquivo `ranking-solid-exercicio10.json` na pasta em que o jogo é executado (o nome preserva a extensão do enunciado, mas o formato é texto pipe-separado — ver [Limitações](#limitações-que-permanecem)).

---

## Alterações realizadas

A refatoração foi feita **do zero**, não sobre o código original. Os principais movimentos:

- **Extração de camadas.** O pacote único do exercício original virou quatro pacotes: `model`, `service`, `repository` e `presentation`. Cada arquivo passou a viver na camada correspondente à sua responsabilidade.
- **Introdução de abstrações no `model`.** Criamos as interfaces `Posicionavel` (para qualquer coisa que ocupa uma casa do mapa) e `Movel` (para qualquer coisa que se mexe), além da classe abstrata `EntidadeMapa` que reúne as duas responsabilidades básicas. Assim, `Asteroide` implementa só `Posicionavel` (ele fica parado) e `Nave`/`Inimigo` implementam também `Movel`.
- **Hierarquia de `Passageiro`.** A classe `Passageiro` virou abstrata, com `Professor`, `Engenheiro` e `Astronauta` como subclasses concretas. Cada uma define sua própria `getPontuacao()` e `getSimbolo()`.
- **`Dificuldade` como enum.** Substituímos as strings soltas por um enum com fábrica estática `deString(...)` que faz o parsing tolerante da entrada do usuário.
- **Persistência isolada por interface.** Criamos a interface `RankingRepository` e a implementação concreta `RankingService` que grava em arquivo texto. O `JogoService` só conhece a interface — a `Main` é o único ponto que amarra a implementação concreta.
- **Renderização separada.** A responsabilidade de desenhar o mapa saiu do fluxo do jogo e virou uma classe própria, `MapaRenderer`, no pacote `presentation`.
- **`JogoService` como orquestrador.** Toda a lógica de rodar a partida ficou em `JogoService`, que recebe o `RankingRepository` no construtor. A `Main` só monta as peças.

---

## Decisões de projeto

- **Quatro camadas, não três.** Poderíamos ter juntado `presentation` dentro de `service`, mas separar deixa claro que o jogo não depende de a saída ser texto no console — se um dia trocarmos por interface gráfica, só `presentation` muda.
- **Composição manual na `Main`, sem framework de injeção de dependência.** O tamanho do projeto não justifica trazer Spring ou Guice. A `Main` faz o papel de container: cria os objetos concretos e passa para quem precisa.
- **Interfaces pequenas e específicas.** `Posicionavel` e `Movel` são duas interfaces separadas de propósito, para não obrigar `Asteroide` a implementar um `mover` vazio. É ISP aplicado.
- **Pontuação e quantidade de entidades por dificuldade no `JogoService`.** Escolha herdada do tutorial. Na revisão crítica identificamos que o lugar correto seria dentro do próprio enum `Dificuldade` — está anotado como prioridade média.
- **Ranking em arquivo texto pipe-separado.** É o formato mais simples possível para atender o requisito de persistir entre execuções sem trazer dependências externas (JSON parser, banco).
- **Manter a versão original preservada.** `src/exercicio10/` continua no repositório para permitir a comparação lado a lado antes/depois na apresentação.

---

## Limitações que permanecem

Documentadas em detalhes no arquivo `REVISAO-SOLID.md`. Em resumo:

- **`JogoService` acumula responsabilidades demais.** Menu, loop de partida, sorteio de posições, I/O e coordenação de ranking convivem numa classe só (~250 linhas). É prioridade alta da revisão — quebrar em `MissaoFactory` e `ConsoleIO`.
- **`MapaRenderer` decide o símbolo por comparação de string.** Todas as entidades já têm `getSimbolo()`, mas o renderer usa `if` sobre `getTipo()`. Prioridade alta.
- **Aleatoriedade não é injetada.** `Random` e `Math.random()` são usados direto, o que impede testes determinísticos. Prioridade média.
- **Balanceamento fica no `service`, não no enum.** Adicionar uma dificuldade nova exige tocar em dois métodos do `JogoService`. Prioridade média.
- **Movimento bloqueado ainda desconta pontuação.** Quando a nave está na borda e o comando é rejeitado, o `score--` acontece mesmo assim. É bug de regra de jogo, não SOLID. Prioridade média.
- **Sem testes automatizados.** Os 15 testes realizados foram todos manuais — descritos em `REVISAO-SOLID.md`, seção 5. Escrever JUnit exige antes injetar `Random` e `MapaRenderer` para conseguir determinismo.
- **Classe `Astronauta` nunca é instanciada.** O método que sorteia passageiros só cria `Professor` e `Engenheiro`. Prioridade baixa — ou incluir `Astronauta` na rotação, ou remover.
- **Nome `RankingService` é confuso.** A classe é adapter de persistência, não serviço de negócio. Melhor renome seria `FileRankingRepository`. Prioridade baixa.
- **Arquivo do ranking tem extensão `.json` mas formato é pipe-separado.** Descuido de nomenclatura herdado do enunciado. Prioridade baixa.

---

## Modelagem UML

### Diagrama de classes do domínio

[Visualizar diagrama de classes do domínio](docs/uml/diagrama-classes-model.png)

[Visualizar diagrama no draw.io](https://drive.google.com/file/d/1v34FQRjx9UyiHlihxl4nbmpREnkkuHlN/view?usp=sharing)

O diagrama representa as principais entidades do pacote `solidexercicio10.model`, incluindo classes abstratas, classes concretas, enumeração e interfaces. Também apresenta as relações de herança, implementação de interfaces, associações e multiplicidades entre as entidades, conforme a estrutura implementada no projeto.

### Diagrama de Pacotes

[Visualizar diagrama de pacotes (imagem)](docs/uml/diagrama-pacotes.png)

[Visualizar diagrama no draw.io](https://drive.google.com/file/d/1kQMIQr5LAlwEmta69JxR921DqDECYpVO/view?usp=sharing)

O projeto está organizado no pacote principal `solidexercicio10`, que contém a classe `Main` e os subpacotes `model`, `service`, `presentation` e `repository`.

O pacote `model` reúne as principais entidades e interfaces do domínio. O pacote `presentation` contém o `MapaRenderer`, responsável pela apresentação do mapa. O pacote `repository` define o contrato `RankingRepository` e as classes relacionadas ao armazenamento do ranking. Já o pacote `service` contém o `JogoService`, responsável pela lógica principal da aplicação.


## Estrutura de pastas

---

```
.
├── src/
│   ├── exercicio10/               # Versão original (procedural), preservada
│   └── solidexercicio10/          # Versão refatorada, entrega principal
│       ├── Main.java
│       ├── model/                 # Entidades e abstrações do jogo
│       ├── service/               # JogoService — orquestra a partida
│       ├── repository/            # Interface e implementação do ranking
│       └── presentation/          # MapaRenderer — desenha o mapa
├── docs/uml/                      # Diagramas UML (pacotes e classes)
├── apostilas-solid/               # Referência teórica dos princípios
├── assets/                        # SVGs de apoio (SOLID e conceitos OO)
├── tutorialSolid.md               # Tutorial que guiou a refatoração
├── REVISAO-SOLID.md               # Revisão crítica da entrega
└── README.md
```

---

## Documentos relacionados

- **`REVISAO-SOLID.md`** — revisão crítica obrigatória: observações por princípio, melhorias adicionais, decisões do tutorial com as quais concordamos e discordamos, testes realizados e prioridades.
- **`tutorialSolid.md`** — passo a passo do tutorial que orientou a refatoração.
- **`apostilas-solid/SOLID-README.md`** — índice das apostilas por princípio.


### Dependências principais

- `solidexercicio10` → `service` e `repository`, por meio da configuração realizada pelo `Main`.
- `service` → `model`, `presentation` e `repository`.
- `presentation` → `model`.
- `repository` → `model`.

### Ponto principal da arquitetura

O `service` depende do contrato **`RankingRepository`**, definido como uma interface no pacote `repository`, e não diretamente dos detalhes de persistência.

A classe `Main` fornece a implementação do `RankingRepository` ao serviço. Dessa forma, o `service` trabalha com uma abstração, enquanto os detalhes de como os dados são armazenados ficam separados da lógica principal da aplicação.
