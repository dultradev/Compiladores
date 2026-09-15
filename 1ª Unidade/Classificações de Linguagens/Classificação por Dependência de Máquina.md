Essa classificação avalia o quanto uma linguagem está "presa" às características específicas do hardware (arquitetura do processador, conjunto de instruções, organização de memória).

```
[ Abstração / Domínio Específico ]
     ▲
     │  POL   (Problem-Oriented: SQL, R, MATLAB, VHDL)
     │  VHLL  (Very High-Level: Python, Ruby, Elixir, Shell)
     │  HLL   (High-Level: C, Go, Rust, Java, C++)
     │  LLL   (Low-Level: Assembly, Código de Máquina)
     ▼
[ Físico / Dependente de Hardware ]
```

![Imagem](<../images/Pasted image 20260913155912.png>)

### 1. LLL (_Low-Level Languages_)

Linguagens que operam em contato direto com a arquitetura física da CPU.

  

- **Características:**
    - Dependência estrita da ISA (_Instruction Set Architecture_);
    - Não há independência de plataforma — o código manipula registradores físicos (`rax`, `r0`), endereços literais de memória e flags aritméticas;
    - Relação quase 1:1 com os opcodes do silício.
        
- **Exemplos:** Código de máquina binário (1GL) e Assembly x86/ARM/RISC-V (2GL).
    
      
    

### 2. HLL (_High-Level Languages_)

Linguagens de propósito geral (_General-Purpose Languages_ - GPL) que introduzem independência de arquitetura através de tradutores (compiladores/interpretadores), mas ainda exigem que o programador descreva a lógica algorítmica fundamental.

- **Características:**
    - Uso de estruturas de controle estruturadas (`if`, `for`, `switch`), tipos de dados abstratos e sub-rotinas;
    - Independência da CPU: o mesmo algoritmo roda em arquiteturas diferentes bastando recompilar;
    - Controle explícito (ou semi-explícito) sobre estruturas de dados, concorrência e memória.
        
- **Exemplos:** C, Go, Rust, C++, Java, Pascal (tradicionalmente enquadradas nas 3GLs).
    
### 3. VHLL (_Very High-Level Languages_)

Linguagens que priorizam a produtividade humana em detrimento da eficiência de máquina por meio de altíssimo grau de automação e expressividade. Abstraem completamente os detalhes da máquina e focam em **o que** fazer, não em **como** fazer (declarativas).

- **Características:**
    - Gerenciamento de memória 100% automático e runtime pesado (coletor de lixo, reflexão dinâmica profunda);
    - Tipagem altamente dinâmica ou abstrações funcionais de primeira classe com ricas coleções embutidas (dicionários, tuplas, slices elásticos, comprehensions);
    - Uma única linha em uma VHLL frequentemente substitui dezenas de linhas de uma HLL tradicional.

- **Exemplos:** SQL, GraphQL, Terraform, YAML, MATLAB

### 4. POL (_Problem-Oriented Languages_)

Diferente das categorias anteriores — que são de propósito geral (_GPL_) —, as POLs são concebidas para **modelar e resolver problemas dentro de uma área de aplicação muito restrita e bem definida**. Elas são as precursoras do que a engenharia de software moderna batizou de **DSLs** (_Domain-Specific Languages_).

  

- **Características:**
    - Sintaxe desenhada para expressar diretamente os axiomas do domínio em vez de rotinas de computação genérica;
    - Frequentemente declarativas (expressam _o que_ resolver, não _como_ computar);
    - Não são necessariamente Turing-completas (embora algumas sejam em ambientes isolados).
        
- **Exemplos Clássicos:**
    - **GPSS (General Purpose Simulation System):** Desenvolvida pela [IBM](https://www.ibm.com/), é o exemplo mais famoso de POL. Ela serve exclusivamente para simular sistemas de filas e processos (como o fluxo de clientes em um banco ou carros em um pedágio).
    - **STRESS (Structural Engineering Systems Solver):** Criada no MIT na década de 1960, serve apenas para engenheiros civis calcularem forças, deformações e tensões em estruturas e fôrmas de edifícios.
        

### Comparativo Estrutural

|Característica|**LLL** (Low-Level Language)|**HLL** (High-Level Language)|**VHLL** (Very High-Level Language)|**POL** (Problem-Oriented Language)|
|---|---|---|---|---|
|**Abstração do Hardware**|**Nenhuma ou quase nenhuma**. Acesso direto a registradores e memória.|**Média/Alta**. Esconde o hardware e gerencia a memória automaticamente.|**Altíssima**. O hardware e o sistema operacional ficam totalmente invisíveis.|**Total**. O foco é o jargão do problema, ignorando qualquer conceito de computação.|
|**Paradigma Principal**|Imperativo / Estruturado (Instruções diretas ao processador).|Imperativo, Orientado a Objetos, Funcional.|**Declarativo** (focado no resultado, não no processo).|**Declarativo ou Paramétrico** voltado ao modelo do problema.|
|**Como o código é escrito?**|Diz ao processador **exatamente o que fazer** e como mover cada bit.|Diz ao computador o passo a passo lógico (**como fazer** o programa rodar).|Diz ao sistema apenas **o que você quer** como resultado final.|Descreve as variáveis do problema técnico (ex: pesos, forças, filas).|
|**Propósito de Uso**|Sistemas operacionais, drivers e firmware.|Desenvolvimento de softwares gerais, web, mobile e IAs.|Banco de dados, automação de infraestrutura e IA lógica.|Engenharia, simulações científicas e automação industrial.|
|**Produtividade do Dev**|**Muito baixa**. Exige dezenas de linhas para somar dois números.|**Alta**. Código legível por humanos e reaproveitável.|**Altíssima**. Comandos de uma linha resolvem problemas complexos.|**Máxima para o especialista** da área; inútil para qualquer outra coisa.|
|**Exemplos Clássicos**|Assembly, Código de Máquina.|Python, Java, C#, C++.|SQL, Prolog, Terraform (HCL).|GPSS (simulação), APT (CNC), STRESS (engenharia).|
