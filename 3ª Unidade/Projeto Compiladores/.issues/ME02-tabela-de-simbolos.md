# [ME02] Gerenciador da Tabela de Símbolos (`SymbolTableManager`)

> **Milestone:** `M2: Núcleo Léxico & Símbolos`  
> **Labels:** `macroentrega`, `tabela-simbolos`, `core`  
> **Dependências:** `ME01`  
> **Bloqueia:** `ME04` (integração final), `ME05`

---

## 1. Objetivo

Implementar de forma completa, isolada e testada o módulo **Gerenciador da Tabela de Símbolos** (`SymbolTableManager` e `SymbolEntry`), responsável por armazenar de maneira única todos os átomos da classe C (`C01`–`C07`), rastrear suas primeiras 5 linhas de ocorrência, registrar comprimentos pré e pós-truncagem e manter o tipo semântico (`TipoSimb`) que será espelhado no relatório `.TAB`.

---

## 2. Regras da Especificação Aplicáveis

Extraídas de [Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md) (seção *Tabela de Símbolos*) e [EntregaEtapa1PROJ.md](../Etapa1_Proj/EntregaEtapa1PROJ.md):

- **Apenas átomos identificadores/classe C** (`C01` a `C07`) entram na Tabela de Símbolos. Palavras reservadas (`A01`–`A26`) e símbolos reservados (`B01`–`B22`) **nunca** entram.
- **Unicidade:** Cada símbolo (lexema truncado em caixa alta) possui um único endereço (índice 1-based sequencial) que nunca muda durante a análise.
- **Imutabilidade vs. Mutabilidade dos Atributos:**
  - *Imutáveis após criação:* `entrada` (índice), `codigo` (`C01`–`C07`), `lexeme` (até 30 caracteres válidos em caixa alta).
  - *Atualizáveis durante a análise:* `tipoSimb` (quando resolvido pelo contexto declarativo), `linhas` (até o teto de 5 ocorrências), e tamanhos pré/pós-truncagem conforme política definida pela equipe.
- **Rastreamento de Linhas:** Armazenar as **5 primeiras ocorrências** do símbolo. Repetições na mesma linha **são registradas** conforme o exemplo oficial (`Linhas: (1, 2, 2, 2, 3)` para `VAR1` aparecendo 3 vezes na linha 2). Ocorrências a partir da 6ª são ignoradas silenciosamente.

---

## 3. Sub-Issues

