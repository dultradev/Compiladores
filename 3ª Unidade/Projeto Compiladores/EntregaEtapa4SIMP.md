# Etapa 4 (SIMP) — Simplificação e Enxugamento da Gramática em Notação de Wirth

**Disciplina:** Compiladores

**Projeto:** Static Checker — Linguagem `BigHole2026-2`

**Etapa:** 4 — `SIMP` (Simplificação e Enxugamento da Gramática da Linguagem)

---

## Identificação da Entrega

- **Código da Equipe:** `EQ04`
- **Componentes:**
    - Leonardo Santos Magalhães; `leonardo.magalhaes@aln.senaicimatec.edu.br`; `(71) 981379640`
    - Marcos Paulo Teles de Menezes Filho; `marcos.p.filho@aln.senaicimatec.edu.br`; `(71) 992782161`
    - Rafael Guerra de Oliveira; `rafael.g.oliveira@aln.senaicimatec.edu.br`; `(75) 991140115`
    - Rian Albert dos Santos Dultra; `rian.santos@aln.senaicimatec.edu.br`; `(71) 987981802`

---

## 1. Objetivo da Etapa

Entrega da simplificação e enxugamento da gramática da linguagem **BigHole2026-2**, partindo da representação em notação de Wirth obtida na Etapa 3 (`TWIR`), com o objetivo de reduzir a gramática ao **menor número possível de símbolos não-terminais**, preparando-a para a geração dos autômatos equivalentes nas etapas seguintes.

O processo segue dois princípios fundamentais:
1. **Redução de não-terminais:** eliminar todos os não-terminais substituíveis (sem recursão ou pertencentes a ciclos indiretos já representados), preservando apenas o símbolo inicial (axioma) e os **não-terminais auto-recursivos centrais essenciais**.
2. **Simplificação contínua em Wirth:** a cada substituição realizada, verificar se a expressão gerada pode ser reescrita de maneira mais simples e enxuta em notação de Wirth (eliminação de parênteses redundantes, absorção de alternativas redundantes e fatoração/fusão de iterações equivalentes), mantendo integralmente a mesma linguagem gerada.

---

## 2. Análise de Dependências e Seleção dos Não-Terminais Essenciais

Partindo dos **35 não-terminais** definidos na gramática em Wirth da Etapa 3, analisa-se o grafo de dependências entre as regras para classificar cada não-terminal:

### 2.1. Não-Terminais Não-Recursivos (Elimináveis Diretamente)

Não participam de nenhum ciclo sintático e podem ser integralmente substituídos nas regras que os referenciam:
- **Declarações e Tipos (5):** `DeclarationList`, `DeclarationVar`, `VariableList`, `VariableListVetor`, `TypeSpecification`.
- **Funções e Parâmetros (7):** `FunctionList`, `DeclarationFunc`, `Parameters`, `ParamTypeList`, `ParamType`, `ParamList`, `Param`.
- **Comandos simples e auxiliares (5):** `ReturnCommand`, `PauseCommand`, `PrintCommand`, `AtribCommand`, `Variable`.
- **Operadores e constantes (6):** `RelatOperator`, `AddOper`, `MultOper`, `Constant`, `Numbers`, `Strings`.

### 2.2. Não-Terminais em Ciclos Auto-Recursivos Centrais

Existem **três ciclos independentes** de recursão ao centro (delimitada), nos quais a recursão não pode ser eliminada por iteração `{ ... }`:

1. **Ciclo de Comandos:** `Command` → `BlockCommand` → `ListCommand` → `Command`; `Command` → `IfCommand` → `Command`; `Command` → `WhileCommand` → `Command`.
   - **Não-terminal essencial escolhido:** `Command` (centraliza todos os desvios de comandos e já é o símbolo diretamente referenciado em `DeclarationFunc`).
   - **Eliminados por absorção no ciclo (4):** `BlockCommand`, `ListCommand`, `IfCommand`, `WhileCommand`.

2. **Ciclo de Expressões Lógicas:** `LogicalExp` → `"(" LogicalExp ")"`.
   - **Não-terminal essencial escolhido:** `LogicalExp` (possui auto-recursão central direta delimitada por parênteses).

3. **Ciclo de Expressões Aritméticas:** `AritmExp` → `AddExp` → `Term` → `UnaryExp` → `Factor` → `"(" AritmExp ")"`.
   - **Não-terminal essencial escolhido:** `AritmExp` (ponto de entrada das expressões aritméticas, diretamente referenciado por `Command`, `LogicalExp` e `Factor`).
   - **Eliminados por absorção no ciclo (4):** `AddExp`, `Term`, `UnaryExp`, `Factor`.

