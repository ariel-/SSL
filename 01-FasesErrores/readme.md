# TP #1 - Fases de la Traducción y Errores

## Autores
Usuario | Legajo | Apellido | Nombre
------- | ------ | -------- | ------
[ariel-](https://github.com/ariel-) | 166216-8 | Silva | Ariel

## Enunciado
### Objetivos
* Identificar las fases de traducción y errores.

### Temas
* Fases de traducción.
* Preprocesamiento.
* Compilación.
* Ensamblado.
* Vinculación (Link).
* Errores en cada fase.

### Tareas
1. Investigar las funcionalidades y opciones que su compilador presenta para limitar el inicio y fin de las fases de traducción.
2. Para la siguiente secuencia de pasos:
    1. Transicribir en `readme.md` cada comando ejecutado y
    2. Describir en `readme.md` el resultado u error obtenidos para cada paso.

#### Secuencia de Pasos
1. Escribir `hello2.c`, que es una variante de `hello.c`:
    ```c
    #include <stdio.h>
    
    int/*medio*/main(void){
      int i=42;
      prontf("La respuesta es %d\n");
    ```
2. Preprocesar `hello2.c`, no compilar, y generar `hello2.i`. Analizar su contenido.
3. Escribir `hello3.c`, una nueva variante:
    ```c
    int printf(const char *s, ...);

    int main(void){
      int i=42;
      prontf("La respuesta es %d\n");
    ```
4. Investigar la semántica de la primera línea.
5. Preprocesar `hello3.c`, no compilar, y generar `hello3.i`. Buscar diferencias entre `hello3.c` y `hello3.i`.
6. Compilar el resultado y generar `hello3.s`, no ensamblar.
7. Corregir en el nuevo archivo `hello4.c` y empezar de nuevo, generar `hello4.s`, no ensamblar.
8. Investigar `hello4.s`.
9. Ensamblar `hello4.s` en `hello4.o`, no vincular.
10. Vincular `hello4.o` con la biblioteca estándar y generar el ejecutable.
11. Corregir en `hello5.c` y generar el ejecutable.
12. Ejecutar y analizar el resultado.
13. Corregir en `hello6.c` y empezar de nuevo.
14. Escribir `hello7.c`, una nueva variante:
    ```c
    int main(void){
      int i=42;
      printf("La respuesta es %d\n", i);
    }
    ```
15. Explicar porqué funciona

### Restricciones
* El programa ejemplo debe enviar por `stdout` la frase `La respuesta es 42`, el valor 42 debe surgir de una variable.

### Productos
* readme.md
* hello2.c
* hello3.c
* hello4.c
* hello5.c
* hello6.c
* hello7.c

## Documentación de los pasos
1. Copiado el contenido del bloque de código en `hello2.c`
2. Se ejecuta en Windows:
    ```
    cl.exe /EP hello2.c > hello2.i
    ```
    Se genera el `hello2.i`, el mismo incluye una gran cantidad de declaraciones y de internos del compilador (header `stdio.h`). Se observa que el preprocesamiento no es completo ya que aparece la directiva `#pragma once`.
    El comentario que separaba `int` de `main` desapareció, no fue reemplazado por espacio como vimos en clase.

    Obs: Utilizando gcc bajo WSL:
    ```
    gcc -E hello2.c -o hello2.i
    ```
    El espacio sí se genera al reemplazar el comentario. Se observa también que el .i generado es mucho más liviano que el obtenido por `cl`
3. Análogamente se copia el bloque de código en `hello3.c`.
4. La primera línea es una *declaración*. Declara la existencia de una función, de nombre `printf` y que retorna un `int`. Toma una cantidad variable de argumentos. El primero y el único nombrado se llama `format` y es un puntero a `char const`ante.
5. Al utilizar `cl`, tanto `hello3.c` como `hello3.i` son idénticos. La diferencia esta vez es al utilizar `gcc`. El mismo incluye algunas directivas al inicio del archivo: (transcripción)
    ```c
    # 1 "hello3.c"
    # 1 "<built-in>"
    # 1 "<command-line>"
    # 31 "<command-line>"
    # 1 "/usr/include/stdc-predef.h" 1 3 4
    # 32 "<command-line>" 2
    # 1 "hello3.c"
    ```
6. Error! Al generar el ensamblado mediante la _línea de comando_:
    ```
    cl.exe /FA /c hello3.c
    ```
    La misma devuelve el error: `hello3.c(3): fatal error C1075: '{': no matching token found`

    En WSL, con gcc utilizo:
    ```
    gcc -std=c11 -S hello3.c -o hello3.s
    ```
    El diagnóstico es más completo:
    ```
    hello3.c: In function ‘main’:
    hello3.c:5:3: warning: implicit declaration of function ‘prontf’; did you mean ‘printf’? [-Wimplicit-function-declaration]
       prontf("La respuesta es %d\n");
       ^~~~~~
       printf
    hello3.c:5:3: error: expected declaration or statement at end of input
    ```
    Se ha generado un _warning_ por la **declaración implícita** de `prontf`, sugiriendo su reemplazo por `printf`.
    Asimismo en ambos casos se genera un error por la falta del token `}`. En VC, porque no se encuentra el matcheo con `{`. En gcc porque _espera_ una declaración o sentencia, pero se ha encontrado con el fin del archivo.
7. El problema se soluciona en `hello4.c`, cerrando la llave de apertura de `main`. Al volver a ejecutar los comandos:

    En Windows, la cmdline se supone incompleta ya que además genera el archivo .obj. Aún asi se genera el .asm (equivalente del .s)
    Utilizando gcc, el único output del compilador es el `hello4.s`
8. En ambos archivos aparece un `call` a un símbolo `prontf`. En el caso del .asm de Windows también se referencia el mismo como `EXTERN`.
9. cmdline de ensamblado:
    ```
    ml64.exe /c hello4.asm
    ```
    Falla con referencias a símbolos no definidos:
    ```
    Microsoft (R) Macro Assembler (x64) Version 14.20.27508.1
    Copyright (C) Microsoft Corporation.  All rights reserved.

     Assembling: hello4.asm
    hello4.asm(39) : error A2006:undefined symbol : FLAT
    hello4.asm(11) : error A2006:undefined symbol : $LN3
    hello4.asm(12) : error A2006:undefined symbol : $LN3
    ```
    Obs: al parecer Microsoft no soporta ensamblar sus propios archivos _listing_, más informacion en [este SO](https://stackoverflow.com/questions/21182665/compile-64-bit-hello-world-assembly-output-from-msvc2012)
    ```
    gcc -c hello4.s -o hello4.o
    ```
    Genera `hello4.o`
10. cmdline:
    ```
    gcc hello4.o -o hello4.out
    ```
    Error!
    ```
    /usr/sbin/ld: hello4.o: in function `main':
    hello4.c:(.text+0x1c): undefined reference to `prontf'
    collect2: error: ld returned 1 exit status
    ```
    Esta vez el error está en la fase de _linkeo_ al no poder encontrar una _definición_ para `prontf`.
11. En `hello5.c`, asumo que la intención del programador original era llamar a `printf`, así que cambio la llamada por esa otra función.
    * cmdline (W): `cl.exe hello5.c`    
    * cmdline (L): `gcc -std=c11 hello5.c -o hello5.out`

    Esta vez el diagnóstico de Windows es el más completo. Se advierte de un argumento faltante en la función `printf`, debido al especificador de formato `%d`. No obstante compila de todas formas.
12. Al ejecutar `hello5.exe` o `./hello5`, se imprime por `stdout` (normalmente consola):
    ```
    La respuesta es: -1363582656
    ```
    Éste número varía en cada ejecución. Esto se debe a que `printf` esperaba un valor para imprimir. Al no pasárselo, el comportamiento es indefinido.
13. La corrección en este caso es pasar el argumento faltante a `printf`, de manera tal que su comportamiento sea determinístico, se opta por la constante numérica `0` de manera arbitraria.
    * cmdline (W): `cl.exe hello6.c`
    * cmdline (L): `gcc -std=c11 hello6.c -o hello6.out`

    No hay diagnósticos de parte de ningún compilador. El programa imprime correctamente en las ejecuciones de prueba:
    `La respuesta es 0`
14. Copiado contenido a `hello7.c`
    Al compilar el archivo con `gcc`, aparece un warning acerca de la declaración implícita de `printf`.
    Se sugiere incluir `<stdio.h>`, o bien proveer una declaración.
15. El mismo funciona porque al no existir declaración previa al uso, C asume que se está declarando en la propia invocación. Además el tipo de dato de retorno por defecto es `int`, lo cual no contradice la definición de `printf` en la biblioteca estándar.
