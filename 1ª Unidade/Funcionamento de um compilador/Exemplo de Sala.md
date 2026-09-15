![Imagem](<../images/Pasted image 20260914212736.png>)



#### Texto fonte:

```Go
POS := 5 + 7 * TEMPO
```

Vamos dissecar o texto-fonte **`POS := 5 + 7 * TEMPO`** ao longo do modelo **Análise-Síntese**, mostrando exatamente como os dados e as estruturas internas são transformados em cada etapa.

### Bloco de Análise

#### 1. Análise Léxica (_Scanner / Tokenizer_)
O compilador lê o texto caractere por caractere (eliminando espaços) e gera uma lista de **tokens**:

- **`Id1`** $\to$ Identificador 1 (`POS`)
- **`SA`** $\to$ Símbolo de Atribuição (`:=`)
- **`CO1`** $\to$ Constante 1 (`5`)
- **`SS`** $\to$ Símbolo de Soma (`+`)
- **`CO2`** $\to$ Constante 2 (`7`)
- **`SM`** $\to$ Símbolo de Multiplicação (`*`)
- **`ID2`** $\to$ Identificador 2 (`TEMPO`)

#### 2. Análise Sintática (_Parser_)
O analisador sintático lê os tokens organizados:

$$\text{Id1} \quad \text{SA} \quad \text{CO1} \quad \text{SS} \quad \text{CO2} \quad \text{SM} \quad \text{ID2}$$

Ele consome os tokens e aplica as regras de gramática livre de contexto, **respeitando a precedência de operadores** (a multiplicação tem prioridade sobre a adição). O resultado é a **Árvore de Sintaxe Abstrata (AST)**:

```
           SA
         /    \
      Id1      SS
              /   \
           CO1      SM
                  /   \
               CO2     ID2
```

De forma visual:

```

        Atribuição (:=)
        /             \
   Ident (POS)        Op (+)
                    /       \
                  (5)        Op (*)
                           /       \
                         (7)   Ident (TEMPO)

```


Neste momento, a árvore não possui metadados de tipo nem validações:

- O compilador apenas sabe que existe uma expressão de multiplicação (`*`) entre `CO2` e `ID2`.
- O resultado dessa multiplicação é somado (`+`) com `CO1`.
- O resultado total é atribuído (`:=`) a `Id1`.

#### 3. Análise Semântica (_Type-Checking & Tabela de Símbolos_)
O compilador consulta e valida os metadados na **Tabela de Símbolos**:

- **Índice (_Index / Pointer_):** A posição física ou chave primária da entrada na tabela (muitas vezes o índice de um array ou o bucket de uma _hash table_). É esse número que o analisador léxico costuma associar ao token (ex.: passar `<id, 0>` em vez de repetir a string toda vez).
    
- **Lexema (_Lexeme_):** A cadeia de caracteres bruta e literal lida do código-fonte original (o texto real que o programador digitou).
    
- **Tipo (_Type_):** A classificação semântica e estrutural do símbolo (preenchida ou validada pelo analisador semântico: `float`, `int`, `void`, etc.).
    
- **Código (_Token Code / Category_):** O código numérico ou mnemônico do átomo léxico que o parser utiliza para tomar decisões gramaticais (ex.: `ID`, `CONST_INT`, `CONST_FLOAT`).
    
- **Linha (_Line Number_):** A coordenada espacial no arquivo de texto onde o símbolo foi encontrado pela primeira vez. É essencial para:
    - Emissão de mensagens de erro diagnósticas úteis (_"Erro de tipo na linha 14"_);
    - Construção das tabelas de símbolos para _debuggers_ (mapeamento linha-código em DWARF/PDB).

