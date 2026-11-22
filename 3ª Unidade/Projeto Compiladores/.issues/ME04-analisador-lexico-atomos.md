# [ME04] Analisador Léxico (`LexicalAnalyzer`) — Reconhecimento de Átomos e Truncagem

> **Milestone:** `M2: Núcleo Léxico & Símbolos`  
> **Labels:** `macroentrega`, `lexico`, `core`, `prioridade:alta`  
> **Dependências:** `ME01`, `ME02`, `ME03`  
> **Bloqueia:** `ME05`

---

## 1. Objetivo

Implementar o autômato completo do `LexicalAnalyzer` (implementando `ILexicalAnalyzer`), capaz de formar todos os átomos da linguagem **BigHole2026-2** sob demanda (um átomo por chamada de `proximoToken(ContextoEscopo contexto)`), aplicando **Maximal Munch**, **insensibilidade a caixa**, **limite de 30 caracteres válidos com truncagem coerente** e **integração direta com `ReservedTable` e `ISymbolTableManager`**.

---

## 2. Padrões Léxicos da Especificação (Apêndice C)

| Classe | Código | Padrão Formal (Apêndice C) | Observações Críticas |
| :--- | :---: | :--- | :--- |
| Palavras Reservadas | `A01`–`A26` | Consultadas na `ReservedTable` após caixa alta | Precedência sobre identificadores |
| Símbolos/Operadores | `B01`–`B22` | 1 ou 2 caracteres (`:=`, `<=`, `>=`, `==`, `!=`, `#`, etc.) | Maximal Munch (`:=` > `:`, `<=` > `<`, `>=` > `>`) |
| `variable` | `C01` | `(<letra> \| _) (<letra> \| <digito> \| _)*` | **Aceita `_` no início e no corpo!** |
| `functionName` | `C02` | `<letra> (<letra> \| <digito>)*` | Apenas quando contexto = `DECLARACAO_FUNCAO` (sem `_`) |
| `programName` | `C03` | `<letra> (<letra> \| <digito>)*` | Apenas quando contexto = `GLOBAL_PROGRAM` (sem `_`) |
| `stringConst` | `C04` | `"` `(<letra> \| <branco> \| <digito> \| $ \| _ \| .)+` `"` | Aspas contam nos 30 chars; truncar força `"` na pos. 30 |
| `charConst` | `C05` | `'` `<letra>` `'` | Exatamente 1 letra entre aspas simples |
| `intConst` | `C06` | `<digito>+` | Truncado em 30 dígitos |
| `realConst` | `C07` | `<digito>+ . <digito>+ [ (e\|E) [+ \| -] <digito>+ ]` | Truncagem em 30 chars deve manter número sintaticamente válido |

---

## 3. Sub-Issues

