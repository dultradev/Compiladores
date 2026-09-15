Um montador é um [[Tradutor]] cuja linguagem fonte é a **linguagem Assembly** e cuja linguagem objeto é o **código de máquina** (linguagem binária executável pelo processador).
![[Pasted image 20260912180759.png]]

### Características
- **Mapeamento quase 1:1:** 
	- Ao contrário de compiladores, que transformam um comando simples em dezenas de instruções de máquina, a maioria das instruções Assembly mapeia diretamente para um único opcode do processador
	- Por isso, o processo de "tradução" feito pelo montador é relativamente simples comparado ao de um compilador: não há grandes transformações estruturais, principalmente:
		- Conversão de mnemônicos (ex: `MOV`, `ADD`, `JMP`) para seus respectivos opcodes binários.
		- Resolução de **rótulos/labels** (endereços simbólicos) para endereços de memória reais.
		- Tratamento de diretivas do montador (reserva de memória, definição de constantes, etc.).
	- Não envolve análise semântica complexa nem otimizações sofisticadas — é uma tradução mais "mecânica" e próxima do hardware.


Resumindo o fluxo:

```
Código Assembly (fonte) → [MONTADOR] → Código de Máquina (objeto)
```









