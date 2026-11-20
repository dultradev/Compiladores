# Etapa 3 (TWIR) — Tradução da Gramática BNF para Notação de Wirth

**Disciplina:** Compiladores

**Projeto:** Static Checker — Linguagem `BigHole2026-2`

**Etapa:** 3 — `TWIR` (Transformação da Gramática em BNF para Notação de Wirth com Eliminação de Recursão)

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

Entrega da tradução da gramática da linguagem **BigHole2026-2**, previamente convertida para a notação BNF na Etapa 2 (`TBNF`), para o seu equivalente na **notação de Wirth**, aplicando obrigatoriamente a **eliminação de recursão** (direta e indireta), preparando a estrutura sintática para as etapas subsequentes de simplificação (`SIMP`) e geração de autômatos (`BRUT` e `OTIM`).

---

## 2. Simbologia da Notação de Wirth Adotada

| Elemento | Representação em Wirth | Descrição |
| --- | --- | --- |
| **Não-terminal** | `X` | Nome de um não-terminal, sem delimitador (remove `<` e `>`). |
| **Terminal** | `"y"` | Representa um símbolo terminal (inclui palavras reservadas, símbolos reservados e os átomos léxicos `C01`–`C07` no nível sintático). |
| **Agrupamento** | `(z)` | Agrupamento de cadeias. |
| **Iteração** | `{z}` | Iterações arbitrárias (0 ou mais repetições) da cadeia `z`. |
| **Opcionalidade** | `[ z ]` | Construção opcional (0 ou 1 ocorrência de `z`). |
| **Concatenação** | `z t` | Concatenação sequencial de duas cadeias `z` e `t`. |
| **Alternativa** | `z \| t` | Alternativas entre duas cadeias `z` e `t`. |
| **Atribuição** | `=` | Símbolo de atribuição/definição (substitui `::=`). |
| **Delimitador** | `.` | Delimitador de fim de produção. |

---

## 3. Processo de Transformação (BNF → Wirth)

### 3.1. Aplicação da Simbologia Base

Para cada produção da gramática BNF:
- **Atribuição e delimitador:** substitui-se `::=` por `=` e finaliza-se cada regra com `.`.
- **Não-terminais:** removem-se os delimitadores `<` e `>` (ex.: `<DeclarationList>` → `DeclarationList`).
- **Terminais:** todo símbolo terminal (reservados `A01`–`A26`, `B01`–`B22` e átomos `C01`–`C07`) passa a ser delimitado por aspas duplas `"..."` (ex.: `program` → `"program"`, `;` → `";"`, `variable` → `"variable"`).

### 3.2. Transformação de regra de BNF para Wirth

A transformação da notação BNF para Wirth pode ser resumida em 4 situações possíveis:

| # | Tipo de recursão        | Tratamento                                                                                         | Representação  | Aplicação  |
| - | ----------------------- | -------------------------------------------------------------------------------------------------- | -------------- |  -------------- |
| 1 | **Recursão à direita**  | Eliminada, transformando-se em uma chave no início da cadeia.                                      | `a<S>`         |  `a { S }`          |
| 2 | **Recursão à esquerda** | Eliminada, transformando-se em uma chave no final da cadeia.                                       | `<S>b`         | `{ S } b`          |
| 3 | **Sem recursão**        | Mantida entre parênteses no meio da regra ou entre colchetes, caso seja vazia.                     | `(S)` ou `[ε]` | `( S )` ou `[ S ]` |
| 4 | **Recursão ao centro**  | Não eliminada, permanecendo entre parênteses no meio da regra ou entre colchetes, caso seja vazia. | `(S)` ou `[ε]` | `( S )` ou `[ S ]` |

---

## 4. Transformação "Bruta" para Notação de Wirth (Com Eliminação de Recursão)

Nesta primeira passagem, mantêm-se todos os **35 não-terminais** da Etapa 2, traduzindo a simbologia e aplicando as regras gerais de eliminação de recursão e fatoração.

