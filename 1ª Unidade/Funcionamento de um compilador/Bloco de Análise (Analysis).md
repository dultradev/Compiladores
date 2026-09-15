A análise tem como papel **compreender o texto fonte** e verificar sua validade em três níveis crescentes de abstração. É dividida em três sub-etapas, onde vamos entender o papel de cada uma com o código simples:

```C
int resultado = base + 10  ;
```
#### 1. Análise Léxica (Scanner)

>**Objetivo Principal**
>*Identificar os átomos do texto fonte*

- **Papel**: ler o texto fonte caractere por caractere e agrupá-los em unidades significativas chamadas **tokens** (ou lexemas) — palavras-chave, identificadores, operadores, números, símbolos de pontuação, etc.
- Também descarta elementos irrelevantes para as próximas etapas, como espaços em branco e comentários.
- **Exemplo**: 
	- `int` → Palavra-reservada (Tipo)
	- `resultado`→ Identificador (Nome da variável)
	- `=` → Operador de atribuição
	- `base` → Identificador (Nome da variável)
	- `+` → Operador de adição
	- `10` → Literal inteiro
	- `;` → Fim de instrução
	


#### 2. Análise Sintática (Parser)

>**Objetivo Principal**
>*Verificar sequência correta de átomos*

- **Papel**: verificar se a sequência de tokens gerada pela análise léxica **obedece à gramática** (regras estruturais) da linguagem — ou seja, se o texto está bem formado sintaticamente.
- Organiza os tokens em uma estrutura hierárquica, geralmente representada como uma **árvore sintática** (parse tree / árvore de derivação) chamada **Árvore Sintática Abstrata (AST)**, que reflete como os tokens se combinam segundo as regras gramaticais.

- **Exemplo**: 

```
						    Atribuição (=)
						      /          \
						resultado       Adição (+)
						                /        \
						             base        10
```

#### 3. Análise Semântica


>**Objetivo Principal**
>*Verificar coerência de significados*

- **Papel**: verificar se o programa faz **sentido** além da estrutura sintática — checagens que a gramática sozinha não consegue capturar.
- Verificações típicas: compatibilidade de tipos (ex: somar uma string com um inteiro é erro?), declaração prévia de variáveis antes do uso, escopo correto de identificadores, número/tipo correto de argumentos em chamadas de função.
- Também é responsável por **anotar** a árvore sintática com informações adicionais (tipos, referências a símbolos), preparando-a para a próxima fase — por isso às vezes é chamada de gerar uma **árvore sintática anotada/decorada**/**estendida**.
- Com a árvore construída, o analisador semântico verifica se o código realmente **faz sentido**. Ele valida as regras de escopo e os tipos de dados envolvidos.

	**Ações da Análise Semântica no exemplo:**

	1. **Verificação de Escopo:** A variável `base` já foi declarada anteriormente? (Se não, gera um erro de "variável não declarada").
	2. **Verificação de Tipos:** A variável `base` é de um tipo numérico que pode ser somado a um inteiro (`10`)?
	3. **Coerção de Tipo (se necessário):** Se `base` for um número de ponto flutuante (Ex: `float`), o analisador semântico converte o inteiro `10` para `10.0` antes de realizar a soma.

Se tentarmos processar uma linha com um erro de tipo, como tentar somar um texto com um número:

```c
int resultado = "texto" + 10;
```

Veja como cada etapa reage. O comportamento muda completamente:

## 1. Scanner (Análise Léxica)

- O que ele faz: Ele lê tudo normalmente. O scanner não sabe o que é uma soma ou o que o código tenta fazer. Ele apenas identifica que `"texto"` é uma string e `10` é um número.
- Resultado: Passa sem erros. Ele gera os tokens com sucesso.

## 2. Parser (Análise Sintática)

- O que ele faz: Ele verifica a estrutura gramatical: `Variável = Valor + Valor;`. Como a estrutura de uma atribuição com soma está perfeitamente correta na gramática da linguagem, ele monta a árvore normalmente.
- Resultado: Passa sem erros. A Árvore Sintática (AST) é criada.

## 3. Análise Semântica

- O que ele faz: Aqui o compilador tenta validar o significado da operação. Ele olha para a árvore e pergunta: _"Posso usar o operador `+` entre um tipo `String` e um tipo `Int`?"_. No C tradicional, a resposta é não (ou gerará um comportamento de ponteiro indesejado).
- Resultado: O compilador falha aqui! É gerado um Erro Semântico (geralmente uma mensagem de _Type Mismatch_ ou "Incompatibilidade de Tipos").

---

## Resumo do comportamento dos erros

|Tipo de Erro|Exemplo de Código|Quem detecta?|
|---|---|---|
|Léxico|`int re$ultado = 10;` _(caractere inválido `$`)_|Scanner|
|Sintático|`int resultado = base + ;` _(falta o segundo número)_|Parser|
|Semântico|`int resultado = "texto" + 10;` _(operação inválida)_|Análise Semântica|