> **Por que `LogicalExp` e `AritmExp` permanecem separados?**
> Apesar de `LogicalExp` referenciar `AritmExp`, `AritmExp` **não** referencia `LogicalExp`. Na linguagem `BigHole2026-2`, operadores relacionais (`<=`, `<`, `>`, `>=`, `==`, `!=`, `#`) são permitidos apenas em `LogicalExp`, não sendo aceitos dentro de expressões aritméticas, em `print`, em `return` ou no interior de parênteses aritméticos (`Factor`). Fundir ambos os ciclos alteraria a linguagem reconhecida; portanto, ambos são auto-recursivos centrais essenciais e distintos.

### 2.3. Símbolo Inicial da Gramática

- **`FileProgram`**: embora não possua recursão, é o **axioma (símbolo inicial)** da gramática. Ele define a estrutura global do arquivo fonte (`"program" ... "endProgram"`) e não aparece no lado direito de nenhuma produção, não havendo regra superior onde possa ser substituído. Assim, permanece obrigatoriamente como ponto de entrada.

Com isso, os **35 não-terminais** iniciais reduzem-se ao mínimo teórico possível de **4 não-terminais**: `FileProgram`, `Command`, `LogicalExp` e `AritmExp`.

---

## 3. Processo Passo a Passo de Substituição e Reescrita em Wirth

### 3.1. Simplificação das Expressões Aritméticas (`AritmExp`)

Partindo das regras originais da Etapa 3:

```wirth
AritmExp = AddExp .
AddExp = Term { AddOper Term } .
AddOper = "-" | "+" .
Term = UnaryExp { MultOper UnaryExp } .
MultOper = "*" | "/" | "%" .
UnaryExp = { "-" } Factor .
Factor = "(" AritmExp ")" | Variable | Constant .
Variable = "variable" [ "[" "intConst" "]" ] .
Constant = Numbers | Strings | "true" | "false" .
Numbers = "intConst" | "realConst" .
Strings = "stringConst" | "charConst" .
```

**Passo 1 — Substituição de `Numbers` e `Strings` em `Constant`:**
Ao substituir `Numbers` e `Strings`, como todas as alternativas resultantes são terminais simples ligados pelo operador `|`, não são necessários parênteses internos de agrupamento:

```wirth
Constant = "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" .
```
*(Verificação Wirth: alternância plana, forma mais simples obtida.)*

**Passo 2 — Substituição de `Variable` e `Constant` em `Factor`:**
Ao substituir `Variable` e `Constant` em `Factor`, também se preserva uma única alternância plana sem parênteses redundantes:

```wirth
Factor = "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" .
```
*(Verificação Wirth: todas as 8 alternativas iniciam por construções distintas; forma mais simples obtida.)*

**Passo 3 — Substituição de `Factor` em `UnaryExp`:**
Como `Factor` é uma alternância precedida de `{ "-" }`, o agrupamento `(...)` ao redor de `Factor` é obrigatório:

```wirth
UnaryExp = { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" ) .
```
*(Verificação Wirth: `{ "-" }` seguido do agrupamento dos fatores; não há simplificação adicional possível neste nível.)*

**Passo 4 — Substituição de `MultOper` e `UnaryExp` em `Term`:**
Substituindo `MultOper = "*" | "/" | "%"` (com parênteses por estar concatenado à direita) e `UnaryExp` (sem necessidade de parênteses externos adicionais em torno de `{ "-" } ( ... )`, pois a concatenação tem precedência maior que `|`):

```wirth
Term = { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" )
       { ( "*" | "/" | "%" ) { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" ) } .
```
*(Verificação Wirth: parênteses externos redundantes em torno de `UnaryExp` já foram omitidos.)*

**Passo 5 — Substituição de `AddOper` e `Term` em `AddExp`, e de `AddExp` em `AritmExp`:**
Sendo `U` a cadeia correspondente a `UnaryExp`, `M = "*" | "/" | "%"` e `A = "-" | "+"`, a substituição direta de `Term` em `AddExp` e depois em `AritmExp` produz:

```wirth
AritmExp = U { ( M ) U } { ( A ) U { ( M ) U } } .
```