### 4.1. Estrutura do Programa

```wirth
FileProgram = "program" "programName" "declarations" DeclarationList "endDeclarations" [ "functions" FunctionList "endFunctions" ] "endProgram" .
```

### 4.2. Declarações e Especificação de Tipos

```wirth
DeclarationList = DeclarationVar { ";" DeclarationVar } .

DeclarationVar = "varType" TypeSpecification ( ":" VariableList | "[" "]" ":" VariableListVetor ) .

VariableList = "variable" { "," "variable" } .

VariableListVetor = "variable" "[" "intConst" "]" { "," "variable" "[" "intConst" "]" } .

TypeSpecification = "real" | "integer" | "string" | "boolean" | "character" | "void" .
```

### 4.3. Funções e Parâmetros

```wirth
FunctionList = DeclarationFunc { ";" DeclarationFunc } .

DeclarationFunc = "funcType" TypeSpecification ":" "functionName" "(" Parameters ")" Command "endFunction" .

Parameters = ParamTypeList | "?" .

ParamTypeList = ParamType { ";" ParamType } .

ParamType = "paramType" TypeSpecification ":" ParamList .

ParamList = Param { "," Param } .

Param = "variable" [ "[" "intConst" "]" ] .
```

### 4.4. Comandos e Controle de Fluxo

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

### 4.5. Expressões, Operadores e Constantes

```wirth
LogicalExp = ( "(" LogicalExp ")" | AritmExp ) { RelatOperator AritmExp } .

AritmExp = AddExp .

RelatOperator = "<=" | "<" | ">" | ">=" | "==" | "!=" | "#" .

AddExp = Term { AddOper Term } .

AddOper = "-" | "+" .

Term = UnaryExp { MultOper UnaryExp } .

MultOper = "*" | "/" | "%" .

UnaryExp = { "-" } Factor .

Factor = "(" AritmExp ")" | Variable | Constant .

Constant = Numbers | Strings | "true" | "false" .

Numbers = "intConst" | "realConst" .

Strings = "stringConst" | "charConst" .
```

---

## 5. Resumo Comparativo

A tabela abaixo consolida os **35 não-terminais** da gramática segundo as 4 situações de transformação definidas no item 3.2:

| Situação / Métrica | Etapa 2 (`TBNF`) | Etapa 3 (`TWIR` — Bruta) | Não-Terminais Envolvidos |
| --- | --- | --- | --- |
| **Recursão à esquerda** | Presente em 10 regras | Eliminada  | `DeclarationList`, `VariableList`, `VariableListVetor`, `FunctionList`, `ParamTypeList`, `ParamList`, `ListCommand`, `LogicalExp` *(mista)*, `AddExp`, `Term` **(10)** |
| **Recursão à direita** | Presente em 1 regra | Eliminada  | `UnaryExp` **(1)** |
| **Recursão ao centro** | Presente (1 direta mista + 6 indiretas por aninhamento) | Mantida entre `(...)` | *Direta:* `LogicalExp` *(já contada acima)*. *Indireta delimitada:* `Command`, `BlockCommand`, `IfCommand`, `WhileCommand`, `AritmExp`, `Factor` **(6)** |
| **Sem recursão** | 18 regras sem recursão (ou 24 sem recursão direta) | Mantidas com simbologia Wirth, `( )` e `[ ]` | `FileProgram`, `DeclarationVar`, `TypeSpecification`, `DeclarationFunc`, `Parameters`, `ParamType`, `Param`, `ReturnCommand`, `PauseCommand`, `PrintCommand`, `AtribCommand`, `Variable`, `RelatOperator`, `AddOper`, `MultOper`, `Constant`, `Numbers`, `Strings` **(18)** |
| **Qtd. de Não-Terminais** | **35** | **35** | `10 + 1 + 6 + 18 = 35` não-terminais identificados |

