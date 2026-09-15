Nos diagramas clássicos de arquitetura de compiladores (como o do _Livro do Dragão_), o Gerenciador da tabela de Símbolos e as Rotinas de Tratamento de Erros não aparecem como etapas sequenciais lineares, mas sim como **barramentos verticais transversais**:

![[Pasted image 20260914224116.png]]

Ambos operam em conjunto com praticamente **todas** as etapas do pipeline.
### 1. O Gerenciador da Tabela de Símbolos (_Symbol Table Handler_)

>**Objetivo Principal**
>*Gerenciar o repositório tabela de símbolos*


A Tabela de Símbolos é a principal estrutura de dados persistente de um compilador. Seu gerenciador é responsável por registrar informações sobre os identificadores encontrados no código-fonte e responder a consultas ao longo de todo o processo de compilação.

#### Principais Responsabilidades

- **Inserção (_Insert_):** Armazena novos símbolos assim que são declarados.
    
- **Consulta (_Lookup_):** Recupera os atributos de um símbolo previamente inserido (tipo, tamanho, escopo, posição de memória).
    
- **Gestão de Escopos e Visibilidade:** Lida com regras de sombreamento de variáveis (_shadowing_) e resolução estática de blocos léxicos (escopo global, escopo de função, escopo de bloco `if`/`for`).
    
#### Estrutura Interna para Múltiplos Escopos

Em vez de uma única tabela plana, os compiladores utilizam estruturas hierárquicas, como uma **pilha de tabelas hash** ou uma **árvore de escopos**. Cada vez que o compilador entra em um novo bloco `{ ... }`, uma nova tabela é empilhada; ao sair do bloco, a tabela correspondente é desempilhada ou marcada como inativa.

  

```
Escopo Global:        [ total: int, calcular: func ]
                            ▲
                            │ (aponta para o pai)
Escopo da Função:     [ x: float, y: float ]
                            ▲
                            │
Escopo do Bloco IF:   [ temp: int ]
```

Na consulta de um símbolo, a busca começa no escopo mais interno e sobe recursivamente até o escopo global. Se não for encontrado, um erro de identificador indefinido é emitido.

### O Fluxo de Registro (Passo a Passo)

|**Fase**|**Ação do Gerenciador da Tabela**|
|---|---|
|**Léxica**|Insere lexemas recém-descobertos e verifica palavras reservadas da linguagem.|
|**Sintática**|Identifica declarações e cria os nós de escopo correspondentes na árvore.|
|**Semântica**|Preenche tipos, assinaturas de funções e valida compatibilidade de operações.|
|**Otimização**|Consulta o tempo de vida das variáveis e frequência de uso para decisões de inlining.|
|**Geração de Código**|Informa o deslocamento físico (_offset_ na stack) ou registrador alocado para cada variável.|

Quem faz o registro inicial e principal na Tabela de Símbolos é o Scanner (Análise Léxica), trabalhando em conjunto com o Parser (Análise Sintática).

Para você entender o fluxo exato de quem faz o quê quando uma variável nova aparece no código (o tal "texto fonte sem registro algum"), o processo funciona como uma linha de montagem:

#### 1. O Scanner encontra a palavra

Quando o Scanner lê o código caractere por caractere e encontra algo como `POS` ou `TEMPO`, ele identifica que aquilo é um Identificador (um nome).

- Ele cria um token: `<IDENTIFICADOR, "POS">`.
- Em muitas implementações, o próprio Scanner já faz uma busca rápida na Tabela de Símbolos. Se o nome não existir lá, ele cria uma entrada básica, registrando apenas o nome da variável.

#### 2. O Parser preenche as regras gramaticais

O Scanner não sabe o que a variável faz; ele só sabe o nome dela. Quando esse token chega ao Parser, este analisa o contexto da frase. Ao perceber que o código é `float POS := ...`, o Parser entende: _"Opa, isso é uma declaração de variável!"_.

