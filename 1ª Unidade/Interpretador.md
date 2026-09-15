um interpretador é um tradutor que analisa e **executa** o programa fonte diretamente, instrução por instrução (ou pequenos blocos por vez), **sem gerar um arquivo objeto separado e completo** previamente. O interpretador elimina a etapa de geração prévia de um binário independente, ele lê, valida e executa o código-fonte diretamente em **tempo de execução** (_runtime_).

>[!note]
> Na interpretação a tradução e a execução do código acontecem ao mesmo tempo, ou seja,
não existe tempo de compilação e tempo de execução, somente a interpretação.

### Características principais:

- Tradução e execução acontecem **juntas**, de forma intercalada — cada instrução é traduzida e imediatamente executada antes de passar para a próxima.
- Não há uma etapa de "compilação completa" separada da execução: o programa fonte é o que é fornecido diretamente ao interpretador toda vez que ele roda.
- Costuma ser mais lento em tempo de execução (comparado a um programa já compilado), pois a tradução ocorre repetidamente durante a execução, mas é mais **flexível** (permite, por exemplo, código gerado dinamicamente, execução linha a linha, ambientes interativos como REPL).
- Erros de sintaxe/semântica em partes do programa podem só ser detectados quando aquela parte específica é alcançada durante a execução (diferente do compilador, que analisa tudo antes).
