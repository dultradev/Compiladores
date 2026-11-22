# [ME06] Geradores de Relatórios de Saída (`.LEX` e `.TAB`)

> **Milestone:** `M3: Orquestração, Escopo & Relatórios` *(pode ser iniciada em paralelo já na `M2` usando mocks!)*  
> **Labels:** `macroentrega`, `relatorios`, `io`  
> **Dependências:** `ME01`  
> **Bloqueia:** `ME07`

---

## 1. Objetivo

Implementar o módulo responsável pela formatação e persistência em disco dos dois relatórios obrigatórios produzidos pelo Static Checker — `<nome>.LEX` e `<nome>.TAB` — seguindo rigorosamente a estrutura, rótulos, pontuação e ordem definidos na especificação oficial ([Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md)).

---

## 2. Layouts Oficiais Exigidos

### Cabeçalho Comum (obrigatório em ambos os arquivos)
```text
Código da Equipe: EQ04
Componentes:
Leonardo Santos Magalhães; leonardo.magalhaes@aln.senaicimatec.edu.br; (71) 981379640
Marcos Paulo Teles de Menezes Filho; marcos.p.filho@aln.senaicimatec.edu.br; (71) 992782161
Rafael Guerra de Oliveira; rafael.g.oliveira@aln.senaicimatec.edu.br; (75) 991140115
Rian Albert dos Santos Dultra; rian.santos@aln.senaicimatec.edu.br; (71) 987981802
```
*(Os dados devem vir exclusivamente de `EquipeConfig` para evitar divergências entre `.LEX` e `.TAB`.)*

### Layout do `<nome>.LEX`
```text
RELATÓRIO DA ANÁLISE LÉXICA.
Texto fonte analisado: Teste.262
Lexeme: VAR1, Código: C01, indiceTabSimb: 1, Linha: 1.
Lexeme: ;, Código: B01, indiceTabSimb: -, Linha: 4.
```

### Layout do `<nome>.TAB`
```text
RELATÓRIO DA TABELA DE SÍMBOLOS.
Texto fonte analisado: Teste.262
Entrada: 1, Código: C01, Lexeme: VAR1,
QtdCharsAntesTrunc: 4, QtdCharDepoisTrunc: 4,
TipoSimb: -, Linhas: (1, 2, 2, 2, 3).
---------------------------------------------------------------
Entrada: 2, Código: C01, Lexeme: VAR2,
QtdCharsAntesTrunc: 4, QtdCharDepoisTrunc: 4,
TipoSimb: -, Linhas: (1, 3, 3).
---------------------------------------------------------------
Entrada: 3, Código: C01, Lexeme: VAR3,
QtdCharsAntesTrunc: 4, QtdCharDepoisTrunc: 4,
TipoSimb: -, Linhas: (1).
```

---

## 3. Sub-Issues

- [ ] **ME06.1** — Implementar formatador do cabeçalho obrigatório comum a partir de `EquipeConfig`
- [ ] **ME06.2** — Implementar gerador e escritor do relatório `<nome>.LEX`
- [ ] **ME06.3** — Implementar gerador e escritor do relatório `<nome>.TAB` com testes de snapshot

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME06.1` — Formatador do Cabeçalho Obrigatório Comum
- **Labels:** `sub-issue`, `relatorios`
- **Descrição:**
  Criar utilitário `ReportHeaderFormatter` no pacote `br.ucsal.bighole.report` que monta o bloco de cabeçalho padrão a partir de `EquipeConfig`, recebendo o título do relatório (`RELATÓRIO DA ANÁLISE LÉXICA.` ou `RELATÓRIO DA TABELA DE SÍMBOLOS.`) e o nome do arquivo analisado (ex.: `Teste.262`, apenas o `fileName`, sem caminho absoluto poluindo o cabeçalho).
- **Critérios de Aceite:**
  - [ ] Exibe código da equipe, lista completa dos 4 componentes (nome; e-mail; telefone), título em maiúsculas e `Texto fonte analisado: <arquivo>.262`.
  - [ ] Usa `System.lineSeparator()` ou `\n` consistente e encoding UTF-8 (para suportar acentuação nos nomes dos componentes e títulos `CÓDIGO`, `RELATÓRIO`, `ANÁLISE LÉXICA`, `SÍMBOLOS`).

---

### Sub-Issue `ME06.2` — Gerador do Relatório `<nome>.LEX`
- **Labels:** `sub-issue`, `relatorios`, `io`
- **Descrição:**
  Implementar a geração do arquivo `<nome>.LEX` no mesmo diretório do arquivo fonte `.262`:
  - Para cada `Token` na ordem exata de aparição no texto fonte, gravar:
    `Lexeme: <lexeme>, Código: <codigo>, indiceTabSimb: <indice ou ->, Linha: <linha>.`
  - Garantir escrita bufferizada (`Files.newBufferedWriter` com UTF-8), sobrescrevendo eventual `.LEX` anterior da mesma pasta.
  - Garantir comportamento limpo para arquivos fonte vazios (ou contendo apenas comentários): gera cabeçalho válido sem linhas detalhe e sem falhar.
- **Critérios de Aceite:**
  - [ ] Saída para os tokens de `Teste.262` coincide exatamente com o formato da especificação oficial.
  - [ ] Tokens `A01`–`A26` e `B01`–`B22` imprimem `indiceTabSimb: -`.
  - [ ] Tokens `C01`–`C07` imprimem o número inteiro 1-based de `indiceTabSimb`.
  - [ ] Arquivo salvo exatamente na mesma pasta do arquivo `.262` original.

---

### Sub-Issue `ME06.3` — Gerador do Relatório `<nome>.TAB` e Testes de Snapshot
- **Labels:** `sub-issue`, `relatorios`, `io`, `testes`
- **Descrição:**
  Implementar a geração do arquivo `<nome>.TAB` na mesma pasta do arquivo fonte `.262` e criar testes unitários de comparação exata (*snapshot tests*) para ambos os relatórios:
  - Para cada `SymbolEntry` retornada por `symbolTable.getEntradas()` (em ordem crescente de `entrada`):
    ```text
    Entrada: <entrada>, Código: <codigo>, Lexeme: <lexeme>,
    QtdCharsAntesTrunc: <antes>, QtdCharDepoisTrunc: <depois>,
    TipoSimb: <tipo>, Linhas: (<l1>, <l2>, ...).
    ```
  - Inserir linha separadora `---------------------------------------------------------------` entre as entradas (sem erro de formatação na última entrada).
  - Formatar a lista de linhas entre parênteses separadas por `", "` (ex.: `(1)` ou `(1, 2, 2, 2, 3)`).
- **Critérios de Aceite:**
  - [ ] Saída para os dados mockados de `Teste.262` coincide linha a linha com o exemplo de `Projeto2026-2 vSN1.md`.
  - [ ] Tabela vazia gera cabeçalho válido sem lançar exceção.
  - [ ] Testes unitários validam `.LEX` e `.TAB` em diretório temporário (`@TempDir` do JUnit 5), garantindo que os arquivos são criados na mesma pasta do `.262` de origem.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 3 Sub-Issues (`ME06.1` a `ME06.3`) concluídas, revisadas e integradas na `main`.
- [ ] `ReportGenerator` implementa `IReportGenerator` e passa em 100% dos testes de snapshot.
- [ ] Acentuação, pontuação, nomes dos campos (`indiceTabSimb`, `QtdCharsAntesTrunc`, `QtdCharDepoisTrunc`, `TipoSimb`, `Linhas`) conferidos contra a especificação oficial.
