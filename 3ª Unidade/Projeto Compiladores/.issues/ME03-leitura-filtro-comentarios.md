# [ME03] Leitura de Entrada, Filtro de 1º Nível e Tratamento de Comentários

> **Milestone:** `M2: Núcleo Léxico & Símbolos`  
> **Labels:** `macroentrega`, `lexico`, `core`, `prioridade:alta`  
> **Dependências:** `ME01`  
> **Bloqueia:** `ME04`

---

## 1. Objetivo

Construir a camada de pré-processamento e varredura caractere a caractere do Analisador Léxico, garantindo leitura confiável de arquivos ASCII com *lookahead*, contagem exata de linhas, aplicação rigorosa do **Filtro de 1º Nível** (caracteres inválidos silenciosamente descartados sem quebrar lexemas), tratamento de espaços/delimitadores e descarte tolerante de comentários (`//` e `/* ... */`).

Separar esta camada em uma macroentrega própria simplifica drasticamente a máquina de estados da `ME04` e elimina a maior fonte de bugs clássicos em compiladores (erros de *pushback*, contagem errada de linha em `\r\n` e caracteres inválidos quebrando átomos).

---

## 2. Regras Críticas da Especificação

Extraídas de [Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md) (seção *Analisador léxico*):

1. **Alfabeto Válido de BigHole2026-2:**
   - Letras ASCII: `a`–`z`, `A`–`Z`
   - Dígitos ASCII: `0`–`9`
   - Símbolos usados em átomos: `;`, `,`, `:`, `=`, `?`, `(`, `)`, `[`, `]`, `{`, `}`, `+`, `-`, `*`, `/`, `%`, `!`, `#`, `<`, `>`, `_`, `.`, `$`, `"`, `'`
   - Espaços e controles de linha: espaço (`ASCII 32`), tabulação (`ASCII 09`), LF (`ASCII 10` / `\n`), CR (`ASCII 13` / `\r`).
