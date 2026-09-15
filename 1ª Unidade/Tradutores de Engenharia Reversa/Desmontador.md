O desmontador é um [[Tradutor de Engenharia Reversa]] realiza o caminho estritamente inverso do [[Montador]] (_Assembler_). Ele recebe uma sequência binária de bytes brutos (código de máquina) e a traduz de volta para **mnemônicos em linguagem Assembly legível**.

![[Pasted image 20260912222336.png]]

### Características principais:
- Diferente do decompilador, o desmontador produz uma saída de **baixo nível** (mnemônicos Assembly), mais próxima e fiel ao código de máquina original — já que existe uma correspondência quase direta entre instrução de máquina e instrução Assembly.
- Por isso, o resultado de um desmontador costuma ser **mais preciso/confiável** do que o de um decompilador (há menos "inferência" envolvida, já que a tradução é quase 1:1), mas também é mais difícil de ler para quem não conhece Assembly.