- [ ] **ME04.1** — Implementar reconhecimento de Palavras Reservadas (`A01`–`A26`) e Identificadores (`C01`, `C02`, `C03`)
- [ ] **ME04.2** — Implementar reconhecimento de Símbolos e Operadores Reservados (`B01`–`B22`) com Maximal Munch
- [ ] **ME04.3** — Implementar reconhecimento de Constantes Numéricas (`C06` `intConst`, `C07` `realConst`) e truncagem coerente
- [ ] **ME04.4** — Implementar reconhecimento de Constantes Textuais (`C04` `stringConst`, `C05` `charConst`) e truncagem com fechamento forçado
- [ ] **ME04.5** — Integrar `LexicalAnalyzer` com `ISymbolTableManager`, contagem pré/pós-truncagem e suíte completa de testes léxicos

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME04.1` — Palavras Reservadas e Identificadores (`A01`–`A26`, `C01`, `C02`, `C03`)
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Implementar o ramo do scanner acionado quando o primeiro caractere válido for `<letra>` ou `_`:
  - **Normalização:** Converter letras imediatamente para `UPPERCASE` (`Character.toUpperCase`).
  - **Diferenciação por contexto (`ContextoEscopo`):**
    - Em `GLOBAL_PROGRAM` (esperando `C03`) ou `DECLARACAO_FUNCAO` (esperando `C02`): se iniciar com `<letra>`, consumir apenas `<letra>` e `<digito>` (um `_` atua como delimitador nesse padrão). Caso o lexema não seja palavra reservada, classificar como `C03` ou `C02`.
    - Em `CORPO_INSTRUCOES` (ou se iniciar com `_`): consumir `<letra>`, `<digito>` e `_` (padrão `variable` — `C01`).
  - **Verificação na `ReservedTable`:** Após coletar o lexema (até 30 chars válidos) e consumir todos os caracteres restantes até o delimitador, verificar se o lexema truncado está na `ReservedTable`:
    - Se sim → retorna `Token(lexeme, codigoA, null, linhaInicial)` (**não** registra na Tabela de Símbolos).
    - Se não → registra na `ISymbolTableManager` com código `C01`/`C02`/`C03` e retorna `Token` com o índice retornado.
  - **Truncagem (`TC02`):** Se o identificador tiver 48 caracteres válidos, armazenar apenas os 30 primeiros, continuar consumindo os 18 restantes até o delimitador, e registrar `qtdCharsAntesTrunc = 48`, `qtdCharDepoisTrunc = 30`.
- **Critérios de Aceite:**
  - [ ] `program`, `PROGRAM`, `PrOgRaM` retornam `Token("PROGRAM", "A19", null, linha)`.
  - [ ] `_temp_1` é reconhecido como `C01` (`_TEMP_1`).
  - [ ] `va@r1` (com caractere inválido `@`) é reconhecido como `VAR1` (`C01`) com `qtdCharsAntesTrunc = 4`.
  - [ ] Identificador com 48 caracteres válidos gera lexema de 30 chars, consome o restante sem vazar lixo para o próximo token, e registra `48` / `30` na Tabela de Símbolos (`TC02`).

---

### Sub-Issue `ME04.2` — Símbolos e Operadores Reservados (`B01`–`B22`) com Maximal Munch
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Implementar o reconhecimento de todos os operadores e delimitadores simples e compostos:
  - **Simples de 1 caractere:** `;` (`B01`), `,` (`B02`), `?` (`B05`), `(` (`B06`), `)` (`B07`), `[` (`B08`), `]` (`B09`), `{` (`B10`), `}` (`B11`), `+` (`B12`), `-` (`B13`), `*` (`B14`), `/` (`B15` — já garantido que não é `//` ou `/*`), `%` (`B16`), `#` (`B18`).
  - **Compostos com lookahead de 1 caractere válido (filtrando inválidos intermediários? definir comportamento!):**
    - `:` → se próximo for `=` → `:=` (`B04`); senão devolve lookahead e emite `:` (`B03`).
    - `<` → se próximo for `=` → `<=` (`B20`); senão devolve lookahead e emite `<` (`B19`).
    - `>` → se próximo for `=` → `>=` (`B22`); senão devolve lookahead e emite `>` (`B21`).
    - `=` → se próximo for `=` → `==` (`B17`); caso contrário (`=` isolado não é átomo válido em BigHole2026-2), descartar `=` conforme filtro tolerante e continuar busca do próximo átomo.
    - `!` → se próximo for `=` → `!=` (`B18`); caso contrário (`!` isolado não é átomo válido), descartar `!` e continuar busca do próximo átomo.
- **Critérios de Aceite:**
  - [ ] `:=` produz `B04` e não `:` + `=`.
  - [ ] `<=`, `>=`, `==`, `!=` produzem `B20`, `B22`, `B17`, `B18`.
  - [ ] `#` produz `B18` com lexema `"#"`.
  - [ ] `a<=10` sem espaços produz `A` (`C01`), `<=` (`B20`), `10` (`C06`).
  - [ ] `=` ou `!` isolados não travam o scanner nem geram loop infinito.

---

### Sub-Issue `ME04.3` — Constantes Numéricas (`C06` `intConst`, `C07` `realConst`) e Truncagem Segura
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Implementar o autômato de reconhecimento numérico acionado quando o primeiro caractere válido for `<digito>` (`0`–`9`):
  - **Inteiro (`C06`):** Consome `<digito>+`.
  - **Transição para Real (`C07`):**
    - Se após os dígitos houver `.` seguido de pelo menos um `<digito>`, consome `.` e `<digito>+`. (Se `.` **não** for seguido de dígito, faz *backtrack*/descarta `.` e emite o `intConst` lido até então).
    - **Parte Exponencial Opcional (`TC04`):** Após `<digitos>.<digitos>`, se houver `e`/`E` seguido de `[+|-]` e `<digito>+`, consome a parte exponencial completa (normalizada com `E` maiúsculo). Se após `E` (ou `E+`/`E-`) não houver dígitos, recua para antes do `E` e fecha o `realConst` apenas na parte decimal!
  - **Regra de Truncagem Coerente (limite de 30 caracteres):**
    - Se um `intConst` exceder 30 dígitos, mantém os 30 primeiros e consome o restante até o delimitador.
    - Se um `realConst` exceder 30 caracteres, garantir que o prefixo truncado (≤ 30) **seja um `realConst` válido** (ou recuar o ponto de corte caso o 30º caractere caia em `.`, `E`, `E+` ou `E-` — ex.: `123...789.` não pode terminar em `.`; `123.45...E+` não pode terminar em `E+`). Registrar `qtdCharsAntesTrunc` (total real válido lido) e `qtdCharDepoisTrunc` (tamanho final do lexema válido mantido).