| **Índice** | **Lexema** | **Tipo** | **Código**             | **Linha** |
| ---------- | ---------- | -------- | ---------------------- | --------- |
| **0**      | `POS`      | `float`  | `ID` (ou `Id1`)        | 1         |
| 1          | `:=`       |          | `SA`                   | 1         |
| **2**      | `5`        | `int`    | `CONST_INT` (ou `CO1`) | 1         |
| **3**      | `+`        |          | `SS`                   | 1         |
| **4**      | `7`        | `int`    | `CONST_INT` (ou `CO2`) | 1         |
| **5**      | `*`        |          | `SM`                   | 1         |
| **6**      | `TEMPO`    | `float`  | `ID` (ou `Id2`)        | 1         |


Ao confrontar a AST com esses tipos, o analisador semântico detecta incompatibilidades de domínio operacional:

1. **Primeiro conflito no nó `(*)`:** O operando esquerdo (`CO2`) é `int`, mas o direito (`ID2`) é `float`. Operações aritméticas da CPU exigem operandos de mesmo tipo.
    
2. **Resolução:** O analisador insere um nó intermediário de **coerção/conversão implícita** (`InttoFloat`) acima de `CO2`.
    
3. **Segundo conflito no nó `(+)`:** O operando esquerdo (`CO1`) é `int`, mas o operando direito (o resultado da multiplicação) agora é `float`.
    
4. **Resolução:** O analisador insere outro nó de conversão (`InttoFloat`) acima de `CO1`.
    
5. **Verificação final no nó `(:=)`:** O lado direito resulta em `float`, e o lado esquerdo (`Id1`) é `float`. Tipos compatíveis, validação concluída.
    

A árvore resultante — agora chamada de **Árvore Sintática Decorada / Estendida** (ou _Decorated AST_) — ganha a seguinte forma:

```
               (:=) [float]
              /            \
    Id1 [float]             (+) [float]
                           /           \
               InttoFloat [float]       (*) [float]
                       |               /           \
                   CO1 [int]   InttoFloat [float]   ID2 [float]
                                       |
                                   CO2 [int]
```



---

### Bloco de Síntese

#### 4. Geração de Código Intermediário (IR)

A árvore hierárquica é "achatada" em instruções lineares de três endereços (_Three-Address Code_ / TAC) usando registradores virtuais temporários.

```
1. Visita CO2 (7) ──► Nó InttoFloat ──► Aux1 := InttoFloat(CO2) 
2. Visita CO1 (5) ──► Nó InttoFloat ──► Aux2 := InttoFloat(CO1) 
3. Visita Nó (*) ──► Multiplicação ──► Aux3 := Aux1 * ID2 
4. Visita Nó (+) ──► Adição ──► Aux4 := Aux2 + Aux3 
5. Visita Nó (:=) ──► Atribuição ──► Id1 := Aux4
```

No exemplo da aula, foi assumido que a variável de entrada `Id2` (`TEMPO`) ou o resultado esperado em `Id1` (`POS`) é de **ponto flutuante** (`float`), enquanto os números literais escritos no código (`5` e `7`) são **inteiros** (`int`).

A CPU não consegue multiplicar diretamente um registrador inteiro por um registrador de ponto flutuante — eles usam formatos binários completamente incompatíveis (complemento de dois vs. padrão IEEE 754).

Por isso, na geração do código intermediário bruto (não otimizado), o compilador é obrigado a inserir instruções explícitas de conversão

```
Aux1 := InttoFloat(co2)   // Converte o inteiro 7 para o float 7.0
Aux2 := InttoFloat(co1)   // Converte o inteiro 5 para o float 5.0
Aux3 := Aux1 * Id2        // Agora multiplica: float * float
Aux4 := Aux2 + Aux3       // Agora soma: float + float
Id1  := Aux4              // Atribui o resultado final a Id1
```

#### 5. Otimização Independente de Máquina

O compilador percebe que `co2` (`7`) e `co1` (`5`) são literais constantes e estáticos. Não há motivo para gastar instruções da CPU convertendo números inteiros para float toda vez que o programa rodar.

