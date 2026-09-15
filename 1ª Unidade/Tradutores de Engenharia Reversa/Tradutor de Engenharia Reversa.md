São [[Tradutor]] que partem de um **texto objeto** (já traduzido — código de máquina, executável, código compilado) e tentam **reconstruir informações sobre o texto fonte original** (ou uma versão aproximada/equivalente dele), invertendo o sentido usual de uma tradução.

```
     ▲  [Alto Nível]       Código-Fonte (Go, C, Java)
     │                           ▲           │
     │                           │           ▼ [Ofuscador]
     │                    [Decompilador]  Código Embaralhado
     │                           │           │
     │  [Médio Nível]      Assembly Simbólico (x86, ARM)
     │                           ▲
     │                           │ [Desmontador / Disassembler]
     │                           │
     ▼  [Baixo Nível]      Binário Nativo / Bytecode (0s e 1s)
```

### Características principais:

- Invertem o **fluxo tradicional** de tradução: em vez de ir de fonte → objeto, vão de objeto → (uma aproximação da) fonte.
- Como muita informação é **perdida** durante o processo original de tradução (compilação/montagem) — nomes de variáveis, comentários, estrutura original do código, certas otimizações que reorganizam a lógica — a reconstrução feita pela engenharia reversa é, em geral, **aproximada**, e não perfeita. Não se recupera 100% do texto fonte original.
- É por isso que o termo mais amplo é "tradutor de engenharia reversa" — dentro dele existem categorias com **objetivos e níveis de abstração diferentes**: decompiladores e desmontadores.

`Texto objeto (código de máquina/executável) → [TRADUTOR DE ENGENHARIA REVERSA] → Aproximação do texto fonte`