**Verificação de Reescrita Mais Simples em Wirth:**
Analisando a estrutura `U { M U } { A U { M U } }`:
- Qualquer cadeia gerada começa com uma ocorrência de `U`, seguida por zero ou mais ocorrências de `M U` e de `A U` em qualquer sequência.
- Pela equivalência formal de linguagens regulares `U (M U)* (A U (M U)*)* = U ((M | A) U)*`, toda sequência que alterna operadores de `M = {"*", "/", "%"}` e de `A = {"-", "+"}` entre operandos `U` é exatamente a mesma gerada por `U { ( M | A ) U }`:
  - Se não houver operador de `A`, ambos geram `U { M U }`.
  - Se houver operadores de `A`, particionar a sequência imediatamente antes de cada operador de `A` produz exatamente o formato `U { M U } { A U { M U } }`.
- Nota-se ainda que o operador unário `"-"` continua preservado dentro de `U` (`{ "-" } ( Factor )`), enquanto o operador `"+"` continua proibido como unário (ex.: `a * + b` continua inválido e `a * - b` continua válido), preservando **100%** da linguagem original.

Portanto, a regra de `AritmExp` simplifica-se pela metade, eliminando a duplicação do bloco multiplicativo e de `U`:

```wirth
AritmExp = { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" )
           { ( "*" | "/" | "%" | "-" | "+" ) { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" ) } .
```

---

### 3.2. Simplificação das Expressões Lógicas (`LogicalExp`)

Partindo das regras originais da Etapa 3:

```wirth
LogicalExp = ( "(" LogicalExp ")" | AritmExp ) { RelatOperator AritmExp } .
RelatOperator = "<=" | "<" | ">" | ">=" | "==" | "!=" | "#" .
```

**Passo 1 — Substituição de `RelatOperator` em `LogicalExp`:**
Substituindo `RelatOperator` (com agrupamento `(...)` por estar concatenado com `AritmExp` dentro de `{ ... }`):

```wirth
LogicalExp = ( "(" LogicalExp ")" | AritmExp ) { ( "<=" | "<" | ">" | ">=" | "==" | "!=" | "#" ) AritmExp } .
```

*(Verificação Wirth: `AritmExp` não pode ser eliminado nem fundido com `"(" LogicalExp ")"`, pois `LogicalExp` permite expressões sem parênteses externos como `x < 10` ou `x`, e não permite expressões relacionais após `RelatOperator`. Não há reescrita mais simples.)*

---

### 3.3. Simplificação dos Comandos (`Command`)

Partindo das regras originais da Etapa 3:

```wirth
Command = BlockCommand | AtribCommand | PrintCommand | IfCommand | WhileCommand | ReturnCommand | PauseCommand .
BlockCommand = "{" ListCommand "}" .
ListCommand = Command { ";" Command } .
IfCommand = "if" "(" LogicalExp ")" Command [ "else" Command ] "endIf" .
WhileCommand = "while" "(" LogicalExp ")" Command "endWhile" .
ReturnCommand = "return" [ AritmExp ] .
PauseCommand = "break" .
PrintCommand = "print" AritmExp .
AtribCommand = Variable ":=" ( AritmExp | LogicalExp ) .
Variable = "variable" [ "[" "intConst" "]" ] .
```

**Passo 1 — Substituição de `ListCommand` em `BlockCommand`:**

```wirth
BlockCommand = "{" Command { ";" Command } "}" .
```
*(Verificação Wirth: forma mínima para um ou mais comandos separados por `";"`, delimitados por chaves.)*

**Passo 2 — Substituição de `Variable` em `AtribCommand` e Verificação de Simplificação:**
Substituindo `Variable = "variable" [ "[" "intConst" "]" ]` em `AtribCommand`:

```wirth
AtribCommand = "variable" [ "[" "intConst" "]" ] ":=" ( AritmExp | LogicalExp ) .
```

**Verificação de Reescrita Mais Simples em Wirth:**
Observando a regra de `LogicalExp = ( "(" LogicalExp ")" | AritmExp ) { ( ... ) AritmExp }`:
- Escolhendo a segunda alternativa do primeiro agrupamento (`AritmExp`) e zero repetições da iteração `{ ... }`, tem-se que **toda derivação válida de `AritmExp` já é também uma derivação válida de `LogicalExp`** (`L(AritmExp) ⊆ L(LogicalExp)`).
- Logo, `L(AritmExp | LogicalExp) = L(AritmExp) ∪ L(LogicalExp) = L(LogicalExp)`.
- A alternância `( AritmExp | LogicalExp )` é redundante e gera ambiguidade desnecessária, podendo ser simplificada diretamente para `LogicalExp`:

