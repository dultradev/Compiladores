# Etapa 2 (TBNF) — Tradução da Gramática Formal para Notação BNF

**Disciplina:** Compiladores

**Projeto:** Static Checker — Linguagem `BigHole2026-2`

**Etapa:** 2 — `TBNF` (Transformação da Gramática do Projeto para Notação BNF)

---

## Identificação da Entrega

- **Código da Equipe:** `E04`
- **Componentes:**
    - Leonardo Santos Magalhães; `leonardo.magalhaes@aln.senaicimatec.edu.br`; `(71) 981379640`
    - Marcos Paulo Teles de Menezes Filho; `marcos.p.filho@aln.senaicimatec.edu.br`; `(71) 992782161`
    - Rafael Guerra de Oliveira; `rafael.g.oliveira@aln.senaicimatec.edu.br`; `(75) 991140115`
    - Rian Albert dos Santos Dultra; `rian.santos@aln.senaicimatec.edu.br`; `(71) 987981802`

---

## 1. Objetivo da Etapa

Entrega da conversão da gramática da linguagem **BigHole2026-2**, da notação formal `G = (V, Σ, P, FileProgram)` na definição da gramática formal da Linguagem, para a notação BNF.

---

## 2. Convenções da Notação BNF Adotada

| Elemento | Representação BNF |
| --- | --- |
| **Símbolo Não-terminal** | <X> representa um simbolo X não terminal |
| **Símbolo Terminal** | X representa um simbolo X terminal |
| **Símbolo de substituição** | ::= representa uma substituição possível de cadeias |
| **Separador das cadeias** | separa as diversas cadeias que definem um simbolo (ou) |

---

## 3. Notas de Padronização e Ajustes Tipográficos

Durante a tradução da gramática Formal da Linguagem, foram corrigidas inconsistências puramente tipográficas identificadas no texto de origem, mantendo-se integralmente a semântica da gramática:

| # | Elemento | Relato da execução e premissas adotadas |
| --- | --- | --- |
| 1 | `endDeclarations` **(**`A06`**)** | Foi adotada a grafia `"endDeclarations"` como padrão, considerando a tabela oficial de palavras reservadas do Apêndice A (`A06`) como referência para corrigir as divergências encontradas na gramática. |
| 2 | `<AritmExp>` | Foi considerada `<AritmExp>` como a nomenclatura correta, uma vez que essa grafia é utilizada consistentemente nas produções de `P`, diferentemente da ocorrência isolada de `AritmEx` em `V`. |
| 3 | **Espaçamento em** `<ParamType>` | Foi presumida a separação entre o terminal `paramType`, o não-terminal `<TypeSpecification>` e o símbolo `:`, interpretando a aglutinação encontrada como um erro tipográfico. |
| 4 | **Espaçamento em** `<DeclarationVar>` | Foi considerado que o símbolo `:` constitui um terminal independente, sendo separado do não-terminal `<TypeSpecification>` devido à inconsistência de formatação original. |
| 5 | **Separação de metassímbolos de conjunto** | Foi interpretada a vírgula ao final de cada linha como delimitador das produções de `P`, diferenciando-a do terminal `","` (`B02`). Também foi considerada indevida a duplicação do símbolo `:` nos conjuntos `V` e `Σ`. |
| 6 | **Ciclo unitário preservado** | Foi mantido o ciclo entre `<Command>` e `<BlockCommand>`, por estar presente na gramática original. Considerou-se que sua resolução pertence às etapas posteriores de simplificação (`TWIR`/`SIMP`), não sendo realizada nesta tradução para BNF. |

---

## 4. Gramática da Linguagem BigHole2026-2 em Notação BNF

### 4.1. Estrutura do Programa

```bnf
<FileProgram> ::= program programName declarations <DeclarationList> endDeclarations functions <FunctionList> endFunctions endProgram | program programName declarations <DeclarationList> endDeclarations endProgram
```

### 4.2. Declarações e Especificação de Tipos