- **Critérios de Aceite:**
  - [ ] `42` → `Token("42", "C06", idx, linha)` com `TipoSimb = IN`.
  - [ ] `3.14159E+10` (`TC04`), `3.14e-2`, `0.5e3` → `C07` em caixa alta (`3.14159E+10`, `3.14E-2`, `0.5E3`) com `TipoSimb = FP`.
  - [ ] Número real com 35 caracteres cuja posição 30 seria `.`, `E` ou `E+` é ajustado para terminar em um dígito válido sem corromper o átomo.
  - [ ] Caracteres excedentes após o 30º são consumidos até o delimitador e contabilizados em `qtdCharsAntesTrunc`.

---

### Sub-Issue `ME04.4` — Constantes Textuais (`C04` `stringConst`, `C05` `charConst`) e Truncagem
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Implementar o reconhecimento de literais de cadeia e caractere:
  - **`stringConst` (`C04`):**
    - Inicia com `"` e consome caracteres do `<miolo-cadeia>` (`<letra>`, `<digito>`, `<branco>`, `$`, `_`, `.`), convertendo letras para caixa alta.
    - Caracteres fora do conjunto permitido no miolo (exceto `"` de fechamento e quebra de linha/EOF) são filtrados pelo filtro de 1º nível (descartados sem fechar a string e sem contar no tamanho).
    - **Regra de 30 caracteres:** As aspas de abertura e fechamento **contam** no tamanho total (`máximo 28 chars de miolo + 2 aspas = 30`). Se o miolo ultrapassar 28 caracteres válidos, continuar consumindo até o `"` de fechamento (contabilizando em `qtdCharsAntesTrunc`), mas **forçar o fechamento `"` na posição 30** do lexema truncado (`qtdCharDepoisTrunc = 30`).
    - **String sem fechamento (quebra de linha `\n` ou EOF):** Fechar a string de forma tolerante na posição limite/quebra de linha sem travar o compilador (conforme seção 2.4 da `Etapa1_Proj`).
  - **`charConst` (`C05`):**
    - Inicia com `'`, aceita `<letra>` (normalizada para caixa alta) seguida de `'` (ex.: `'a'` → `'A'`, tamanho 3).
    - Tratar graciosamente entradas malformadas (ex.: `''` ou `'ab'` ou `'1'`) sem travar o scanner.
- **Critérios de Aceite:**
  - [ ] `"Ola Mundo 123_$."` → `Token("\"OLA MUNDO 123_$.\"", "C04", idx, linha)` com `TipoSimb = ST`.
  - [ ] String com 40 caracteres válidos (incluindo aspas) gera lexema de exatos 30 caracteres iniciando e terminando com `"`, `qtdCharsAntesTrunc = 40`, `qtdCharDepoisTrunc = 30`.
  - [ ] String não fechada antes de `\n` ou EOF encerra graciosamente sem consumir a próxima linha de código.
  - [ ] `'z'` → `Token("'Z'", "C05", idx, linha)` com `TipoSimb = CH`.

---

### Sub-Issue `ME04.5` — Integração do `LexicalAnalyzer` com `ISymbolTableManager` e Suíte de Testes
- **Labels:** `sub-issue`, `lexico`, `testes`
- **Descrição:**
  Finalizar a classe `LexicalAnalyzer implements ILexicalAnalyzer` integrando `SourceReader`, `FirstLevelFilter`, `ReservedTable` e `ISymbolTableManager`:
  - Garantir que `linha` registrada no `Token` e na `SymbolEntry` seja sempre a **linha inicial** onde o átomo começou.
  - Garantir que `proximoToken(contexto)` retorne `null` exclusivamente ao atingir EOF após pular espaços/comentários.
  - Escrever bateria robusta de testes unitários (`LexicalAnalyzerTest`) cobrindo todos os átomos `A01`–`A26`, `B01`–`B22`, `C01`–`C07`, `TC01`–`TC04`, filtro de 1º nível, truncagem e casos de borda.
- **Critérios de Aceite:**
  - [ ] Todos os átomos `C01`–`C07` são cadastrados via `ISymbolTableManager` e retornam seu `indiceTabSimb` no `Token`.
  - [ ] Nenhum átomo `A01`–`A26` ou `B01`–`B22` é inserido na Tabela de Símbolos (`indiceTabSimb == null`).
  - [ ] `LexicalAnalyzerTest` passando 100% com cobertura de todos os padrões do Apêndice C.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 5 Sub-Issues (`ME04.1` a `ME04.5`) concluídas, revisadas e integradas na `main`.
- [ ] `LexicalAnalyzer` reconhece integralmente programas válidos e malformados em `BigHole2026-2` sem lançar exceções não tratadas.
- [ ] Todos os testes unitários da `M2` (`ME02` + `ME03` + `ME04`) passando em conjunto.
