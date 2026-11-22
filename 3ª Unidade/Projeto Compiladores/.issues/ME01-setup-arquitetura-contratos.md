# [ME01] Setup do Projeto, Arquitetura Base e Contratos entre Módulos

> **Milestone:** `M1: Fundação & Arquitetura`  
> **Labels:** `macroentrega`, `arquitetura`, `setup`, `prioridade:alta`  
> **Dependências:** Nenhuma (marco inicial)  
> **Bloqueia:** `ME02`, `ME03`, `ME04`, `ME05`, `ME06`

---

## 1. Objetivo

Estabelecer toda a base estrutural do repositório e do projeto Java (JDK 27) conforme a arquitetura modular definida na [Etapa 1 — PROJ](../Etapa1_Proj/EntregaEtapa1PROJ.md). Ao concluir esta macroentrega, todas as interfaces, modelos de dados imutáveis e a tabela de palavras/símbolos reservados estarão congelados e compilando, permitindo que os 4 integrantes trabalhem em paralelo nas frentes da Milestone 2 e Milestone 3 sem conflitos de integração.

---

## 2. Escopo Arquitetural

Estrutura de pacotes definida para o projeto:

```text
src/
├── main/java/br/ucsal/bighole/
│   ├── StaticChecker.java                 # Ponto de entrada e controlador sintático inicial (ME05)
│   ├── config/
│   │   └── EquipeConfig.java              # Metadados da equipe para cabeçalhos .LEX e .TAB
│   ├── model/
│   │   ├── Token.java                     # Record imutável de átomo reconhecido
│   │   ├── ContextoEscopo.java            # Enum de contexto sintático (C01 vs C02 vs C03)
│   │   └── TipoSimbolo.java               # Enum dos 11 tipos semânticos + indefinido ("-")
│   ├── lexical/
│   │   ├── ILexicalAnalyzer.java          # Contrato do Analisador Léxico
│   │   └── ReservedTable.java             # Mapa O(1) de palavras (A01-A26) e símbolos (B01-B22)
│   ├── symboltable/
│   │   ├── ISymbolTableManager.java       # Contrato da Tabela de Símbolos
│   │   └── SymbolEntry.java               # Estrutura de entrada (implementada em ME02)
│   └── report/
│       └── IReportGenerator.java          # Contrato de emissão dos relatórios .LEX e .TAB
└── test/java/br/ucsal/bighole/            # Suíte de testes automatizados (JUnit 5)
```

---

## 3. Sub-Issues

