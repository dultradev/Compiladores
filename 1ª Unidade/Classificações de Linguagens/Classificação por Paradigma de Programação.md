Os paradigmas específicos costumam ser vistos como "subcategorias" dentro do imperativo/declarativo (embora alguns paradigmas modernos combinem características das duas linhas).

#### Paradigma procedural (ou estruturado)

- Organiza o programa em **procedimentos/funções** que operam sobre dados, seguindo uma sequência lógica de execução.
- É a evolução direta do paradigma imperativo, com ênfase em **decomposição do problema em sub-rotinas**.
- Exemplos: C, Pascal, Fortran.

#### Paradigma orientado a objetos (OO)

- Organiza o programa em torno de **objetos**, que agrupam dados (atributos) e comportamentos (métodos) relacionados.
- Conceitos-chave: encapsulamento, herança, polimorfismo, abstração.
- Exemplos: Java, C++, C#, Python (suporta OO).

#### Paradigma funcional

- Baseado no conceito matemático de **funções**, tratando computação como avaliação de expressões, evitando (ou minimizando) mudanças de estado e efeitos colaterais.
- Conceitos-chave: funções puras, imutabilidade, funções de alta ordem (funções que recebem/retornam outras funções).
- Exemplos: Haskell, Lisp, Erlang (e traços funcionais em linguagens multiparadigma como Python, JavaScript).

#### Paradigma lógico

- Baseado em **lógica formal**: o programa é um conjunto de fatos e regras, e a execução consiste em fazer **inferências** para responder a consultas.
- Exemplo clássico: **Prolog**.

#### Outros paradigmas frequentemente citados
- **Orientado a eventos**: a execução é guiada por eventos (cliques, mensagens, sinais) — comum em interfaces gráficas.
- **Concorrente/paralelo**: focado em execução simultânea de múltiplas tarefas.

| **Paradigma** | **Ideia Central** | **Abstração Primária** | **Exemplos Notáveis** |
|---|---|---|---|
|**Procedural / Estruturado** |Sequências de instruções agrupadas em sub-rotinas e blocos de controle hierárquicos (`if`, `for`).|Procedimentos / Funções|C, Pascal, Go (em sua base)|
|**Orientado a Objetos (OOP)** |Encapsulamento de dados (estado) e comportamentos (métodos) em entidades autônomas. |Classes, Objetos e Interfaces|Java, C++, C#, Smalltalk|
|**Funcional**|Avaliação de funções matemáticas puras; dados imutáveis; funções como cidadãos de primeira classe (_first-class citizens_). |Expressões, Funções e Recursão|Haskell, Clojure, Elixir, Erlang|
|**Lógico**|Definição de axiomas (fatos) e regras de inferência; o programa busca soluções via unificação e dedução. |Fatos, Regras e Consultas|Prolog, Datalog|
|**Baseado em Restrições** |Modelagem de relações entre variáveis via equações; o resolvedor busca atribuições válidas. |Restrições matemáticas / CSP|MiniZinc, ECLiPSe |
***Observação importante**: muitas linguagens modernas são **multiparadigma** (ex: Python, JavaScript, Scala) — suportam mais de um estilo de programação simultaneamente, então classificar uma linguagem em "um único paradigma" pode ser uma simplificação didática.*
