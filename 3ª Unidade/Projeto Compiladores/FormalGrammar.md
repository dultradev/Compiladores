## Gramática para a sintaxe dos comandos:


**G** = (`V, Σ, P, FileProgram`)

**V** = {
```
FileProgram, DeclarationList, DeclarationVar, TypeSpecification, VariableListVetor,
VariableList, FunctionList, DeclarationFunc, Parameters, ParamTypeList, ParamType,
ParamList, Param, Command, BlockCommand, ListCommand, IfCommand,
WhileCommand, ReturnCommand, PauseCommand, PrintCommand, AtribCommand,
Variable, LogicalExp, AritmEx, AddExp, RelatOperator , AddOper, Term, MultOper,
UnaryExp, Factor, Constant, Numbers, Strings, program, programName, declarations,
endDeclaracions, functions, endFunctions, endProgram, ; , varType, : , [ , ] , : , , ,
variable, intConst, real, integer, string, boolean, character, void, funcType,
functionName, (,) , endFunction, ?, paramType, { , }, if, endIf, else, while, endWhile,
return, break, print, :=, <= , <, >, >=, == , !=, #, -, +, *, /, %, true, false, realConst,
stringConst, charConst
```
}

**Σ** = {
```
program, programName, declarations, endDeclaracions, functions, endFunctions,
endProgram, ; , varType, : , [ , ] , : , , , variable, intConst, real, integer, string,
boolean, character, void, funcType, functionName, (,) , endFunction, ?, paramType, { ,
}, if, endIf, else, while, endWhile, return, break, print, :=, <= , <, >, >=, == , !=, #, -, +, *,
/, %, true, false, realConst, stringConst, charConst
```
}

**P** = {
```
FileProgram -> program programName declarations DeclarationList endDeclararions
functions FunctionList endFunctions endProgram ,
FileProgram -> program programName declarations DeclarationList endDeclararions
endProgram ,
DeclarationList -> DeclarationList ; DeclarationVar ,
DeclarationList -> DeclarationVar ,
DeclarationVar -> varType TypeSpecification: VariableList ,
DeclarationVar -> varType TypeSpecification [ ]: VariableListVetor ,
VariableList -> VariableList , variable ,
VariableList -> variable ,
VariableListVetor -> VariableListVetor , variable [ intConst ] ,
VariableListVetor -> variable [ intConst ] ,
TypeSpecification -> real ,
TypeSpecification -> integer ,
TypeSpecification -> string ,
TypeSpecification -> boolean ,
TypeSpecification -> character ,
TypeSpecification -> void ,
FunctionList -> FunctionList ; DeclarationFunc ,
FunctionList -> DeclarationFunc ,
DeclarationFunc -> funcType TypeSpecification: functionName ( Parameters )
Command endFunction ,
Parameters -> ParamTypeList ,
Parameters -> ? ,
ParamTypeList -> ParamTypeList ; ParamType ,
ParamTypeList -> ParamType ,
ParamType -> paramTypeTypeSpecification: ParamList ,
ParamList -> ParamList , Param ,
ParamList -> Param ,
Param -> variable ,
Param -> variable [ intConst ] ,
Command -> BlockCommand ,
Command -> AtribCommand ,
Command -> PrintCommand ,
Command -> IfCommand ,
Command -> WhileCommand ,
Command -> ReturnCommand ,
Command -> PauseCommand ,
BlockCommand -> { ListCommand } ,
BlockCommand -> Command ,
ListCommand -> ListCommand ; Command ,
ListCommand -> Command ,
IfCommand -> if ( LogicalExp ) Command endIf ,
IfCommand -> if ( LogicalExp ) Command else Command endIf ,
WhileCommand -> while ( LogicalExp ) Command endWhile ,
ReturnCommand -> return ,
ReturnCommand -> return AritmExp ,
PauseCommand -> break ,
PrintCommand -> print AritmExp ,
AtribCommand -> Variable := AritmExp ,
AtribCommand -> Variable := LogicalExp ,
Variable -> variable ,
Variable -> variable [ intConst ] ,
LogicalExp -> LogicalExp RelatOperator AritmExp ,
LogicalExp -> ( LogicalExp ) ,
LogicalExp -> AritmExp ,
AritmExp -> AddExp ,
RelatOperator -> <= ,
RelatOperator -> < ,
RelatOperator -> > ,
RelatOperator -> >= ,
RelatOperator -> == ,
RelatOperator -> != ,
RelatOperator -> # ,
AddExp -> AddExp AddOper Term ,
AddExp -> Term ,
AddOper -> - ,
AddOper -> + ,
Term -> Term MultOper UnaryExp ,
Term -> UnaryExp ,
MultOper -> * ,
MultOper -> / ,
MultOper -> % ,
UnaryExp -> - UnaryExp ,
UnaryExp -> Factor ,
Factor -> ( AritmExp ) ,
Factor -> Variable ,
Factor -> Constant ,
Constant -> Numbers ,
Constant -> Strings ,
Constant -> true ,
Constant -> false ,
Numbers -> intConst ,
Numbers -> realConst ,
Strings -> stringConst ,
Strings -> charConst
}
```