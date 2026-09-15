Um **compilador** é um [[Tradutor]] que é um software de sistema (software básico) que traduz um texto-fonte escrito em uma **linguagem fonte de programação de alto nível** para um texto-objeto formulado em uma linguagem objeto de programação de alto ou baixo nível, preservando a sua semântica original a fim de viabilizar e propiciar a sua execução futura.


>O senso comum costuma restringir a saída de um compilador a código de máquina (Assembly/binário). No entanto, conceitualmente:
>
>- Um compilador tradicional gera código de **baixo nível** (ex.: Assembly, código de máquina x86-64/ARM, ou bytecode de JVM).
>- Um compilador _Source-to-Source_ ( [[Transpilador]] ) gera outra linguagem de **alto nível** (ex.: TypeScript para JavaScript, Babel transformando ECMAScript moderno em legado, ou cfront que traduzia C++ para C).

### Características
- **Tradução antecipada (ahead-of-time)**: 
	- todo o programa é traduzido de uma vez, gerando um arquivo executável (ou objeto) completo, que só depois será executado — separando claramente a etapa de _compilação_ da etapa de _execução_.
	- O compilador gera um **artefato estático** desacoplado do tempo de execução (_Ahead-of-Time_ ou pré-processamento), enquanto o interpretador analisa e executa a instrução concomitantemente.
- Como o programa inteiro é analisado antes de rodar, o compilador consegue detectar muitos erros **antes da execução** (erros de sintaxe, tipos, etc.) e também aplicar **otimizações** mais profundas, já que tem uma visão completa do programa.
![Imagem](<images/Pasted image 20260912205034.png>)

### Tempo de Compilação (Compile Time)
É a fase em que o compilador analisa, valida e traduz o código-fonte em artefato binário/objeto.
- O processador executa o compilador, **não o seu programa**.
- Valida-se tudo o que pode ser deduzido estaticamente: sintaxe, compatibilidade de tipos, visibilidade de variáveis e bibliotecas importadas.
- Se houver erros nesta fase (como tentar somar uma string com um inteiro em uma linguagem estática), a geração do binário é abortada antes que o programa chegue a existir como executável.
### Tempo de Execução (Runtime)
É o momento em que o sistema operacional carrega o binário na memória RAM e a CPU passa a executar as instruções de máquina contidas nele.
- O código-fonte original e o compilador já não precisam mais estar presentes no sistema.
- Ocorrem eventos que só podem ser determinados dinamicamente: acesso à rede, leitura de disco, alocação dinâmica de memória no _Heap_ e tratamento de exceções/pânicos.

| ==**Aspecto**==      | ==**Tempo de Compilação (Compile Time)**==                          | ==**Tempo de Execução (Runtime)**==                                                                   |
| -------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Quem executa**     | A CPU executa o compilador.                                         | A CPU executa o seu programa compilado.                                                               |
| **Dados conhecidos** | Apenas valores constantes e literais conhecidos no código.          | Dados dinâmicos fornecidos pelo usuário, rede ou arquivos.                                            |
| **Erros típicos**    | Sintaxe inválida, incompatibilidade de tipos, escopos inexistentes. | Divisão por zero, falta de memória (_OOM_), ponteiro nulo (_nil pointer dereference_), falhas de I/O. |
| **Frequência**       | Ocorre uma vez (ou sempre que o código-fonte é alterado).           | Ocorre quantas vezes o usuário desejar rodar o binário gerado.                                        |