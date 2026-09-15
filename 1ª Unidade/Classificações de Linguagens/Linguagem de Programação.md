Uma **linguagem de programação** é um sistema formal de comunicação composto por um conjunto de regras gramaticais (sintaxe) e de significado (semântica), projetado para descrever algoritmos e emitir instruções precisas que um computador pode interpretar e executar.

Em termos fundamentais, ela atua como uma **ponte de abstração** entre o modelo mental humano (lógica, fluxo de dados e problemas do mundo real) e a realidade física da máquina (transistores, registradores, pulsos elétricos e aritmética binária).

### O Espectro de Abstração

As linguagens organizam-se hierarquicamente pelo grau de proximidade com o hardware:

```
[ Alto Nível ]      Python, Go, Rust, Java
         │          (Abstrações: Garbage Collection, Concorrência, Tipos Genéricos)
         ▼
[ Médio / Baixo ]   C, C++
         │          (Controle explícito de ponteiros, alocação de memória direta)
         ▼
[ Baixo Nível ]     Assembly (x86, ARM, RISC-V)
         │          (Mnemônicos mapeados quase 1:1 com instruções da CPU)
         ▼
[ Hardware ]        Código de Máquina (Bits / Opcodes executados pelo processador)
```

- **Baixo Nível (Código de Máquina e Assembly):** 
	- Dependente diretamente da microarquitetura da CPU (ISA). Opera com registradores físicos (`rax`, `w0`), endereços brutos de memória e flags de status.
    
- **Alto Nível:** 
	- Oculta a arquitetura subjacente. Permite portabilidade (o mesmo algoritmo roda em processadores com arquiteturas completamente distintas) e oferece estruturas como laços (`for`), tipos de dados complexos, interfaces e canais de concorrência.

>[!note]
> Na ciência da computação formal e na teoria de linguagens de programação, o termo **"médio nível" sequer existe como uma classificação técnica oficial**. Ele é considerado por muitos acadêmicos, autores clássicos e engenheiros de compiladores como um **jargão pedagógico ou mercadológico**, e não uma categoria taxonômica rigorosa.
> 
> **C e C++ são inequivocamente linguagens de alto nível**. Um código em C pode ser compilado tanto para um microcontrolador de 8 bits quanto para um supercomputador x86-64 sem alterar uma única linha de algoritmo.
> 
> O conceito de "médio nível" é tolerado na didática e em conversas informais para explicar de forma rápida linguagens que oferecem **acesso a detalhes de baixo nível sem abrir mão da portabilidade de alto nível**. No entanto, sob o ponto de vista da ciência da computação e da arquitetura de compiladores, essa fronteira intermediária não se sustenta como uma categoria formal independente.


O processo completo de transformar uma necessidade do mundo real em silício operando transistores não é um salto único; é uma **cadeia sucessiva de traduções e reduções de abstração**.

Cada elo dessa corrente tem uma única responsabilidade: receber uma representação mais abstrata e reescrevê-la em uma forma mais concreta, determinística e restrita.

