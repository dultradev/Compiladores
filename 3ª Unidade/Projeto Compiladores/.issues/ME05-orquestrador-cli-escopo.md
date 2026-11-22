# [ME05] Orquestrador Principal (`StaticChecker`), CLI e Controle de Escopo Contextual

> **Milestone:** `M3: Orquestração, Escopo & Relatórios`  
> **Labels:** `macroentrega`, `sintatico-escopo`, `cli`  
> **Dependências:** `ME02`, `ME04`  
> **Bloqueia:** `ME07`

---

## 1. Objetivo

Implementar o **Módulo Principal / Analisador Sintático Inicial** (`StaticChecker`), responsável por atuar como controlador *syntax-driven* do compilador: receber e resolver o arquivo fonte `.262` (via CLI ou prompt), inicializar todas as estruturas de dados, conduzir o loop de solicitação de tokens ao `LexicalAnalyzer`, manter a máquina de estados de **Controle de Escopo Contextual** (`C03` vs `C02` vs `C01`) e preencher os **Tipos Semânticos** (`TipoSimb`) das declarações na `SymbolTableManager`.

---

## 2. Regras da Especificação Aplicáveis

Extraídas de [Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md) (seções *Entradas*, *Analisador sintático* e *Tabela de Símbolos*) e [EntregaEtapa1PROJ.md](../Etapa1_Proj/EntregaEtapa1PROJ.md):

1. **Entrada e Resolução do Arquivo Fonte:**
   - O usuário passa **apenas o nome base sem extensão** (ex.: `MeuTeste` ou `/caminho/para/MeuTeste`).
   - O `StaticChecker` anexa `.262` e procura o arquivo no diretório corrente (se nome simples) ou no caminho completo fornecido.
   - *(Conveniência resiliente)*: se o usuário não informar argumento na linha de comando, solicitar interativamente o nome/caminho do arquivo no console; se o usuário digitar `.262` por engano, normalizar evitando `.262.262`.
   - Se o arquivo não existir ou não puder ser lido, exibir mensagem clara de erro no `System.err` e encerrar com código de saída `1`.
2. **Controle de Escopo Contextual (Desambiguação `C01` / `C02` / `C03`):**
   - `program <programName>` → após ler `A19` (`PROGRAM`), próximo identificador é `C03` (`GLOBAL_PROGRAM`).
   - `funcType <TypeSpecification> : <functionName>` → após `A14` (`FUNCTYPE`) + tipo (`A01`/`A03`/`A16`/`A20`/`A22`/`A25`) + `B03` (`:`), próximo identificador é `C02` (`DECLARACAO_FUNCAO`).
   - Demais posições → `C01` (`CORPO_INSTRUCOES`).
3. **Captura Contextual de Tipos Semânticos (`TipoSimb`):**
   - Declaração escalar: `varType <tipo> : var1 , var2 ...` → atribui `FP`, `IN`, `ST`, `CH`, `BL` ou `VD` a cada variável listada até `;` ou fim da seção.
   - Declaração de vetor: `varType <tipo> [ ] : vet1 [ 10 ] , vet2 [ 5 ]` → atribui `AF`, `AI`, `AS`, `AC` ou `AB`.
   - Declaração de função: `funcType <tipo> : nomeFunc` → atribui o tipo de retorno (`FP`, `IN`, `ST`, `CH`, `BL`, `VD`) à entrada `C02`.
   - Declaração de parâmetros: `paramType <tipo> : p1 , p2 [ 10 ]` → atribui tipo escalar para `p1` e tipo array (`AF`/`AI`/`AS`/`AC`/`AB`) quando seguido de `[ intConst ]`.
   - Constantes (`C04`–`C07`): recebem automaticamente `ST`, `CH`, `IN`, `FP`.

---

## 3. Sub-Issues