```bnf
<DeclarationList> ::= <DeclarationList> ; <DeclarationVar> | <DeclarationVar>

<DeclarationVar> ::= varType <TypeSpecification> : <VariableList> | varType <TypeSpecification> [ ] : <VariableListVetor>

<VariableList> ::= <VariableList> , variable | variable

<VariableListVetor> ::= <VariableListVetor> , variable [ intConst ] | variable [ intConst ]

<TypeSpecification> ::= real | integer | string | boolean | character | void
```

### 4.3. Funções e Parâmetros

```bnf
<FunctionList> ::= <FunctionList> ; <DeclarationFunc> | <DeclarationFunc>

<DeclarationFunc> ::= funcType <TypeSpecification> : functionName ( <Parameters> ) <Command> endFunction

<Parameters> ::= <ParamTypeList> | ?

<ParamTypeList> ::= <ParamTypeList> ; <ParamType> | <ParamType>

<ParamType> ::= paramType <TypeSpecification> : <ParamList>

<ParamList> ::= <ParamList> , <Param> | <Param>

<Param> ::= variable | variable [ intConst ]
```

### 4.4. Comandos e Controle de Fluxo

```bnf
<Command> ::= <BlockCommand> | <AtribCommand> | <PrintCommand> | <IfCommand> | <WhileCommand> | <ReturnCommand> | <PauseCommand>

<BlockCommand> ::= { <ListCommand> } | <Command>

<ListCommand> ::= <ListCommand> ; <Command> | <Command>

<IfCommand> ::= if ( <LogicalExp> ) <Command> endIf | if ( <LogicalExp> ) <Command> else <Command> endIf

<WhileCommand> ::= while ( <LogicalExp> ) <Command> endWhile

<ReturnCommand> ::= return | return <AritmExp>

<PauseCommand> ::= break

<PrintCommand> ::= print <AritmExp>

<AtribCommand> ::= <Variable> := <AritmExp> | <Variable> := <LogicalExp>

<Variable> ::= variable | variable [ intConst ]
```

### 4.5. Expressões, Operadores e Constantes

```bnf
<LogicalExp> ::= <LogicalExp> <RelatOperator> <AritmExp> | ( <LogicalExp> ) | <AritmExp>

<AritmExp> ::= <AddExp>

<RelatOperator> ::= <= | < | > | >= | == | != | #

<AddExp> ::= <AddExp> <AddOper> <Term> | <Term>

<AddOper> ::= - | +

<Term> ::= <Term> <MultOper> <UnaryExp> | <UnaryExp>

<MultOper> ::= * | / | %

<UnaryExp> ::= - <UnaryExp> | <Factor>

<Factor> ::= ( <AritmExp> ) | <Variable> | <Constant>

<Constant> ::= <Numbers> | <Strings> | true | false

<Numbers> ::= intConst | realConst

<Strings> ::= stringConst | charConst
```

---

## 5. Mapeamento e Inventário das Regras Sintáticas

| Grupo | Não-Terminais | Qtd. Regras | Qtd. Produções |
| --- | --- | --- | --- |
| **1. Estrutura do Programa** | `<FileProgram>` | 1 | 2 |
| **2. Declarações e Tipos** | `<DeclarationList>`, `<DeclarationVar>`, `<VariableList>`, `<VariableListVetor>`, `<TypeSpecification>` | 5 | 14 |
| **3. Funções e Parâmetros** | `<FunctionList>`, `<DeclarationFunc>`, `<Parameters>`, `<ParamTypeList>`, `<ParamType>`, `<ParamList>`, `<Param>` | 7 | 12 |
| **4. Comandos e Fluxo** | `<Command>`, `<BlockCommand>`, `<ListCommand>`, `<IfCommand>`, `<WhileCommand>`, `<ReturnCommand>`, `<PauseCommand>`, `<PrintCommand>`, `<AtribCommand>`, `<Variable>` | 10 | 22 |
| **5. Expressões e Constantes** | `<LogicalExp>`, `<AritmExp>`, `<RelatOperator>`, `<AddExp>`, `<AddOper>`, `<Term>`, `<MultOper>`, `<UnaryExp>`, `<Factor>`, `<Constant>`, `<Numbers>`, `<Strings>` | 12 | 33 |
| **Total** | **35 não-terminais**  | **35** | **83** |

---