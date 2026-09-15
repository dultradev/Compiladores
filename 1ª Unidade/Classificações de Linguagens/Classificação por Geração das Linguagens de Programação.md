Classificação histórica e evolutiva baseada na distância em relação aos circuitos de hardware:

- **1ª Geração (1GL) — Linguagem de Máquina:**
    
    - Instruções codificadas diretamente em binário (ou representação hexadecimal).
        
    - Sem qualquer abstração; entrada feita manualmente nos painéis ou cartões perfurados.
        
    - _Exemplo:_ Sequências cruas de opcodes como `10110000 01100001`.
        
- **2ª Geração (2GL) — Linguagem de Montagem (_Assembly_):**
    
    - Introdução de mnemônicos legíveis por humanos (`MOV`, `ADD`, `JMP`) e símbolos para substituir endereços de memória brutos.
        
    - Relação quase 1:1 com a ISA do processador; exige o utilitário montador (_assembler_).
        
    - _Exemplo:_ NASM, MASM, GNU Assembler.
        
- **3ª Geração (3GL) — Linguagens Estruturadas de Alto Nível:**
    
    - Salto para independência de hardware com a introdução de compiladores/interpretadores completos.
        
    - Introdução de álgebra matemática legível, estruturas de dados, laços e procedimentos formais.
        
    - Base da infraestrutura computacional moderna.
        
    - _Exemplos:_ Fortran, COBOL, C, Pascal, Java, Go, Rust, Python.
        
- **4ª Geração (4GL) — Linguagens de Domínio Específico e Alta Abstração:**
    
    - Projetadas para resolver problemas de nicho específicos sem forçar o usuário a lidar com algoritmos de baixo nível.
        
    - Foco massivo em bancos de dados, relatórios e manipulação de conjuntos de dados.
        
    - _Exemplos:_ SQL (bancos relacionais), R/MATLAB (estatística/cálculo matricial), ABAP.
        
- **5ª Geração (5GL) — Linguagens Baseadas em Conhecimento e Lógica:**
    
    - Projetadas originalmente para inteligência artificial clássica e sistemas especialistas.
        
    - O desenvolvedor fornece uma base de conhecimento e restrições lógicas; a linguagem usa algoritmos internos de _backtracking_ e resolução de restrições para deduzir as respostas.
        
    - _Exemplos:_ Prolog, Mercury, OPS5.

Essa classificação é **histórica/evolutiva**, agrupando linguagens conforme sua proximidade com o hardware e o nível de abstração alcançado ao longo do tempo:

|Geração|Nome|Características|Exemplos|
|---|---|---|---|
|**1ª Geração (1GL)**|Linguagem de máquina|Código binário puro, diretamente executável pelo processador; totalmente dependente de máquina|Código de máquina|
|**2ª Geração (2GL)**|Linguagem de montagem (Assembly)|Usa mnemônicos em vez de binário puro; ainda dependente de máquina; requer montador|Assembly|
|**3ª Geração (3GL)**|Linguagens de alto nível (procedurais/imperativas)|Independentes de máquina; mais próximas da linguagem humana; requerem compilador/interpretador|C, Pascal, Java, Python|
|**4ª Geração (4GL)**|Linguagens voltadas a domínios específicos, mais declarativas|Focadas em produtividade e em tarefas específicas (consultas a banco de dados, geração de relatórios)|SQL, MATLAB|
|**5ª Geração (5GL)**|Linguagens baseadas em resolução de problemas/restrições, próximas de IA|O programador especifica restrições/objetivos, e o sistema tenta resolver o problema (raciocínio lógico, IA)|Prolog, Mercu|
>[!note]
> O enquadramento das **redes neurais e da inteligência artificial (modelos de fundação, LLMs e programação conexionista) como a "6ª Geração" (6GL)** é uma tese recorrente na indústria e na divulgação científica, mas **não é um consenso taxonômico formal na ciência da computação**.

Pontos importantes para reter:

- Quanto **maior a geração**, geralmente **maior o nível de abstração** e **menor a dependência de máquina** (ligando essa classificação diretamente à classificação #1).
- As gerações também refletem uma evolução histórica real: das linguagens mais antigas e primitivas (1GL, anos 1940-50) até abordagens mais modernas e abstratas (4GL/5GL).
- Vale notar que a **classificação por geração** não é totalmente consensual/rígida na literatura (alguns autores discordam sobre o que conta como 4GL ou 5GL) — mas para fins de prova, o importante é entender a lógica evolutiva: **menos abstração/mais controle de hardware → mais abstração/mais produtividade**.