Um transpilado (*Transpilers ou Source-Source compilers*) é um tradutor que converte um texto fonte escrito em uma linguagem de programaçã oem alto nível para outro texto objeto escrito em **outra linguagem de programação de nível de abstração semelhante** (ambas são linguagens de alto nível legíveis por humanos).

![[Pasted image 20260912215058.png]]

### Características principais:

- A diferença central em relação a um compilador "tradicional" está no **nível de abstração**: um compilador tradicional normalmente traduz de alto nível para baixo nível (ex: C → código de máquina); um transpilador traduz **entre linguagens de nível parecido** (alto nível → alto nível).
- Também são chamados de **compiladores fonte-a-fonte** (source-to-source compilers), justamente porque tanto a entrada quanto a saída são "código fonte" legível/editável por humanos, e não código de máquina.
- O texto objeto gerado geralmente ainda precisa passar por outro processo de tradução (compilação ou interpretação) para ser efetivamente executado.

#### Exemplos clássicos:

- **TypeScript → JavaScript**
- **Sass/SCSS → CSS** (embora CSS não seja uma linguagem de programação no sentido estrito, o princípio é o mesmo)
- **CoffeeScript → JavaScript**
- Transpiladores entre versões diferentes da mesma linguagem (ex: código Python 2 → Python 3)
Código fonte (linguagem A, alto nível) → [TRANSPILADOR] → Código fonte (linguagem B, alto nível)
