# [ME07] Validação, Testes End-to-End, Empacotamento e Entrega Final (`Etapa 7 - LEX`)

> **Milestone:** `M4: Validação, Testes & Entrega Final`  
> **Labels:** `macroentrega`, `testes`, `entrega`, `documentacao`  
> **Dependências:** `ME05`, `ME06`  
> **Bloqueia:** Submissão final do Static Checker no Google Classroom / Canvas

---

## 1. Objetivo

Garantir a qualidade final do Static Checker através de uma bateria completa de testes ponta a ponta (*end-to-end*) com arquivos `.262` reais, auditar cada detalhe dos relatórios `.LEX` e `.TAB` frente à especificação oficial [Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md), gerar o artefato executável pronto para avaliação do professor e montar o pacote da **Etapa 7 (`LEX`)**.

---

## 2. Requisitos da Entrega Final (Etapa 7 — `LEX`)

Conforme a seção *Entregas do projeto* e o item 18 do *Roteiro sugerido* em [Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md):
- Código-fonte completo e organizado do Static Checker (programa principal com controle de escopo e inicializações, analisador léxico e tabela de símbolos).
- Executável correspondente funcionando e produzindo `.LEX` e `.TAB` na mesma pasta do arquivo `.262` informado.
- Pequena documentação do projeto (como compilar, como executar, arquitetura final e eventuais ajustes em relação à Etapa 1 `PROJ`).
- Estrutura de pasta padrão da entrega (ex.: `EQ04LEX` / `E04LEX` — confirmar convenção final da turma) + planilha `ENTREGA` preenchida no Classroom.

---

## 3. Sub-Issues

- [ ] **ME07.1** — Criar suíte de testes E2E com `Teste.262` oficial e cenários mínimos `TC01`–`TC06`
- [ ] **ME07.2** — Criar suíte de testes de estresse, programas completos em `BigHole2026-2` e casos de borda
- [ ] **ME07.3** — Auditar conformidade dos relatórios `.LEX`/`.TAB` e corrigir eventuais divergências/bugs
- [ ] **ME07.4** — Gerar executável, redigir documentação final e montar pacote da Entrega `Etapa 7 (LEX)`

---

## 4. Detalhamento das Sub-Issues

### Sub-Issue `ME07.1` — Suíte E2E: `Teste.262` Oficial e Casos Mínimos `TC01`–`TC06`
- **Labels:** `sub-issue`, `testes`, `prioridade:alta`
- **Descrição:**
  Criar pasta de fixtures (ex.: `src/test/resources/fixtures/`) contendo arquivos `.262` reais e testes automatizados JUnit que executam o `StaticChecker` de ponta a ponta e validam os arquivos `.LEX` e `.TAB` gerados:
  - `Teste.262` (exemplo literal da especificação oficial).
  - `TC01_FiltroPrimeiroNivel.262` (`va@r#x` → `VAR`, `#`, `X`).
  - `TC02_IdentificadorLongo.262` (identificador com 48 caracteres válidos → truncado em 30, `QtdAntes: 48`, `QtdDepois: 30`).
  - `TC03_ComentarioBlocoAberto.262` (`a := 10; /* comentario infinito...`).
  - `TC04_ConstanteRealExpoente.262` (`3.14159E+10`, `0.5e-3`, `2.0e4`).
  - `TC05_DesambiguacaoEscopo.262` (`program MeoProg declarations ... functions funcType real : calc ( ? ) ...`).
  - `TC06_TetoOcorrenciasTab.262` (símbolo aparecendo em 7 linhas distintas → apenas as 5 primeiras no `.TAB`).
- **Critérios de Aceite:**
  - [ ] Todos os 7 arquivos `.262` versionados no repositório e executados automaticamente na suíte de testes.
  - [ ] Asserções verificam tanto o conteúdo completo do `.LEX` quanto do `.TAB`.

---

