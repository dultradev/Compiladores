O ofuscador é um [[Compilador]]/[[Filtros]] defensivo que reescreve o código antes da distribuição pública, preservando rigorosamente o comportamento lógico observável, mas **maximizando o custo cognitivo e computacional da engenharia reversa**.

![Imagem](<../images/Pasted image 20260912223157.png>)

Consiste em remover informações relacionadas a depuração como tabelas de símbolos,
número de linhas, renomear pacotes, classes, métodos, variáveis. Apesar das informações de depuração não serem necessárias para a execução do código, elas são utilizadas pelos depuradores, o que ajuda na descompilação do código.
### Características principais:

- Diferente dos outros tradutores estudados, o ofuscador **não muda o nível de abstração nem a linguagem** necessariamente — o objetivo não é traduzir entre linguagens, mas **dificultar a compreensão** mantendo o comportamento (equivalência de significado é preservada, mas a legibilidade é deliberadamente destruída).
- Técnicas comuns: renomear variáveis/funções para nomes sem sentido, remover formatação e comentários, reestruturar o fluxo lógico de forma desnecessariamente complexa, inserir código morto ou redundante.
- **Motivação principal**: proteger propriedade intelectual e dificultar engenharia reversa (dificultar justamente a ação de decompiladores/desmontadores sobre o código).

O ofuscador conecta-se diretamente com os [[Tradutor de Engenharia Reversa]] — ele existe, em boa parte, **como uma contramedida** a decompiladores e desmontadores, tornando o trabalho deles mais difícil.