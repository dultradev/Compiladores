
A síntese tem como papel **construir o texto objeto** a partir da representação intermediária validada pela análise. Também costuma ser dividida em sub-etapas, onde vamos  entender o papel de cada uma a partir do código de exemplo:

```C
int final = base + (5 * 2) + zero * y;
```
_Suponha que, em algum lugar antes, o compilador já sabe que a variável `zero` vale `0`._
#### 1. Geração de Código Intermediário


>**Objetivo Principal**
>*Traduzir para lógica de formato de máquina*

- **Papel**: traduzir a representação validada (árvore anotada) para uma **representação intermediária** — um código que não é nem o texto fonte original, nem ainda o código de máquina final, mas um formato intermediário mais simples e uniforme 
- Essa etapa existe para **desacoplar** a lógica de análise da lógica de geração de código final, facilitando, por exemplo, a portabilidade. Esta etapa traduz a árvore sintática em uma linguagem abstrata, independente da máquina (independente se o seu processador é Intel, AMD ou ARM)
- **Exemplo:** Se o compilador gerasse o código diretamente da árvore sintática, ele calcularia absolutamente tudo, passo a passo, criando várias variáveis temporárias (`Aux1`, `Aux2`, etc.):

```
Aux1 = 5 * 2
Aux2 = base + Aux1
Aux3 = zero * y
final = Aux2 + Aux3
```

#### 2. Otimização de Código

>**Objetivo Principal**
>*Melhorar código intermediário gerado*

- **Papel**: melhorar o código intermediário (ou, em alguns modelos, o código final) **sem alterar seu significado/comportamento** (preservando a equivalência de significados que vimos).
- Objetivos comuns: reduzir tempo de execução, reduzir uso de memória, eliminar código redundante ou morto, simplificar expressões.
- **Importante**: O otimizador analisa o código intermediário para torná-lo mais rápido ou menor, eliminando redundâncias. Essa etapa é opcional em termos de "existência de um compilador funcional" (um compilador pode gerar código correto sem otimizar), mas é essencial para gerar código **eficiente**.
- **Exemplo:** O otimizador entra em ação aplicando três técnicas clássicas de otimização nesta mesma linha:
	- **Técnica 1: Dobradura de Constante (_Constant Folding_)**  
	    O otimizador vê `5 * 2`. Ele pensa: _"O resultado disso sempre será 10. Por que fazer o processador calcular isso toda vez que o programa rodar?"_. Ele pré-calcula e substitui por `10`.
	- **Técnica 2: Simplificação Algébrica**  
	    Ele vê `zero * y`. Como ele sabe que a variável `zero` vale `0`, qualquer número multiplicado por 0 é 0. Ele apaga a multiplicação e substitui tudo por `0`.
	- **Técnica 3: Eliminação de Elemento Neutro**  
	    Agora o código está virando `base + 10 + 0`. Ele sabe que somar `0` não muda nada, então ele simplesmente joga o `+ 0` no lixo.
	 - **Resultado:**
	   ```
									final = base + 10
	   ```

#### 3. Geração de Código (Final)

>**Objetivo Principal**
>*Melhorar código intermediário gerado*

- **Papel**: traduzir o código intermediário (já otimizado, se aplicável) para a **linguagem objeto final** — geralmente Assembly ou código de máquina, dependendo da arquitetura do compilador estudado no pipeline anterior.
- Envolve decisões específicas de baixo nível: alocação de registradores, seleção de instruções específicas da arquitetura alvo, gerenciamento de endereços de memória.
- **Exemplo:** Para o código otimizado (`final = base + 10`), o **Gerador de Código Final** precisa traduzir essa instrução lógica para a linguagem de máquina real do processador. Agora sim o compilador olha para a arquitetura do seu computador (por exemplo, **x86/x64** da Intel/AMD) e traduz o código otimizado para a linguagem de montagem real daquela máquina (**Assembly**). Ele também decide em quais registradores físicos do processador as variáveis vão ficar.

```Assembly
mov R1, [base]       ; 1. Copia o valor da variável 'base' da RAM para o registrador R1
add R1, 10           ; 2. Soma a constante 10 diretamente no valor dentro de R1
mov [final], R1      ; 3. Salva o novo valor de R1 de volta na memória na variável 'final'

```
 
 - **Saída**: o texto objeto final, que seguirá para as próximas etapas do pipeline (montador, linker, loader) como vimos antes.