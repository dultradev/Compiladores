Um **tradutor** é qualquer **pessoa, máquina ou programa** que recebe como entrada um **texto fonte** expresso em uma **linguagem fonte** e gera como saída um texto objeto equivalente, escrito em uma linguagem objeto, preservando estritamente a equivalência textual (mesmo significado nas duas linguagens) e traduzindo apenas informações relevantes. 

| Elemento         | Descrição                                      |
| ---------------- | ---------------------------------------------- |
| Texto fonte      | A entrada — o que será traduzido               |
| Linguagem fonte  | A linguagem em que o texto fonte foi escrito   |
| Texto objeto     | A saída — o resultado da tradução              |
| Linguagem objeto | A linguagem em que o texto objeto será escrito |
Qualquer tradutor formal opera sob um contrato fundamental: **preservação semântica**. Se a entrada produz determinado resultado sob certas condições, a saída deve produzir exatamente o mesmo resultado lógico.

**a) Equivalência de significados**

- O texto objeto deve representar, na linguagem objeto, o **mesmo sentido/comportamento** que o texto fonte tinha na linguagem fonte.
- Em compiladores, isso normalmente significa: o programa objeto, quando executado, deve produzir o mesmo resultado/comportamento que o programa fonte "pretendia" ter.
- É essa equivalência que garante que a tradução seja **confiável** — sem ela, o tradutor estaria simplesmente gerando outro texto, não uma tradução de fato.

**b) Preservação de informações relevantes**

- Nem toda informação do texto fonte precisa (ou consegue) ser preservada literalmente — mas as informações **relevantes** para o significado devem ser mantidas ou adaptadas de forma equivalente.
- Informações irrelevantes para o objetivo da tradução podem ser descartadas (ex: comentários de código-fonte geralmente não são "traduzidos" para o código objeto, pois não afetam o comportamento).
- Essa característica ajuda a entender por que compiladores podem, por exemplo, remover código morto ou otimizar estruturas: eles preservam o significado relevante (comportamento), não a forma literal.

### Conjunto dos Compiladores

- Nem todo tradutor é objeto de estudo da disciplina de Compiladores. É preciso delimitar esse subconjunto:

- O estudo de **compiladores** foca em tradutores cuja:
    - **Linguagem fonte** é uma linguagem de programação (formal, com sintaxe e semântica bem definidas), geralmente de alto ou médio nível.
    - **Processo de tradução** segue etapas formais e bem estabelecidas (análise léxica → análise sintática → análise semântica → geração de código intermediário → otimização → geração de código objeto).
- Isso exclui, por exemplo:
    - Tradutores de linguagem natural (português → inglês), que lidam com ambiguidade e contexto de forma muito diferente.
    - Conversores de formato simples que não envolvem análise sintática/semântica de uma linguagem formal (embora alguns "filtros" tenham zonas cinzentas, como veremos depois).

Essa delimitação é importante porque justifica por que a disciplina estuda profundamente conceitos como gramáticas formais, autômatos, análise léxica e sintática — são as ferramentas teóricas necessárias para lidar com **linguagens de programação** especificamente.