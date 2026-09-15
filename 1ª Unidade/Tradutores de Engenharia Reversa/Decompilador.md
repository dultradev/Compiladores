O decompilador é um [[Tradutor de Engenharia Reversa]] de engenharia reversa que tenta tenta o salto conceitual mais complexo da engenharia reversa: traduzir código de máquina nativo ou bytecode intermediário de volta para uma **linguagem de programação de alto nível estruturada** (como C ou Java).

![[Pasted image 20260912222542.png]]

Características principais:

- Busca produzir uma saída **próxima da linguagem de programação original** (ex: algo parecido com C, Java), com estruturas como laços (`for`, `while`), condicionais (`if/else`), funções — tentando recuperar a lógica de alto nível.
- É o "oposto" conceitual do [[Compilador]]: enquanto o compilador vai de alto nível → baixo nível, o decompilador tenta ir de baixo nível → alto nível.
- Como informações de alto nível (nomes de variáveis, comentários, nomes de funções, estrutura original) geralmente não existem mais no código objeto, o decompilador **infere** essas estruturas, o que torna o resultado aproximado e nem sempre fiel ao código original.