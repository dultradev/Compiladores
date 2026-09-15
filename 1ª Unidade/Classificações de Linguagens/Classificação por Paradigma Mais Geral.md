No nível conceitual mais amplo, toda linguagem responde a uma pergunta filosófica de execução: **você dita o caminho passo a passo ou descreve o resultado esperado?**

```
                  PARADIGMA GERAL
                 /               \
                ▼                 ▼
          IMPERATIVO          DECLARATIVO
        ("Como fazer")      ("O que obter")
```

- **Imperativo:**
    
    - Foco no **fluxo de controle explícito** e na **mutação de estado**.
    - O desenvolvedor comanda a CPU instrução por instrução: _"incremente esta variável, verifique esta condição, salte para aquele ponto"_.
    - Modela fielmente a arquitetura física de Von Neumann (CPU buscando instruções sequenciais na memória).
        
- **Declarativo:**
    
    - Foco na **especificação lógica do problema** ou relações matemáticas.
    - O desenvolvedor expressa _o que_ precisa ser computado, delegando ao motor de execução (planner de banco de dados, motor de inferência ou runtime) a escolha do melhor algoritmo para chegar ao resultado.
    - Minimiza ou elimina variáveis de estado mutável observáveis.

|                  | Imperativa                 | Declarativa                      |
| ---------------- | -------------------------- | -------------------------------- |
| Foco             | Como fazer (passo a passo) | O que fazer (resultado desejado) |
| Conceito central | Estado e comandos          | Expressões e relações            |
| Exemplo          | C, Java                    | SQL, Prolog                      |