- O Parser adiciona informações cruciais à tabela: o tipo (que é `float`) e o escopo (se ela é global ou local daquela função).

#### 3. O Analisador Semântico valida e usa

Quando o código chega na Análise Semântica, a tabela já está populada. O analisador semântico vai até ela para checar as regras. Se ele encontrar outra linha usando `base`, ele olha na tabela: _"Deixa eu ver... a variável `POS` está registrada? Sim. Qual o tipo dela? `float`. Beleza, a conta está certa"_.
#### Resumo das responsabilidades na Tabela de Símbolos:

- **Scanner:** Descobre o nome da variável e faz o cadastro inicial (insere o texto do identificador).
- **Parser:** Define a estrutura e contexto (insere o tipo de dado, escopo e tamanho na memória).
- **Analisador Semântico:** Faz a consulta e validação (checa se o que está registrado bate com as regras da linguagem).

A Tabela de Símbolos não é criada por um único módulo, mas sim alimentada e consultada por todas as etapas do compilador do início ao fim do processo.

---

### 2. Rotinas de Tratamento de Erros (_Error Handler_)


>**Objetivo Principal**
>*Controlar a ocorrência de erros*


O objetivo central do manipulador de erros não é apenas interromper a compilação, mas **detectar a falha, isolar sua causa exata, reportar uma mensagem diagnóstica precisa e tentar se recuperar** para continuar analisando o restante do arquivo em busca de erros adicionais.

#### Categorias de Erros por Fase

- **Erros Léxicos:** Caracteres inválidos não reconhecidos pelo alfabeto da linguagem (ex.: `let @x = 10;`) ou strings/comentários literais que nunca são fechados.
    
- **Erros Sintáticos:** Quebra das regras gramaticais livres de contexto (ex.: parênteses desbalanceados, operadores consecutivos como `a ++ * b`, ausência de ponto e vírgula).
    
- **Erros Semânticos:** Código sintaticamente correto, mas semanticamente inconsistente (ex.: usar variável não declarada, tentar somar uma struct a uma string, incompatibilidade de tipos ou chamar função com quantidade errada de parâmetros).
    
#### Estratégias de Recuperação de Erros (_Error Recovery_)

Quando o analisador sintático/semântico encontra um erro, ele aplica estratégias formais para não "travar" imediatamente:
  

1. **Modo Pânico (_Panic Mode_):**
    
    - A abordagem mais comum e robusta. Ao encontrar uma discrepância, o parser descarta tokens sucessivos até encontrar um **token de sincronização** pré-definido (como `;`, `}`, ou `end`). A partir dali, retoma a análise no próximo comando.
        
2. **Recuperação em Nível de Frase (_Phrase-Level Recovery_):**
    
    - O compilador tenta corrigir o erro localmente inserindo um token faltante provável (ex.: injetar virtualmente um `;` ausente ao final da linha) e continua a compilação com um aviso (_warning_).
        
3. **Produções de Erro (_Error Productions_):**
    
    - Gramáticas aumentadas que incluem deliberadamente regras sintáticas para erros frequentes cometidos por programadores. Ao casar com uma dessas produções, o compilador emite um diagnóstico altamente customizado em vez de um genérico "erro de sintaxe".
        
### Como Ambos se Conectam

O tratamento de erros depende diretamente do gerenciador da tabela de símbolos:

- Para emitir a mensagem `"Variável 'TEMPO' não declarada na linha 14, coluna 8"`, o analisador consulta a tabela e constata que o símbolo não existe no escopo ativo.
    
- Para evitar uma cascata de dezenas de mensagens de erro artificiais após a primeira falha, o manipulador de erros pode inserir temporariamente um **tipo coringa/falso** (frequentemente chamado de `type_error`) na tabela de símbolos para aquele identificador. Assim, o analisador semântico ignora verificações futuras daquela mesma variável, evitando poluir o terminal do desenvolvedor com erros repetidos.