- [ ] **ME01.1** — Configurar projeto Java 27 no IntelliJ IDEA, estrutura de diretórios e build com testes
- [ ] **ME01.2** — Implementar modelos de dados (`Token`, `ContextoEscopo`, `TipoSimbolo` e `EquipeConfig`)
- [ ] **ME01.3** — Definir interfaces e contratos entre módulos (`ILexicalAnalyzer`, `ISymbolTableManager`, `IReportGenerator`)
- [ ] **ME01.4** — Implementar `ReservedTable` com palavras reservadas (`A01`–`A26`) e símbolos reservados (`B01`–`B22`)

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME01.1` — Configurar projeto Java 27 no IntelliJ IDEA, estrutura de diretórios e build
- **Labels:** `sub-issue`, `setup`, `prioridade:alta`
- **Descrição:**
  Inicializar o projeto Java (JDK 27) compatível com IntelliJ IDEA 2026.2.3, configurando a estrutura padrão `src/main/java` e `src/test/java`, ferramenta de build (Maven ou Gradle — definir em consenso da equipe) com suporte a JUnit 5 e geração de JAR executável, além de `.gitignore` adequado para projetos Java/IntelliJ.
- **Critérios de Aceite:**
  - [ ] Projeto configurado com target JDK 27 e encoding UTF-8 (com leitura explícita de fontes ASCII).
  - [ ] Estrutura de pacotes `br.ucsal.bighole.{config,model,lexical,symboltable,report}` criada.
  - [ ] `.gitignore` ignorando arquivos `.class`, `target/` ou `build/`, `.idea/` (ou mantendo apenas configs compartilháveis), `.LEX` e `.TAB` temporários.
  - [ ] Comando de build e execução de testes rodando com sucesso via terminal e via IntelliJ IDEA.

---

### Sub-Issue `ME01.2` — Implementar modelos de dados (`Token`, `ContextoEscopo`, `TipoSimbolo` e `EquipeConfig`)
- **Labels:** `sub-issue`, `arquitetura`, `prioridade:alta`
- **Descrição:**
  Criar os tipos de domínio compartilhados entre todos os módulos do Static Checker:
  1. `Token` (Java `record`): `String lexeme`, `String codigo`, `Integer indiceTabSimb`, `int linha`, com método auxiliar `getIndiceFormatado()` retornando `"-"` quando `indiceTabSimb == null`.
  2. `ContextoEscopo` (`enum`): `GLOBAL_PROGRAM` (esperando `programName` — `C03`), `DECLARACAO_FUNCAO` (esperando `functionName` — `C02`), `CORPO_INSTRUCOES` (`variable` — `C01`).
  3. `TipoSimbolo` (`enum`): contemplando todas as siglas de 2 caracteres da especificação oficial:
     - Escalares: `FP` (Floating Point), `IN` (Integer), `ST` (String), `CH` (Character), `BL` (Boolean), `VD` (Void).
     - Vetores: `AF` (Array of Floating Point), `AI` (Array of Integer), `AS` (Array of String), `AC` (Array of Character), `AB` (Array of Boolean).
     - Não definido: `INDEFINIDO` (`"-"`).
     - Incluir método utilitário de conversão a partir da palavra reservada de tipo (`REAL`, `INTEGER`, `STRING`, `CHARACTER`, `BOOLEAN`, `VOID`) + flag `isArray`.
  4. `EquipeConfig`: constantes centralizadas com o código oficial da equipe e a lista dos 4 componentes (nome completo, e-mail institucional e telefone) para uso unificado nos cabeçalhos `.LEX` e `.TAB`.
- **Critérios de Aceite:**
  - [ ] `Token` validado como `record` imutável conforme contrato da `Etapa1_Proj`.
  - [ ] `TipoSimbolo` cobre todos os 11 códigos oficiais + `"-"` e possui teste unitário de mapeamento escalar/vetor.
  - [ ] `EquipeConfig` validado pela equipe quanto ao código oficial (`EQ04` vs `E04`) e dados de contato corretos.

---

### Sub-Issue `ME01.3` — Definir interfaces e contratos entre módulos
- **Labels:** `sub-issue`, `arquitetura`, `prioridade:alta`
- **Descrição:**
  Codificar as interfaces públicas com Javadoc completo que desacoplam os 3 módulos principais e o módulo de relatórios:
  - `ILexicalAnalyzer`: `Token proximoToken(ContextoEscopo contexto)` e `int getLinhaAtual()` (e `AutoCloseable` para garantir fechamento seguro de recursos de I/O).
  - `ISymbolTableManager`: `int registrarSimbolo(String lexemeTruncado, String codigo, int tamAntesTrunc, int tamDepoisTrunc, int linha)`, `void atualizarTipo(int indice, TipoSimbolo tipoSimb)` (ou `String`), `List<SymbolEntry> getEntradas()`.
  - `IReportGenerator`: métodos para gerar os arquivos `.LEX` e `.TAB` a partir do caminho do fonte, lista de tokens e tabela de símbolos.
- **Critérios de Aceite:**
  - [ ] Todas as interfaces compilam sem dependências cíclicas entre pacotes.
  - [ ] Javadoc descreve pré-condições, pós-condições e comportamento em EOF/nulo.
  - [ ] Assinaturas revisadas e aprovadas por todos os 4 integrantes antes de iniciar a `M2`.

---

### Sub-Issue `ME01.4` — Implementar `ReservedTable` (`A01`–`A26` e `B01`–`B22`)
- **Labels:** `sub-issue`, `lexico`, `arquitetura`, `prioridade:alta`
- **Descrição:**
  Implementar a tabela fixa e imutável de palavras reservadas e símbolos reservados da linguagem `BigHole2026-2`, carregada em memória antes da análise com consulta em $O(1)$ (`Map.ofEntries(...)`).
  - **26 Palavras Reservadas (`A01`–`A26`)** em caixa alta:
    `BOOLEAN` (`A01`), `BREAK` (`A02`), `CHARACTER` (`A03`), `DECLARATIONS` (`A04`), `ELSE` (`A05`), `ENDDECLARATIONS` (`A06`), `ENDFUNCTION` (`A07`), `ENDFUNCTIONS` (`A08`), `ENDIF` (`A09`), `ENDPROGRAM` (`A10`), `ENDWHILE` (`A11`), `FALSE` (`A12`), `FUNCTIONS` (`A13`), `FUNCTYPE` (`A14`), `IF` (`A15`), `INTEGER` (`A16`), `PARAMTYPE` (`A17`), `PRINT` (`A18`), `PROGRAM` (`A19`), `REAL` (`A20`), `RETURN` (`A21`), `STRING` (`A22`), `TRUE` (`A23`), `VARTYPE` (`A24`), `VOID` (`A25`), `WHILE` (`A26`).
  - **23 Símbolos/Operadores Reservados (`B01`–`B22`)**:
    `;` (`B01`), `,` (`B02`), `:` (`B03`), `:=` (`B04`), `?` (`B05`), `(` (`B06`), `)` (`B07`), `[` (`B08`), `]` (`B09`), `{` (`B10`), `}` (`B11`), `+` (`B12`), `-` (`B13`), `*` (`B14`), `/` (`B15`), `%` (`B16`), `==` (`B17`), `!=` (`B18`), `#` (`B18`), `<` (`B19`), `<=` (`B20`), `>` (`B21`), `>=` (`B22`).
  - Disponibilizar métodos claros: `Optional<String> buscarPalavraReservada(String lexemeUpper)`, `Optional<String> buscarSimboloReservado(String simbolo)`, `boolean isPalavraReservada(String lexemeUpper)`.
- **Critérios de Aceite:**
  - [ ] Todas as 26 palavras `A01`–`A26` mapeadas exatamente conforme o Apêndice A.1 (atenção à grafia `ENDDECLARATIONS` com `T`, não `endDeclaracions`).
  - [ ] Todos os 23 símbolos `B01`–`B22` mapeados exatamente conforme o Apêndice A.2, garantindo que `!=` e `#` retornem `B18`.
  - [ ] Mapa imutável (lança `UnsupportedOperationException` se tentarem alterar).
  - [ ] Teste unitário validando 100% dos 49 mapeamentos e retornos vazios para lexemas não reservados.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 4 Sub-Issues (`ME01.1` a `ME01.4`) concluídas e integradas na `main`.
- [ ] Build limpo (`compile` + `test`) executando com 0 erros e 0 warnings críticos.
- [ ] Equipe alinhada quanto aos contratos e pronta para paralelizar `ME02`, `ME03` e `ME06`.
