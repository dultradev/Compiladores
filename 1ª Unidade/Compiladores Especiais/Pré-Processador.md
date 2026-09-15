Um pré-processador é um programa que realiza transformações no **texto fonte de entrada (obrigatoriamente um programa fonte escrito em linguagem de programação de alto nível) antes da etapa de compilação propriamente dita**, produzindo como saída um código objeto em outra linguagem de programação de alto nível com vistas a ser fornecida como entrada para um compilador na sequencia do processo de tradução.

![[Pasted image 20260912220933.png]]

## Características principais:

- Atua como uma **etapa preliminar**, separada (ou embutida no início) do processo de compilação.
- Trabalha tipicamente por meio de **diretivas** especiais dentro do próprio texto fonte, que instruem o pré-processador sobre o que fazer (essas diretivas não fazem parte da linguagem de programação "final" em si).


## Operações comuns realizadas por pré-processadores:

#### 1. Inclusão de Arquivos (`#include`)

A inclusão de arquivos é uma operação de **interpolação textual recursiva**. Quando o pré-processador encontra `#include`, ele suspende a leitura do arquivo atual, abre o arquivo apontado e "cola" o conteúdo integral no ponto exato da diretiva.

```
arquivo.c:                         math.h:
┌─────────────────────┐            ┌─────────────────────┐
│ #include "math.h"   │ ──► Abre ─►│ int soma(int, int); │
│ int main() { ... }  │            └─────────────────────┘
└─────────────────────┘                       │
           │                                  │
           ▼  (Substituição direta)           ▼
┌────────────────────────────────────────────────────────┐
│ int soma(int, int);                                    │
│ int main() { ... }                                     │
└────────────────────────────────────────────────────────┘
```

#### Mecanismos de Busca

- **`#include <arquivo.h>`:** Busca nos diretórios padrão do sistema operacional e da toolchain (ex.: `/usr/include`, `/usr/local/include`).
    
- **`#include "arquivo.h"`:** Busca primeiro no diretório relativo ao arquivo-fonte atual; caso não encontre, recorre aos caminhos do sistema.

### 2. Definição, Processamento e Utilização de Macros (`#define`)

Internamente, o pré-processador mantém uma **Tabela de Macros** (um mapa de _chave-valor_ em memória). Cada vez que uma macro é definida, ela é registrada nessa tabela até o fim da tradução ou até que uma diretiva `#undef` a remova.

#### A. Macros Simples (Constantes Textuais)

```C
#define TAMANHO_MAX 1024
int buffer[TAMANHO_MAX];
```

- **Processamento:** O pré-processador armazena a chave `"TAMANHO_MAX"` vinculada à cadeia `"1024"`. Ao varrer o código, onde encontrar o token exato, realiza a substituição pura: `int buffer[1024];`.
    

#### B. Macros com Argumentos (Funções Textuais)

```C
#define QUADRADO(x) ((x) * (x))
int res = QUADRADO(a + 1);
```

- **Processamento:**
    
    1. Identifica os parâmetros formais da macro (`x`).
        
    2. Isola o argumento passado na chamada (`a + 1`).
        
    3. Substitui todas as ocorrências de `x` no padrão de substituição por `(a + 1)`:
        
        $$\text{res} = ((a + 1) * (a + 1));$$
        

> **Risco de Efeitos Colaterais:** Como a expansão é puramente textual, passar expressões com mutação gera avaliações repetidas:
> 
> `QUADRADO(a++)` expande para `((a++) * (a++))`, incrementando a variável duas vezes de forma indeterminada.

#### C. Operadores Especiais de Processamento de Macros

- **Stringificação (`#`):** Converte o argumento recebido em uma string literal entre aspas duplas:
    
    ```C
    #define PRINT_VAR(x) printf(#x " = %d\n", x)
    PRINT_VAR(idade); // Expande para: printf("idade" " = %d\n", idade);
    ```
    
- **Concatenação de Tokens (`##`):** Une dois tokens em um único identificador léxico válido:
    
    ```C
    #define DECLARAR_VAR(tipo, id) tipo var_##id
    DECLARAR_VAR(int, contador); // Expande para: int var_contador;
    ```

### 3. Expansão de Linguagem (_Language Extension_)

A expansão de linguagem refere-se à capacidade do pré-processador de **criar dialetos sintáticos, abstrair construções repetitivas ou emular recursos que a linguagem nativa não possui formalmente**.

Isso transforma a sintaxe bruta antes que o compilador formal precise interpretá-la.

#### A. Criação de DSLs (_Domain-Specific Languages_) Embutidas

É possível remodelar a aparência da linguagem para aproximá-la de um domínio específico ou torná-la mais expressiva:


```C
// Redefinindo palavras-chave para criar uma sintaxe alternativa
#define SE if
#define ENTAO {
#define SENAO } else {
#define FIM }

SE (x > 0) ENTAO
    y = 1;
SENAO
    y = 0;
FIM
```

O compilador tradicional jamais saberia lidar com `SE` ou `ENTAO`, mas após o pré-processamento, a saída gerada é puro C estruturado padrão (`if (x > 0) { ... }`).

#### B. Emulação de Recursos Modernos (Generics e Polimorfismo Falso)

Antes da introdução de tipos genéricos em linguagens procedurais clássicas, o pré-processador era a única ferramenta para gerar estruturas de dados reutilizáveis por meio de "templates textuais":


```C
// Macro geradora de estruturas tipadas (Vector/Stack)
#define DECLARAR_PILHA(TIPO, NOME)             \
    typedef struct {                            \
        TIPO itens[100];                        \
        int topo;                               \
    } Pilha_##NOME;                             \
    void push_##NOME(Pilha_##NOME *p, TIPO val) { \
        p->itens[++p->topo] = val;              \
    }

// Expandindo implementações concretas:
DECLARAR_PILHA(int, Int)       // Cria Pilha_Int e push_Int
DECLARAR_PILHA(float, Float)   // Cria Pilha_Float e push_Float
```

#### C. Abstração de Atributos de Plataforma

O compilador real varia entre fornecedores (GCC, Clang, MSVC) e possui comandos específicos para otimização ou alinhamento de memória. O pré-processador homogeneíza isso:

```C
#if defined(__GNUC__)
    #define INLINE __attribute__((always_inline)) inline
#elif defined(_MSC_VER)
    #define INLINE __forceinline
#else
    #define INLINE inline
#endif

// O desenvolvedor usa uma sintaxe universal:
INLINE void rotina_rapida() { ... }
```

| **Mecanismo**             | **O que o Pré-processador Faz**                                                          | **Efeito no Compilador**                                                                      |
| ------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Inclusão (`#include`)** | Lê arquivos externos do disco e insere o texto diretamente no fluxo de caracteres.       | O compilador recebe um único arquivo monolítico consolidado (`.i`).                           |
| **Macros (`#define`)**    | Registra chaves em uma tabela interna e executa substituições/interpolações textuais.    | O compilador só enxerga os literais e código resultantes, sem vestígios de macros.            |
| **Expansão de Linguagem** | Modifica ou combina tokens estruturais usando `#`, `##` e blocos de código paramétricos. | Permite expressar metaprogramação, templates rudimentares e compatibilidade multi-plataforma. |