- [ ] **ME05.1** — Implementar entrada CLI/interativa e resolução de caminho do arquivo `.262`
- [ ] **ME05.2** — Implementar máquina de estados de Controle de Escopo Contextual (`C01` / `C02` / `C03`)
- [ ] **ME05.3** — Implementar captura contextual de Tipos Semânticos (`TipoSimb`) em declarações de variáveis, vetores, funções e parâmetros
- [ ] **ME05.4** — Implementar loop principal *syntax-driven* e orquestração completa com `IReportGenerator`

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME05.1` — Entrada CLI/Interativa e Resolução do Arquivo `.262`
- **Labels:** `sub-issue`, `cli`, `prioridade:alta`
- **Descrição:**
  Implementar um resolvedor de arquivos (`SourceFileResolver`) e o método `StaticChecker.main(String[] args)`:
  - Caso `args.length >= 1`, usar `args[0]`; caso contrário, ler o nome do arquivo via `stdin` com prompt claro ao usuário.
  - Resolver o caminho utilizando `java.nio.file.Path`:
    - Se o nome não terminar em `.262` (case-insensitive), acrescentar `.262`.
    - Se for relativo, resolver contra o diretório de trabalho corrente (`Path.of("").toAbsolutePath()`).
    - Extrair `diretorioSaida` (mesma pasta do arquivo fonte) e `nomeBase` (sem extensão) para nomear `<nomeBase>.LEX` e `<nomeBase>.TAB`.
  - Validar existência e permissão de leitura (`Files.isRegularFile` e `Files.isReadable`); em caso de erro, imprimir mensagem descritiva e encerrar com `System.exit(1)` (mantendo a lógica testável fora do `System.exit`).
- **Critérios de Aceite:**
  - [ ] `java -jar static-checker.jar MeuTeste` localiza `./MeuTeste.262` e define saídas `./MeuTeste.LEX` e `./MeuTeste.TAB`.
  - [ ] `java -jar static-checker.jar /tmp/dir/MeuTeste` localiza `/tmp/dir/MeuTeste.262` e define saídas `/tmp/dir/MeuTeste.LEX` e `/tmp/dir/MeuTeste.TAB`.
  - [ ] Arquivo inexistente produz mensagem amigável no console e código de saída `1`.
  - [ ] Testes unitários validando caminhos relativos, absolutos, com espaço no nome e arquivos ausentes.

---

### Sub-Issue `ME05.2` — Máquina de Estados de Controle de Escopo Contextual (`ScopeController`)
- **Labels:** `sub-issue`, `sintatico-escopo`, `prioridade:alta`
- **Descrição:**
  Encapsular a lógica de transição de `ContextoEscopo` em uma classe dedicada (`ScopeController`) para manter `StaticChecker` limpo e facilmente testável:
  - Estado inicial: `CORPO_INSTRUCOES`.
  - Ao receber `A19` (`PROGRAM`) → próximo contexto passa a ser `GLOBAL_PROGRAM`. Assim que um token `C03` (ou qualquer token inesperado que encerre a expectativa) for lido, retorna imediatamente para `CORPO_INSTRUCOES`.
  - Ao receber `A14` (`FUNCTYPE`) → entra em subestado `ESPERANDO_TIPO_FUNCAO` → ao receber um tipo (`A01`, `A03`, `A16`, `A20`, `A22`, `A25`) → entra em `ESPERANDO_DOIS_PONTOS_FUNCAO` → ao receber `B03` (`:`) → próximo contexto passa a ser `DECLARACAO_FUNCAO`. Logo após reconhecer `C02` (ou token inesperado), retorna a `CORPO_INSTRUCOES`.
  - **Robustez:** Mesmo em arquivos-fonte sintaticamente incorretos (ex.: `program ; var1`), a máquina não deve ficar "presa" em `GLOBAL_PROGRAM` ou `DECLARACAO_FUNCAO`!
- **Critérios de Aceite:**
  - [ ] **TC05:** Em `program MeoProg declarations varType integer : x endDeclarations`, `MEOPROG` recebe `C03` e `X` recebe `C01`.
  - [ ] Em `funcType real : calculaMedia ( ? ) ... endFunction`, `CALCULAMEDIA` recebe `C02`.
  - [ ] Entradas quebradas/malformadas resetam o contexto de forma segura para `CORPO_INSTRUCOES`.
  - [ ] Testes unitários cobrindo múltiplas funções e declarações no mesmo arquivo.

---

### Sub-Issue `ME05.3` — Captura Contextual de Tipos Semânticos (`TipoSimb`)
- **Labels:** `sub-issue`, `sintatico-escopo`
- **Descrição:**
  Estender o controlador contextual (ou criar `DeclarationTypeTracker`) para acompanhar declarações da gramática de `BigHole2026-2` e atualizar o atributo `TipoSimb` na `ISymbolTableManager`:
  - **`varType <tipo> : v1 , v2 ;`** → atualiza `v1`, `v2` com o tipo escalar (`IN`, `FP`, `ST`, `CH`, `BL`, `VD`).
  - **`varType <tipo> [ ] : vet1 [ 10 ] , vet2 [ 20 ] ;`** → detecta `[ ]` antes de `:` e atualiza `vet1`, `vet2` com tipo de array (`AI`, `AF`, `AS`, `AC`, `AB`).
  - **`funcType <tipo> : fn`** → atualiza `fn` (`C02`) com o tipo de retorno (`IN`, `FP`, `ST`, `CH`, `BL`, `VD`).
  - **`paramType <tipo> : p1 , p2 [ 10 ]`** → atualiza `p1` com tipo escalar e, se `p2` for seguido de `[` (`B08`), promove o tipo de `p2` para o tipo de array correspondente!
  - Finaliza o contexto declarativo ao encontrar `;` (`B01`), `)` (`B07`), `endDeclarations` (`A06`), `functions` (`A13`) ou nova palavra-chave declarativa.
- **Critérios de Aceite:**
  - [ ] Variáveis escalares e vetores têm `TipoSimb` corretamente preenchido no `.TAB` (`IN`, `FP`, `ST`, `CH`, `BL`, `VD`, `AI`, `AF`, `AS`, `AC`, `AB`).
  - [ ] Funções têm seu tipo de retorno preenchido no `.TAB`.
  - [ ] Parâmetros escalares e vetores têm seus respectivos tipos preenchidos no `.TAB`.
  - [ ] Identificadores usados sem declaração prévia (ex.: trecho solto como `Teste.262`) mantêm `TipoSimb: -` exatamente como no exemplo da especificação.

---

### Sub-Issue `ME05.4` — Loop Principal *Syntax-Driven* e Integração Completa
- **Labels:** `sub-issue`, `sintatico-escopo`, `core`
- **Descrição:**
  Implementar o método `StaticChecker.executarAnalise(Path caminhoArquivo)`:
  1. Instanciar `ReservedTable`, `SymbolTableManager`, `LexicalAnalyzer` (dentro de `try-with-resources`), `ScopeController` e `DeclarationTypeTracker`.
  2. Executar o loop principal:
     ```java
     List<Token> tokens = new ArrayList<>();
     Token token;
     while ((token = scanner.proximoToken(scopeController.getContextoAtual())) != null) {
         tokens.add(token);
         scopeController.atualizar(token);
         typeTracker.processar(token, symbolTable);
     }
     ```
  3. Invocar o `IReportGenerator` para gravar `<nomeBase>.LEX` e `<nomeBase>.TAB` na mesma pasta do arquivo `.262`.
  4. Exibir resumo sucinto de conclusão no console (arquivo analisado, total de tokens, total de símbolos e caminhos dos relatórios gerados).
- **Critérios de Aceite:**
  - [ ] Recursos de arquivo são sempre fechados corretamente (`try-with-resources`).
  - [ ] Fluxo completo executa de ponta a ponta sem acoplamento direto a `System.exit` dentro de `executarAnalise`, permitindo testes automatizados de integração.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 4 Sub-Issues (`ME05.1` a `ME05.4`) concluídas, revisadas e integradas na `main`.
- [ ] Execução via CLI com `Teste.262` completa sem erros e aciona a geração de `Teste.LEX` e `Teste.TAB`.
- [ ] `TC05` (desambiguação de `programName`, `functionName` e `variable`) validado com sucesso.