2. **Filtro de 1º Nível:**
   - Qualquer caractere fora do conjunto válido acima (ex.: `@`, `~`, `^`, `&`, `|`, `\`, caracteres acentuados fora de comentário) **não gera erro** e **não atua como delimitador**.
   - Exemplo oficial (`TC01`): `va@r#x` → o `@` é ignorado sem quebrar o átomo, produzindo `VAR` (`C01`), `#` (`B18`), `X` (`C01`).
   - Caracteres filtrados **não entram** na contagem de tamanho (`QtdCharsAntesTrunc` / `QtdCharDepoisTrunc`).
3. **Comentários:**
   - **Bloco (`/* ... */`):** Atua como delimitador entre átomos. Dentro do comentário, qualquer caractere (mesmo inválido) é ignorado, mas quebras de linha **continuam incrementando o contador de linhas**. Se chegar ao EOF sem `*/`, encerra normalmente sem erro fatal (`TC03`).
   - **Linha (`// ...`):** Atua como delimitador e descarta tudo até `\n` (ou EOF).
   - Comentários **não** sofrem limite de 30 caracteres.

---

## 3. Sub-Issues

- [ ] **ME03.1** — Implementar `SourceReader` (leitura ASCII caractere a caractere, *lookahead* e contagem de linhas)
- [ ] **ME03.2** — Implementar `FirstLevelFilter` (classificação do alfabeto válido e descarte transparente de inválidos)
- [ ] **ME03.3** — Implementar tratamento de espaços em branco, tabulações e quebras de linha
- [ ] **ME03.4** — Implementar reconhecedor e descarte de comentários de linha (`//`) e de bloco (`/* ... */`)

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME03.1` — Implementar `SourceReader` com *lookahead* e controle de linhas
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Criar um leitor encapsulado sobre `BufferedReader` / `PushbackReader` (ou buffer próprio em memória / buffer de duas metades) no pacote `br.ucsal.bighole.lexical`:
  - Operações: `int read()`, `int peek()`, `void unread(int ch)`, `boolean isEof()`, `int getLinhaAtual()`.
  - Normalizar quebras de linha Windows (`\r\n`), Unix (`\n`) e Mac antigo (`\r`) para que `\r\n` conte como **apenas 1** incremento de linha (e não 2!).
  - Garantir que `unread('\n')` decremente a linha corretamente caso um `\n` seja devolvido ao buffer (fundamental para não errar a linha do próximo token).
  - Implementar `AutoCloseable` para fechar o arquivo corretamente.
- **Critérios de Aceite:**
  - [ ] Linha inicia em `1`.
  - [ ] `\n`, `\r\n` e `\r` incrementam a linha exatamente em `+1`.
  - [ ] `peek()` não consome o caractere nem altera o contador de linhas.
  - [ ] `unread()` devolve o caractere e preserva a consistência de `getLinhaAtual()`.
  - [ ] Testes unitários com strings contendo `\n`, `\r\n`, arquivo vazio e múltiplos `peek`/`unread`.

---

### Sub-Issue `ME03.2` — Implementar `FirstLevelFilter` (Filtro de 1º Nível)
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Implementar componente dedicado à classificação de caracteres segundo o alfabeto de BigHole2026-2:
  - Métodos auxiliares semânticos: `isCaractereValido(int ch)`, `isLetra(int ch)`, `isDigito(int ch)`, `isEspacoOuQuebra(int ch)`, `isValidoEmString(int ch)` (`letra | digito | ' ' | '$' | '_' | '.'`).
  - Método `lerProximoCaractereValido()` que avança sobre o `SourceReader` descartando silenciosamente qualquer caractere inválido (`!isCaractereValido(ch)`), parando no próximo caractere válido ou EOF.
  - **Atenção:** Dentro de comentários (`//` e `/* */`), o filtro não precisa validar o alfabeto (tudo até o fechamento é ignorado); dentro de `stringConst`, caracteres inválidos para o miolo da string devem ser filtrados conforme a regra de 1º nível (sem quebrar a string).
- **Critérios de Aceite:**
  - [ ] `va@r1` lido via filtro produz sequência contínua `v`, `a`, `r`, `1`.
  - [ ] `12@3.4§5` produz `1`, `2`, `3`, `.`, `4`, `5`.
  - [ ] Caracteres acentuados (`á`, `ç`, `ã`), `@`, `~`, `^`, `&`, `|`, `\` são descartados sem lançar exceção.
  - [ ] Testes unitários validando fronteiras do alfabeto válido e inválido.

---

### Sub-Issue `ME03.3` — Implementar consumo de espaços em branco e delimitadores de arrumação
- **Labels:** `sub-issue`, `lexico`
- **Descrição:**
  Implementar rotina para pular sequências contíguas de espaços (`ASCII 32`), tabulações (`ASCII 09`) e quebras de linha (`ASCII 10` e `13`) entre átomos:
  - Colapsar espaços repetidos sem gerar tokens.
  - Manter o espaço (`ASCII 32`) preservável quando o scanner estiver explicitamente no estado de leitura de `stringConst` (pois `<branco>` faz parte de `<miolo-cadeia>`).
- **Critérios de Aceite:**
  - [ ] Múltiplos espaços, tabs e linhas em branco entre tokens são descartados e o contador de linhas reflete exatamente a linha inicial do próximo átomo.
  - [ ] Arquivo contendo apenas espaços/quebras retorna EOF imediatamente.

---

### Sub-Issue `ME03.4` — Implementar descarte de comentários (`//` e `/* ... */`)
- **Labels:** `sub-issue`, `lexico`, `prioridade:alta`
- **Descrição:**
  Implementar o consumo de comentários integrado ao ciclo de avanço de espaços/delimitadores do scanner:
  - Ao encontrar `/` seguido de `/` → consumir comentário de linha até `\n` (sem consumir o caractere da linha seguinte) ou até EOF.
  - Ao encontrar `/` seguido de `*` → consumir comentário de bloco até encontrar a sequência `*/` ou até EOF (tolerante a comentário não fechado — `TC03`).
  - Ao encontrar `/` seguido de qualquer outro caractere válido → **não** é comentário; devolver o lookahead e deixar o scanner reconhecer o operador de divisão `/` (`B15`).
  - **Cuidado especial:** E se houver comentários consecutivos intercalados por espaços (ex.: `/* c1 */   // c2 \n /* c3 */ var1`)? O loop de descarte de espaços e comentários deve iterar até que o próximo caractere válido não seja espaço nem início de comentário!
  - **Cuidado especial 2:** `/***/` (múltiplos asteriscos antes de `/`) deve fechar corretamente no `*/` final.
- **Critérios de Aceite:**
  - [ ] Comentários de bloco multilinha incrementam corretamente `getLinhaAtual()` para o token subsequente.
  - [ ] Comentário de bloco não fechado até EOF termina graciosamente sem loop infinito nem exceção (`TC03`).
  - [ ] Comentário de linha na última linha do arquivo (sem `\n` final) termina graciosamente em EOF.
  - [ ] Comentários atuam como delimitadores (ex.: `var1/*coment*/var2` produz dois tokens: `VAR1` e `VAR2`).
  - [ ] `a / b` continua sendo reconhecido como operador `/` (`B15`).
  - [ ] `/**/` e `/***/` são fechados corretamente.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 4 Sub-Issues (`ME03.1` a `ME03.4`) concluídas e integradas na `main`.
- [ ] Testes unitários cobrindo leitura com lookahead, `\r\n`, filtro de 1º nível (`TC01`), comentários de linha, comentários de bloco, comentários encadeados e comentário aberto até EOF (`TC03`).
