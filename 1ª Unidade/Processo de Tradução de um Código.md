O pipeline clássico que transforma um texto-fonte bruto em um processo em execução na memória é conhecido como a **Cadeia de Ferramentas de Construção (_Toolchain_)**.

No modelo canônico de sistemas operacionais e linguagens compiladas (o modelo GCC/Clang e sistemas Unix), esse fluxo é composto por algumas etapas de tradução e geração de artefatos estáticos, finalizando com o carregamento dinâmico na memória física.

```
Texto fonte bruto
     ↓
[PRÉ-PROCESSADOR] (etapa 1 e/ou 2)
     ↓
Texto fonte "puro" (expandido)
     ↓
[Compilador]
     ↓
Código Assembly (ou código objeto, dependendo da implementação)
     ↓
[MONTADOR]
     ↓
Código objeto relocável (módulo objeto)
     ↓
[LINK-EDITOR / LIGADOR]
     ↓
Programa executável
     ↓
[LOADER / CARREGADOR]
     ↓
Programa em execução (na memória)
```

![[Pasted image 20260913164424.png]]
## 1. [[Pré-Processador]]

O pré-processador atua em nível puramente **textual** antes do início da análise léxica formal. Ele não compreende a sintaxe da linguagem (não valida tipos, variáveis ou regras semânticas).

#### Pré-processador 1

- Atua diretamente sobre o **texto fonte bruto**, antes de qualquer análise mais estruturada.
- Trata diretivas mais "textuais"/superficiais: inclusão de arquivos (`#include`), remoção de comentários, tratamento de espaços em branco/formatação.
#### Pré-processador 2 (quando existe como etapa separada)

- Atua sobre o resultado da primeira etapa, realizando transformações mais elaboradas: **expansão de macros** (substituição de identificadores por trechos de código), **compilação condicional** (`#ifdef`, `#ifndef`), resolução de constantes simbólicas (`#define`).
- Em algumas implementações, essas duas etapas são unificadas em um único pré-processador — daí a referência a "1 ou 2": o número de fases pode variar conforme a complexidade da linguagem e da implementação do tradutor.

	**Saída do pré-processador**: um texto fonte "limpo"/expandido, já sem diretivas de pré-processamento, pronto para ser analisado pelo compilador como se fosse o programa "real" escrito pelo desenvolvedor.

## 2. [[Compilador]]

- Recebe o **texto fonte expandido** (já sem diretivas de pré-processamento) e realiza a tradução propriamente dita, passando pelas etapas clássicas: análise léxica → análise sintática → análise semântica → geração de código intermediário → otimização → geração de código.

- **Saída do compilador**: normalmente **código Assembly** (não código de máquina diretamente) — é essa a razão de existir uma etapa de montagem logo em seguida. Em algumas implementações modernas, o compilador pode gerar diretamente código objeto, "pulando" a geração explícita de Assembly, mas conceitualmente a etapa de montagem continua existindo (mesmo que embutida no próprio compilador).

## 3. [[Montador]]

O montador converte as instruções simbólicas do assembly diretamente para o binário da máquina (opcodes e operandos).

- **Entrada:** Arquivo Assembly (`.s`).
    
- **Operações Principais:**
    - Tradução quase 1:1 de mnemônicos (`MOV`, `ADD`, `CALL`) para bytes literais do processador.
    - Resolução de deslocamentos internos dentro das funções.
    - Criação da **Tabela de Símbolos** e da **Tabela de Realocação**: se o código invoca uma função externa (`printf` ou uma função de outro arquivo), o montador não conhece o endereço físico final. Ele insere zeros no lugar do endereço e grava um "aviso de realocação" na tabela dizendo: _"o endereço real deste ponto precisa ser corrigido depois"_.

- **Saída do montador**: um **código objeto relocável** (também chamado de _módulo objeto_).

	- "Relocável" significa que os endereços de memória usados no código ainda **não são absolutos/finais** — eles são relativos, pois o módulo objeto ainda não sabe em que posição de memória vai ser carregado, nem sabe sobre outros módulos com os quais será combinado.
	- Esse módulo objeto também costuma conter referências a símbolos externos (funções/variáveis definidas em outros arquivos ou bibliotecas) que ainda **não foram resolvidas**.

### 4. Link-Editor (Ligador/Linker)

- Recebe **um ou mais módulos objeto** (gerados a partir de diferentes arquivos fonte compilados separadamente) e também **bibliotecas** necessárias (padrão ou de terceiros).

- Principais tarefas:
    - **Resolução de símbolos externos**: conecta chamadas de função/variáveis usadas em um módulo às suas definições em outro módulo ou biblioteca.
    - **Combinação dos módulos objeto** em um único programa.
    - **Realocação**: ajusta os endereços relativos de cada módulo para formar um espaço de endereçamento único e coerente do programa final.

- **Saída do link-editor**: um **programa executável** — mas ainda pode conter alguns endereços que só serão definidos no momento do carregamento (dependendo do sistema).
### 5. Loader (Carregador)

- Recebe o **programa executável** gerado pelo linker.
- Responsável por:
    - Carregar o programa da memória secundária (disco) para a **memória principal (RAM)**.
    - Realizar a **realocação final** dos endereços (ajustando para o endereço real de memória onde o programa foi carregado, especialmente relevante em sistemas com realocação dinâmica).
    - Preparar o ambiente de execução (pilha, heap, registradores iniciais) e transferir o controle da CPU para o ponto de entrada do programa.
- **Saída/resultado**: o programa efetivamente **em execução** na memória.

### Observação: Link-editor x Loader, junto ou separado?

- Em alguns sistemas/textos, o **link-editor** e o **loader** são tratados como **uma única etapa combinada**, chamada de _linking loader_ — ligação e carregamento acontecem juntos, dinamicamente.
- Em outros, são etapas **claramente separadas**: primeiro o linker gera um executável completo em disco; depois, em um momento diferente (quando o programa é de fato executado), o loader entra em ação.
- Essa distinção se relaciona também com o conceito de **bibliotecas estáticas x dinâmicas**: bibliotecas estáticas são resolvidas já na etapa de linkedição; bibliotecas dinâmicas (`.dll`, `.so`) podem ser resolvidas só em tempo de carregamento/execução (linking dinâmico).

### Quadro-síntese do pipeline completo

|Etapa|Entrada|Saída|Responsabilidade principal|
|---|---|---|---|
|Pré-processador (1/2)|Texto fonte bruto|Texto fonte expandido|Resolver diretivas, macros, inclusões|
|Compilador|Texto fonte expandido|Código Assembly|Traduzir alto nível → Assembly|
|Montador|Código Assembly|Código objeto relocável|Traduzir Assembly → código de máquina|
|Link-editor|Módulo(s) objeto + bibliotecas|Programa executável|Resolver símbolos, combinar módulos, realocar|
|Loader|Programa executável|Programa em execução (RAM)|Carregar na memória, realocação final, iniciar execução|