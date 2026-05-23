# Contexto: Simulador 8-bit Schweighauser (SI725)

## Registros
Solo existen A, B, C, D (8 bits, 0-255). No hay E, F ni otros.

## Instrucciones (SOLO estas)
MOV, ADD, SUB, MUL, DIV, INC, DEC, AND, OR, XOR, NOT, SHL, SHR
CMP, JMP, JZ, JNZ, JC, JNC, JA, JBE, CALL, RET, PUSH, POP, HLT, DB

**No existen:** JE, JNE, JGE, JLE, JL, JG, JAE, JB

## Reglas críticas
- `MOV A, label` = dirección de label (puntero). `MOV A, [label]` = valor almacenado.
- CMP: primer operando siempre registro. `CMP [B], 0` no funciona → usar `MOV C, [B]` + `CMP C, 0`
- INC/DEC: solo sobre registros. `INC [label]` no funciona → `MOV C, [x]` / `INC C` / `MOV [x], C`
- AND/OR/XOR destruyen el destino: `AND C, 1` → C queda en 0 o 1, no el original
- Contadores a 0: `MOV B, 0` (no `MOV B, nombreVariable`)

## Patrones obligatorios

**Estructura base:**
```
JMP start
var: DB 0
texto: DB "Hola"
       DB 0
start:
  ...
HLT
```

**Output (pantalla = dirección 232):**
```
MOV D, 232
MOV [D], A
INC D
```

**Imprimir número 2 dígitos (estilo profesor):**
```
MOV D, 232
MOV C, A
DIV 10
ADD A, 48
MOV [D], A
INC D
MOV A, C
MOV B, C
DIV 10
MUL 10
SUB B, A
MOV A, B
ADD A, 48
MOV [D], A
INC D
HLT
```

**Imprimir número 3 dígitos (con PUSH/POP):**
```
MOV D, 232
PUSH A
DIV 100
ADD A, 48
MOV [D], A
INC D
POP A
PUSH A
MOV B, A
DIV 100
MUL 100
SUB B, A
MOV A, B
PUSH A
DIV 10
ADD A, 48
MOV [D], A
INC D
POP A
MOV B, A
DIV 10
MUL 10
SUB B, A
MOV A, B
ADD A, 48
MOV [D], A
INC D
HLT
```

**Aritmética con valores en memoria:**
```
MOV A, [n1]
ADD A, [n2]
MOV A, [n1]
MUL [n2]
```

**Modificar variable en memoria:**
```
MOV C, [contador]
INC C
MOV [contador], C
```

**String invertido (recorrer hacia atrás hasta dirección base):**
```
MOV B, texto
.findEnd:
  MOV A, [B]
  CMP A, 0
  JZ .printBack
  INC B
  JMP .findEnd
.printBack:
  DEC B
  MOV A, B
  CMP A, texto
  JB .done
  MOV A, [B]
  MOV [D], A
  INC D
  DEC B
  JMP .printBack
.done:
```

**Recorrer string (termina en DB 0):**
```
MOV B, texto
.loop:
  MOV A, [B]
  CMP A, 0
  JZ .fin
  ...
  INC B
  JMP .loop
.fin:
```

**Comparar strings (punteros en A y B):**
```
.loop:
  MOV C, [A]
  MOV D, [B]
  CMP C, 0
  JZ igual
  CMP C, D
  JNZ diferente
  INC A
  INC B
  JMP .loop
```

## Responde
- Solo código limpio, sin comentarios
- Estrategia en 2-3 líneas antes del código
- Si hay error: pega el mensaje exacto del simulador y el código