- O próprio compilador converte os números durante a compilação:
    
    - `InttoFloat(7)` vira diretamente o literal `7.0`
    - `InttoFloat(5)` vira diretamente o literal `5.0`
        
- Com isso, as instruções `Aux1 := ...` e `Aux2 := ...` são completamente **eliminadas**.
    

##### B. _Copy Propagation_ e Redução de Registradores Temporários

Observe o final do código bruto:

```
Aux4 := Aux2 + Aux3
Id1  := Aux4
```

Guardar o resultado em `Aux4` para imediatamente depois passá-lo para `Id1` é uma instrução redundante. O compilador colapsa esse passo gravando a soma diretamente no destino final `Id1`.

#### O Resultado Final Explicado

Aplicando essas duas transformações sobre o código bruto, chegamos com precisão ao código otimizado da lousa:

```
Aux3 := 7.0 * Id2    // O 7 já foi convertido para 7.0 em tempo de compilação
Id1  := 5.0 + Aux3   // O 5 já é 5.0 e o resultado vai direto para Id1, eliminando Aux4
```

#### 6. Gerador de Código Final
O compilador escolhe as instruções da arquitetura (exemplo em **Assembly x86-64**) e aloca as variáveis aos registradores físicos. Neste exemplo seguiremos à risca o modelo clássico e didático de uma **máquina de dois endereços** com suporte a registradores de ponto flutuante (_Floating Point_, indicado pelo sufixo **`F`** nas instruções).

### Mapeamento Linha a Linha

1. **`MOVF Id2, R1`**
    
    - Carrega o valor em ponto flutuante da variável `Id2` (`TEMPO`) da memória para o registrador de trabalho `R1`.
        
    - _Estado:_ $R1 = \text{Id2}$.
        
2. **`MULF 7.0, R1`**
    
    - Realiza a multiplicação em ponto flutuante: multiplica o conteúdo de `R1` pela constante `7.0`.
        
    - _Estado:_ $R1 = \text{Id2} \times 7.0$.
        
3. **`MOVF R1, Aux3`**
    
    - Grava o resultado intermediário do registrador `R1` na variável/área temporária de memória `Aux3`.
        
    - _Efeito:_ Finaliza a tradução exata da instrução intermediária: `Aux3 := 7.0 * Id2`.
        
4. **`MOVF Aux3, R1`**
    
    - Recarrega o valor de `Aux3` da memória de volta para o registrador `R1` para iniciar a próxima instrução.
        
5. **`ADDF 5.0, R1`**
    
    - Executa a adição em ponto flutuante: soma a constante `5.0` ao valor presente em `R1`.
        
    - _Estado:_ $R1 = \text{Aux3} + 5.0$.
        
6. **`MOVF R1, Id1`**
    
    - Descarrega o resultado final contido em `R1` diretamente no endereço de memória da variável destino `Id1` (`POS`).
        
    - _Efeito:_ Finaliza a tradução exata de: `Id1 := 5.0 + Aux3`.

### Observação de Engenharia: A Otimização de Janela (_Peephole Optimization_)

Note as linhas 3 e 4 geradas pelo processo ingênuo (_naive code generation_):

```
MOVF R1, Aux3    ; Salva R1 na memória
MOVF Aux3, R1    ; Lê imediatamente da memória de volta para o mesmo R1
```

Esse é o exemplo de livro-texto para introduzir a próxima fase do compilador: a **Otimização de Janela (_Peephole Optimization_)**.

O compilador analisa um pequeno intervalo de instruções sequenciais (_a janela_) e identifica que o valor de `Aux3` já reside no registrador `R1`. Caso a variável `Aux3` não seja utilizada em nenhuma outra parte do programa adiante, essas duas instruções de acesso à memória RAM/Stack tornam-se completamente redundantes e podem ser removidas:

#### Código final

```
MOVF Id2, R1
MULF 7.0, R1
ADDF 5.0, R1
MOVF R1, Id1
```

![Imagem](<../images/Pasted image 20260914223544.png>)