```wirth
AtribCommand = "variable" [ "[" "intConst" "]" ] ":=" LogicalExp .
```

**Passo 3 — Substituição dos demais comandos em `Command`:**
Substituindo `BlockCommand`, `AtribCommand`, `PrintCommand`, `IfCommand`, `WhileCommand`, `ReturnCommand` e `PauseCommand` em `Command`:

```wirth
Command = "{" Command { ";" Command } "}"
        | "variable" [ "[" "intConst" "]" ] ":=" LogicalExp
        | "print" AritmExp
        | "if" "(" LogicalExp ")" Command [ "else" Command ] "endIf"
        | "while" "(" LogicalExp ")" Command "endWhile"
        | "return" [ AritmExp ]
        | "break" .
```
*(Verificação Wirth: os 7 ramos iniciam com terminais mutuamente exclusivos — `"{"`, `"variable"`, `"print"`, `"if"`, `"while"`, `"return"` e `"break"` — e todas as ocorrências recursivas de `Command` são centrais e delimitadas; não há fatoração ou redução adicional.)*

---

### 3.4. Simplificação da Estrutura do Programa (`FileProgram`)

Partindo das regras originais da Etapa 3:

```wirth
FileProgram = "program" "programName" "declarations" DeclarationList "endDeclarations" [ "functions" FunctionList "endFunctions" ] "endProgram" .
DeclarationList = DeclarationVar { ";" DeclarationVar } .
DeclarationVar = "varType" TypeSpecification ( ":" VariableList | "[" "]" ":" VariableListVetor ) .
VariableList = "variable" { "," "variable" } .
VariableListVetor = "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } .
TypeSpecification = "real" | "integer" | "string" | "boolean" | "character" | "void" .
FunctionList = DeclarationFunc { ";" DeclarationFunc } .
DeclarationFunc = "funcType" TypeSpecification ":" "functionName" "(" Parameters ")" Command "endFunction" .
Parameters = ParamTypeList | "?" .
ParamTypeList = ParamType { ";" ParamType } .
ParamType = "paramType" TypeSpecification ":" ParamList .
ParamList = Param { "," Param } .
Param = "variable" [ "[" "intConst" "]" ] .
```

**Passo 1 — Substituição de `VariableList`, `VariableListVetor` e `TypeSpecification` em `DeclarationVar`:**

```wirth
DeclarationVar = "varType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ( ":" "variable" { "," "variable" } | "[" "]" ":" "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } ) .
```
*(Verificação Wirth: a separação entre lista de variáveis escalares após `":"` e lista de vetores dimensionados após `"[" "]" ":"` é obrigatória para não aceitar escalares em declaração de vetor ou vetores em declaração escalar; o prefixo comum `"varType" TypeSpecification` já se encontra fatorado.)*

**Passo 2 — Substituição de `Param` em `ParamList`, de `ParamList` e `TypeSpecification` em `ParamType`, e de `ParamType` em `ParamTypeList`:**

- `ParamList` com `Param`:
```wirth
ParamList = "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } .
```
- `ParamType` com `TypeSpecification` e `ParamList`:
```wirth
ParamType = "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } .
```
- `ParamTypeList` com `ParamType` (sem parênteses externos redundantes em torno de `ParamType`):
```wirth
ParamTypeList = "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] }
                { ";" "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } } .
```

**Passo 3 — Substituição de `ParamTypeList` em `Parameters` e de `Parameters` e `TypeSpecification` em `DeclarationFunc`:**
- Em `Parameters = ParamTypeList | "?"`, não são necessários parênteses internos em volta de `ParamTypeList` na alternância.
- Ao substituir `Parameters` dentro de `"(" Parameters ")"` em `DeclarationFunc`, mantêm-se apenas os parênteses de agrupamento que delimitam a alternância `( ... | "?" )` entre os terminais `"("` e `")"`:

```wirth
DeclarationFunc = "funcType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "functionName" "("
                  ( "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] }
                    { ";" "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } }
                  | "?" )
                  ")" Command "endFunction" .
```

**Passo 4 — Substituição de `DeclarationVar` em `DeclarationList`, de `DeclarationFunc` em `FunctionList`, e de ambos em `FileProgram`:**
Substituindo `DeclarationList = DeclarationVar { ";" DeclarationVar }` e `FunctionList = DeclarationFunc { ";" DeclarationFunc }` em `FileProgram`, sem parênteses externos supérfluos ao redor das cadeias de `DeclarationVar` e `DeclarationFunc`:

```wirth
FileProgram = "program" "programName" "declarations"
              "varType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ( ":" "variable" { "," "variable" } | "[" "]" ":" "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } )
              { ";" "varType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ( ":" "variable" { "," "variable" } | "[" "]" ":" "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } ) }
              "endDeclarations"
              [ "functions"
                "funcType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "functionName" "("
                ( "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] }
                  { ";" "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } }
                | "?" )
                ")" Command "endFunction"
                { ";" "funcType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "functionName" "("
                  ( "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] }
                    { ";" "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } }
                  | "?" )
                  ")" Command "endFunction" }
              "endFunctions" ]
              "endProgram" .
```
*(Verificação Wirth: regra totalmente livre de não-terminais auxiliares, referenciando unicamente `Command`.)*

---

## 4. Gramática Simplificada Final em Notação de Wirth

Ao final do processo de simplificação, toda a linguagem **BigHole2026-2** passa a ser descrita por apenas **4 regras** (`FileProgram`, `Command`, `LogicalExp` e `AritmExp`):

```wirth
FileProgram = "program" "programName" "declarations"
              "varType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ( ":" "variable" { "," "variable" } | "[" "]" ":" "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } )
              { ";" "varType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ( ":" "variable" { "," "variable" } | "[" "]" ":" "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } ) }
              "endDeclarations"
              [ "functions"
                "funcType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "functionName" "("
                ( "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] }
                  { ";" "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } }
                | "?" )
                ")" Command "endFunction"
                { ";" "funcType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "functionName" "("
                  ( "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] }
                    { ";" "paramType" ( "real" | "integer" | "string" | "boolean" | "character" | "void" ) ":" "variable" [ "[" "intConst" "]" ] { "," "variable" [ "[" "intConst" "]" ] } }
                  | "?" )
                  ")" Command "endFunction" }
              "endFunctions" ]
              "endProgram" .

Command = "{" Command { ";" Command } "}"
        | "variable" [ "[" "intConst" "]" ] ":=" LogicalExp
        | "print" AritmExp
        | "if" "(" LogicalExp ")" Command [ "else" Command ] "endIf"
        | "while" "(" LogicalExp ")" Command "endWhile"
        | "return" [ AritmExp ]
        | "break" .

LogicalExp = ( "(" LogicalExp ")" | AritmExp ) { ( "<=" | "<" | ">" | ">=" | "==" | "!=" | "#" ) AritmExp } .

AritmExp = { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" )
           { ( "*" | "/" | "%" | "-" | "+" ) { "-" } ( "(" AritmExp ")" | "variable" [ "[" "intConst" "]" ] | "intConst" | "realConst" | "stringConst" | "charConst" | "true" | "false" ) } .
```

---

## 5. Resumo Comparativo da Simplificação

| Métrica / Categoria | Etapa 3 (`TWIR`) | Etapa 4 (`SIMP`) | Observações e Não-Terminais Envolvidos |
| --- | --- | --- | --- |
| **Qtd. de Não-Terminais** | **35** | **4** | Redução de **31 não-terminais** (**88,57%** de enxugamento) |
| **Axioma (Símbolo Inicial)** | `FileProgram` | `FileProgram` | Mantido por ser o ponto de entrada da gramática (absorveu 12 não-terminais de declarações, tipos, funções e parâmetros) |
| **Auto-Recursivo Central — Comandos** | `Command`, `BlockCommand`, `IfCommand`, `WhileCommand` (+ 6 auxiliares) | `Command` | `Command` manteve-se como representante essencial do ciclo (absorveu 9 não-terminais de comandos) |
| **Auto-Recursivo Central — Lógico** | `LogicalExp` (+ `RelatOperator`) | `LogicalExp` | Mantido pela auto-recursão central `"(" LogicalExp ")"` (absorveu `RelatOperator`) |
| **Auto-Recursivo Central — Aritmético** | `AritmExp`, `Factor` (+ 9 auxiliares) | `AritmExp` | `AritmExp` manteve-se como representante essencial do ciclo `"(" AritmExp ")"` (absorveu 10 não-terminais e simplificou a cadeia `Term`/`AddExp`) |
| **Principais Reescritas em Wirth** | — | 2 reduções algébricas + limpeza de agrupamentos | `AritmExp | LogicalExp` → `LogicalExp` em atribuições; `U { M U } { A U { M U } }` → `U { ( M | A ) U }` em `AritmExp`; remoção de `(...)` redundantes |
