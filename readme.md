# Fases de la Traducción y Errores

Este trabajo tiene como objetivo identificar las fases del proceso de traducción o *Build* y los posibles errores asociados a cada fase.  
Para lograr esa identificación, se ejecutan las fases de traducción una a una, se detectan y corrigen errores, y se registran las conclusiones en `readme.md`.  
No es un trabajo de desarrollo; el programa usado como ejemplo es simple, similar a `hello.c`, pero con errores que se deben corregir. La complejidad está en la identificación y comprensión de las etapas y sus productos.

---

## 7.3. Tareas

### 1. Investigación de las fases de traducción

Las funcionalidades y opciones del compilador/implementador se pueden observar en las siguientes 4 fases de traducción:

1. **Preprocesador**: Sustituye macros y elimina comentarios. Genera un archivo `.i`.
2. **Compilador**: Convierte el código preprocesado en código ensamblador. Genera un archivo `.s`.
3. **Asembler**: Convierte el código ensamblador en código máquina o archivo objeto. Genera un archivo `.o`.
4. **Enlazador**: Une los archivos objeto con las bibliotecas necesarias para crear un ejecutable funcional.

### ¿Cómo limitar el inicio y fin de las fases de traducción?

- **Preprocesador (`-E`)**: Detiene el programa después del preprocesamiento.
- **Compilador (`-S`)**: Genera el código ensamblador sin ensamblar.
- **Asembler (`-c`)**: Genera un archivo objeto en lenguaje máquina.
- **Enlazador**: Por defecto, sin flags adicionales (`gcc archivo.o -o ejecutable`), completa el proceso de enlazado.

---

## 7.3.1 Tareas

### 1. Preprocesador

#### a. Código inicial:
```c
#include <stdio.h>
int/*medio*/main(void){
 int i=42;
 prontf("La respuesta es %d\n");
}
```

#### b. Observaciones:
1. El preprocesador modifica el código fuente (`hello.c`).
2. El archivo resultante es significativamente más grande debido al uso de librerías.
3. El preprocesador elimina los comentarios, haciendo más legible el código para el compilador.

---

### 2. Compilación

#### a. Código corregido:
```c
int printf(const char * restrict s, ...);
int main(void){
 int i=42;
 prontf("La respuesta es %d\n");
}
```

#### b. Análisis de errores:
- **Léxico**: Error en `prontf` (debería ser `printf`).
- **Sintáctico**: Falta de cierre de llaves.
- **Semántico**: Variable `i` no utilizada correctamente.

---

### 3. Ensamblador

#### Código ensamblador generado:
```assembly
.text: indica que empieza la sección de código 

.section .rodata: define un string de solo lectura

.LC0:
  .string "la respuesta es %d\n"  # Define el string de formato para printf

.text 
.globl main
.type main, @function

main:
.LFB0:
    .cfi_startproc
    endbr64  # Seguridad
    pushq	%rbp  # Guarda el valor actual de la base del stack
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq	%rsp, %rbp  # Establece una nueva base del stack
    .cfi_def_cfa_register 6
    subq	$16, %rsp  # Reserva 16 bytes en la pila para variables locales
    movl	$42, -4(%rbp)  # Guarda el número 42 en una variable local
    movl	-4(%rbp), %eax  # Carga ese valor en el registro eax
    movl	%eax, %esi  # Prepara el primer argumento para printf
    leaq	.LC0(%rip), %rax  # Carga la dirección del string
    movq	%rax, %rdi  # Pone esa dirección como primer argumento para printf
    movl	$0, %eax  # Limpia %eax
    call	printf@PLT  # Llama a printf
    movl	$0, %eax  # Devuelve 0 como valor de retorno
    leave  # Restaura el stack antes de salir de la función
    ret  # Sale de la función y vuelve al sistema operativo.
    .cfi_endproc
```

---

### 4. Vinculación

#### a. Intento de vinculación:
Se intenta vincular `hello4.o` con la biblioteca estándar para generar el ejecutable.

**Resultado**:  
El ejecutable no se puede generar debido a un error léxico en la declaración de la función. El enlazador no encuentra la función `prontf` en la biblioteca estándar.

#### b. Corrección mínima:
En `hello5.c`, se corrige `prontf` por `printf` y se genera el ejecutable.

**Resultado**:  
El ejecutable se genera con un warning, pero puede ejecutarse.

#### c. Ejecución:
El resultado es un número aleatorio, ya que no se asignó correctamente un valor entero.

---

### 5. Remoción de prototipo

#### a. Intento sin prototipo:
Se elimina el prototipo de `printf`.

**Resultado**:  
El compilador arroja 2 warnings y 1 error.

#### b. Análisis:
- **Prototipo de función**: Es una declaración anticipada que informa al compilador sobre el nombre, tipo de retorno y parámetros de una función.
- **Declaración implícita**: Ocurre cuando se usa una función sin declarar su prototipo. Esto está prohibido en C99 y versiones posteriores.
- **Especificación**: Según C99, toda función debe estar declarada antes de ser utilizada.
- **Implementaciones**:
  - GCC: Permite declaraciones implícitas en modo C90, pero lanza warnings o errores en C99/C11/C17.
  - Clang: Similar a GCC.

---

### Conclusiones

1. **Errores comunes**:
   - Declaraciones implícitas de funciones.
   - Uso incorrecto de prototipos.
   - Errores léxicos y semánticos.

2. **Buenas prácticas**:
   - Siempre incluir los encabezados necesarios.
   - Declarar prototipos de funciones antes de usarlas.
   - Usar estándares modernos de C (C99 o superior).

---

### Bibliografía

- [Alegsa](https://www.alegsa.com.ar/Diccionario/C/25059.php#gsc.tab=0)  
- [Guru99](https://www.guru99.com/es/compiler-design-phases-of-compiler.html)  
- [CppReference](https://en.cppreference.com/w/c/io/fprintf)  
- [Ray Notes](https://cs.lmu.edu/~ray/notes/x86assembly/)  
- [Shiksha](https://www.shiksha.com/online-courses/articles/function-prototype-in-c/)  