### Sub-Issue `ME07.2` — Testes de Estresse, Programa Completo e Casos de Borda
- **Labels:** `sub-issue`, `testes`
- **Descrição:**
  Elaborar arquivos `.262` complexos que exercitam toda a gramática de `BigHole2026-2` e armadilhas conhecidas:
  - Programa completo válido contendo `program`, `declarations` (variáveis escalares e vetores de todos os tipos), `functions` (funções com `?` e com `paramType` escalar/vetor), blocos `{ ... }`, `if/else/endIf`, `while/endWhile`, `print`, `return`, `break`, expressões lógicas com todos os operadores (`<=`, `<`, `>`, `>=`, `==`, `!=`, `#`) e aritméticas (`+`, `-`, `*`, `/`, `%`).
  - **Casos de borda obrigatórios:**
    - Truncagem de `realConst` onde o 30º caractere cairia sobre `.`, `E`, `E+` ou `E-`.
    - Truncagem de `stringConst` com > 30 caracteres (fechando com `"` na posição 30) e string não fechada antes de quebra de linha/EOF.
    - `_var_1` válido como `variable` (`C01`) vs `_` como delimitador em `programName`/`functionName`.
    - Caracteres inválidos misturados em números, strings, palavras reservadas e operadores.
    - Comentários consecutivos (`/* a */// b\n/* c */`), `/***/`, comentário de linha sem `\n` no EOF.
    - Arquivo `.262` vazio e arquivo contendo apenas comentários/espaços.
    - Execução com caminho relativo, caminho absoluto e diretório contendo espaços no nome (ex.: `Projeto Compiladores/Teste`).
    - Mistura de quebras de linha `\r\n` e `\n` verificando exatidão do número das linhas.
- **Critérios de Aceite:**
  - [ ] 100% dos cenários executam sem `Exception`, loop infinito ou corrupção de tokens.
  - [ ] Todos os 11 códigos de `TipoSimb` (`FP`, `IN`, `ST`, `CH`, `BL`, `VD`, `AF`, `AI`, `AS`, `AC`, `AB`) testados e validados no `.TAB`.

---

### Sub-Issue `ME07.3` — Auditoria de Conformidade e Correção de Bugs
- **Labels:** `sub-issue`, `testes`, `prioridade:alta`
- **Descrição:**
  Realizar revisão coletiva com os 4 integrantes comparando o comportamento real do Static Checker contra cada parágrafo de [Projeto2026-2 vSN1.md](../Projeto2026-2%20vSN1.md):
  - Checklist de conferência de todos os 26 códigos `A01`–`A26`, 23 símbolos `B01`–`B22` e 7 códigos `C01`–`C07`.
  - Conferência de cabeçalhos, nomes de equipe, e-mails, telefones e código oficial da equipe.
  - Conferência de eventuais decisões de sala de aula / WhatsApp / Classroom (seção *Questões Omissas* da especificação).
  - Abertura e correção de quaisquer bugs encontrados antes de congelar a versão de entrega.
- **Critérios de Aceite:**
  - [ ] Checklist de conformidade revisado pelos 4 integrantes.
  - [ ] Zero bugs abertos com severidade alta ou média.

---

### Sub-Issue `ME07.4` — Empacotamento do Executável, Documentação e Pacote da Entrega (`Etapa 7 - LEX`)
- **Labels:** `sub-issue`, `entrega`, `documentacao`, `prioridade:alta`
- **Descrição:**
  Preparar todos os artefatos necessários para entrega final da disciplina:
  1. Configurar build do JAR executável (`StaticChecker.jar` com `Main-Class: br.ucsal.bighole.StaticChecker` no `MANIFEST.MF`) e scripts auxiliares de execução (`run.sh` / `run.bat`, se útil).
  2. Testar a execução do JAR fora do IntelliJ IDEA, diretamente via linha de comando (`java -jar StaticChecker.jar MeuTeste`), em diretório distinto daquele onde o JAR está salvo.
  3. Escrever o `README.md` técnico final do projeto contendo: visão geral da arquitetura, pré-requisitos (JDK 27), instruções de compilação/execução, estrutura dos módulos e registro de ajustes feitos em relação à `Etapa1_Proj`.
  4. Montar a pasta de entrega padrão (`EQ04LEX` / `E04LEX`) com: código-fonte, executável, documentação resumida e planilha `ENTREGA` atualizada.
- **Critérios de Aceite:**
  - [ ] `java -jar StaticChecker.jar <caminho_sem_extensao>` funciona perfeitamente fora da IDE (macOS/Windows/Linux).
  - [ ] Documentação final clara, enxuta e revisada por toda a equipe.
  - [ ] Todos os 4 integrantes revisaram o código final e estão aptos a explicar qualquer módulo na arguição/apresentação.
  - [ ] Pacote de entrega verificado e pronto para submissão no Classroom/Canvas.

---

## 5. Critérios de Aceite da Macroentrega

- [ ] Todas as 4 Sub-Issues (`ME07.1` a `ME07.4`) concluídas e integradas na `main`.
- [ ] Release tag criada no GitHub (ex.: `v1.0.0-etapa7-lex`).
- [ ] Static Checker 100% validado, empacotado e pronto para entrega final da disciplina.
