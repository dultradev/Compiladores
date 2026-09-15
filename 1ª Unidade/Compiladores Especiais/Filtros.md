Um filtro é um programa que transforma um programa fonte escrito em linguagem de programação de alto nível de entrada em outro formato de saída (um código objeto em outra linguagem de programação de alto nível), aplicando alguma transformação ou regra específica — **sem necessariamente envolver a tradução completa de uma linguagem de programação formal** (com todas as etapas de análise léxica, sintática e semântica).

>[!note]
> Diferente de um transpilador, o filtro geralmente não muda a linguagem nem a semântica do programa; ele foca em normalização, formatação, saneamento ou instrumentação.

![Imagem](<../images/Pasted image 20260912220158.png>)
### Características principais:

- É um conceito mais **genérico** que o de compilador/tradutor propriamente dito: o processamento pode ser mais simples, como reformatação de texto, remoção/substituição de padrões, conversão de codificação, etc.
- Nem todo filtro exige uma gramática formal complexa por trás — muitas vezes envolve apenas reconhecimento de padrões (ex: expressões regulares) e substituição direta.
- É um termo que ajuda a **delimitar** o que é (e o que não é) objeto central de estudo de compiladores: um filtro pode até compartilhar semelhanças estruturais com um tradutor, mas normalmente não realiza uma tradução completa e formal de uma linguagem para outra.

Exemplo: um programa que lê um arquivo de texto e remove todas as linhas em branco, ou que converte todas as letras para maiúsculas, é tecnicamente um "filtro" — ele transforma entrada em saída, mas não está traduzindo entre linguagens formais no sentido estudado em compiladores.