- [ ] **ME02.1** — Implementar entidade `SymbolEntry` com todos os atributos e limite de 5 ocorrências
- [ ] **ME02.2** — Implementar `SymbolTableManager` com indexação $O(1)$ (`LinkedHashMap`) e política de inserção/atualização
- [ ] **ME02.3** — Implementar gestão de tipos semânticos (`TipoSimb`) e consulta por índice/lexema
- [ ] **ME02.4** — Criar suíte de testes unitários para o módulo de Tabela de Símbolos

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME02.1` — Implementar entidade `SymbolEntry`
- **Labels:** `sub-issue`, `tabela-simbolos`
- **Descrição:**
  Implementar a classe `SymbolEntry` no pacote `br.ucsal.bighole.symboltable` com:
  - `entrada` (`int`, 1-based)
  - `codigo` (`String`, `"C01"` a `"C07"`)
  - `lexeme` (`String`, caixa alta, máximo 30 caracteres)
  - `qtdCharsAntesTrunc` (`int`)
  - `qtdCharDepoisTrunc` (`int`, máximo 30)
  - `tipoSimb` (`TipoSimbolo`, padrão `INDEFINIDO` / `"-"`)
  - `linhas` (`List<Integer>`, capacidade máxima 5)
  - Método `adicionarOcorrencia(int linha)` que só insere se `linhas.size() < 5`.
  - Retornar cópia imutável (`Collections.unmodifiableList`) em `getLinhas()` para evitar mutação externa acidental.
- **Critérios de Aceite:**
  - [ ] Construtor inicializa a primeira linha automaticamente e define `tipoSimb = TipoSimbolo.INDEFINIDO`.
  - [ ] Ao chamar `adicionarOcorrencia` 7 vezes, apenas as 5 primeiras linhas permanecem registradas (inclusive quando houver linhas repetidas).
  - [ ] Validações defensivas: `entrada >= 1`, `qtdCharDepoisTrunc <= 30`, `qtdCharsAntesTrunc >= qtdCharDepoisTrunc`.

---

### Sub-Issue `ME02.2` — Implementar `SymbolTableManager` com indexação $O(1)$ e unicidade
- **Labels:** `sub-issue`, `tabela-simbolos`, `core`
- **Descrição:**
  Implementar `SymbolTableManager implements ISymbolTableManager` utilizando:
  - `LinkedHashMap<String, SymbolEntry>` indexado pela chave normalizada do símbolo (lexema truncado em caixa alta) para busca $O(1)$ preservando a ordem exata de inserção para o relatório `.TAB`.
  - Lista auxiliar ou mapa por índice (`List<SymbolEntry>` indexado por `indice - 1`) para permitir atualização de tipo por índice (`atualizarTipo(int indice, ...)`) também em tempo $O(1)$.
  - **Ponto de atenção (colisão entre classes diferentes):** Avaliar e documentar o comportamento caso uma constante (ex.: `"VAR1"`) e uma variável (`VAR1`) coexistam — como `stringConst` guarda as aspas (`"VAR1"`), `charConst` guarda apóstrofos (`'A'`) e números começam com dígito, não há colisão entre `C01`/`C02`/`C03` e `C04`–`C07`! Ainda assim, manter a normalização consistente.
- **Critérios de Aceite:**
  - [ ] Primeira inserção atribui índice `1`, segunda `2`, e assim sucessivamente.
  - [ ] Re-inserção do mesmo lexema retorna o **mesmo índice** existente e registra apenas a nova linha de ocorrência (se `< 5`).
  - [ ] `getEntradas()` devolve lista imutável estritamente na ordem de criação (entrada `1`, `2`, ..., `N`).

---

### Sub-Issue `ME02.3` — Implementar gestão de tipos semânticos (`TipoSimb`)
- **Labels:** `sub-issue`, `tabela-simbolos`
- **Descrição:**
  Implementar a lógica de associação e atualização de `TipoSimbolo` nas entradas da Tabela de Símbolos:
  - Suportar `atualizarTipo(int indice, TipoSimbolo tipoSimb)` (e sobrecarga por `String` se prevista no contrato).
  - Respeitar a regra da especificação: uma vez definido um tipo válido para o símbolo, impedir sobrescrita indevida por `INDEFINIDO` (`"-"`).
  - Para constantes (`C04` → `ST`, `C05` → `CH`, `C06` → `IN`, `C07` → `FP`), permitir preenchimento imediato do tipo no momento do registro ou via helper específico.
- **Critérios de Aceite:**
  - [ ] `atualizarTipo` funciona em $O(1)$ pelo índice 1-based e lança exceção clara (`IllegalArgumentException`) para índice inexistente.
  - [ ] Todos os 11 códigos (`FP`, `IN`, `ST`, `CH`, `BL`, `VD`, `AF`, `AI`, `AS`, `AC`, `AB`) e `"-"` são armazenados e recuperados corretamente.
  - [ ] Não permite sobrescrever um tipo já resolvido por `"-"`.

---

### Sub-Issue `ME02.4` — Criar suíte de testes unitários para `SymbolTableManager`
- **Labels:** `sub-issue`, `tabela-simbolos`, `testes`
- **Descrição:**
  Implementar `SymbolTableManagerTest` cobrindo todos os requisitos isolados do módulo.
- **Cenários Obrigatórios:**
  - [ ] Inserção de novos símbolos gera índices sequenciais `1, 2, 3...`.
  - [ ] Símbolo repetido mantém índice único e acumula linhas (incluindo múltiplas ocorrências na mesma linha, reproduzindo `Teste.262`: `(1, 2, 2, 2, 3)`).
  - [ ] **TC06:** Símbolo repetido em 7 linhas distintas armazena exatamente as 5 primeiras.
  - [ ] Preservação de `qtdCharsAntesTrunc` (ex.: `48`) e `qtdCharDepoisTrunc` (`30`).
  - [ ] Atualização de tipos escalares e de vetores (`AI`, `AF`, `AS`, `AC`, `AB`) sem alterar ordem das entradas.
  - [ ] Tabela vazia retorna lista vazia sem `NullPointerException`.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 4 Sub-Issues (`ME02.1` a `ME02.4`) concluídas, revisadas e mescladas na `main`.
- [ ] `SymbolTableManager` implementa integralmente `ISymbolTableManager`.
- [ ] Cobertura de testes unitários validando todas as políticas da seção *Tabela de Símbolos* da especificação.
