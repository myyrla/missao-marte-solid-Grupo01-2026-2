# Missão Marte Unifor: atividade prática de SOLID

Este repositório é uma atividade guiada de refatoração em Java. O aluno parte
do mini-jogo em `src/exercicio10`, acompanha o tutorial em `src/README.md` e
cria uma versão reorganizada, mantendo o comportamento do jogo.

## Roteiro da atividade

1. Execute e conheça o jogo original em `src/exercicio10`.
2. Leia as apostilas em `apostilas-solid/` e identifique os problemas de design
   antes de alterar o código.
3. Siga o tutorial em `src/README.md`, criando a nova implementação em um
   pacote separado, como `solidexercicio10`.
4. Compile e execute a versão refatorada após cada etapa.
5. Faça a revisão final: procure novos pontos de melhoria, registre as
   decisões com as quais você concorda ou discorda e justifique cada uma.

## Entrega sugerida pelo GitHub

Crie um repositório individual no GitHub e envie o link pelo Moodle. O
repositório deve conter o código original preservado, a versão refatorada,
`REVISAO-SOLID.md`, instruções no `README.md` e commits que mostrem a evolução
da atividade.

Não existe uma única divisão correta de classes. A refatoração deve ser
explicada pelo problema que ela resolve, pelo princípio relacionado e pelo
custo que introduz.

## Materiais

- [Tutorial passo a passo](src/README.md)
- [Apostilas sobre SOLID](apostilas-solid/SOLID-README.md)
- [Código inicial do exercício](src/exercicio10/README.md)

## Modelagem UML

### Diagrama de classes do domínio

[Visualizar imagem no GitHub](docs/uml/diagrama-classes-model.png)

[Visualizar diagrama no Gdraw.io](https://drive.google.com/file/d/1v34FQRjx9UyiHlihxl4nbmpREnkkuHlN/view?usp=sharing)

O diagrama representa as principais entidades do pacote `solidexercicio10.model`, incluindo classes abstratas, classes concretas, enumeração e interfaces. Também apresenta as relações de herança, implementação de interfaces, associações e multiplicidades entre as entidades, conforme a estrutura implementada no projeto.

### Diagrama de Pacotes

[Visualizar diagrama de pacotes (imagem)](docs/uml/diagrama-pacotes.png)

[Visualizar diagrama no draw.io]([https://drive.google.com/file/d/1kQMIQr5LAlwEmta69JxR921DqDECYpVO/view?usp=sharing](https://drive.google.com/file/d/1kQMIQr5LAlwEmta69JxR921DqDECYpVO/view?usp=sharing))

O projeto está organizado no pacote principal `solidexercicio10`, que contém a classe `Main` e os subpacotes `model`, `service`, `presentation` e `repository`.

O pacote `model` reúne as principais entidades e interfaces do domínio. O pacote `presentation` contém o `MapaRenderer`, responsável pela apresentação do mapa. O pacote `repository` define o contrato `RankingRepository` e as classes relacionadas ao armazenamento do ranking. Já o pacote `service` contém o `JogoService`, responsável pela lógica principal da aplicação.

### Dependências principais

- `solidexercicio10` → `service` e `repository`, por meio da configuração realizada pelo `Main`.
- `service` → `model`, `presentation` e `repository`.
- `presentation` → `model`.
- `repository` → `model`.

### Ponto principal da arquitetura

O `service` depende do contrato **`RankingRepository`**, definido como uma interface no pacote `repository`, e não diretamente dos detalhes de persistência.

A classe `Main` fornece a implementação do `RankingRepository` ao serviço. Dessa forma, o `service` trabalha com uma abstração, enquanto os detalhes de como os dados são armazenados ficam separados da lógica principal da aplicação.
