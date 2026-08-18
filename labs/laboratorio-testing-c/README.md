# Laboratorio: Testing en C

## Verificación y calificación

Desde la raíz del repositorio ejecutá:

```bash
make grade LAB=laboratorio-testing-c
```

Si ya estás dentro de esta carpeta también podés usar `make test`. Ambos
comandos ejecutan los mismos checks y muestran el puntaje sin necesidad de hacer
push. La verificación oficial se ejecuta cuando cambia
`labs/laboratorio-testing-c/`; la nota queda en el resumen y en los artefactos
de GitHub Actions.

No modifiques `.github/`, `scripts/` ni `test_local.sh`: son infraestructura
docente y el workflow restaura automáticamente su versión oficial.

---

## Antes de empezar

Para trabajar local necesitás GCC:
- **Linux:** `sudo apt install gcc`
- **macOS:** `xcode-select --install` o `brew install gcc`
- **Windows:** WSL con Ubuntu recomendado

Si querés trabajar sin instalar nada, usá **GitHub Codespaces**: hacé click en el botón verde `<> Code` → pestaña `Codespaces` → `Create codespace on main`.

---

## Introducción

Tenés que implementar el backend de un carrito de compras simple. El código ya está escrito en `src/carrito.c`, pero **tiene bugs**. Tu trabajo es escribir tests que los descubran y luego corregirlos.

El módulo expone estas funciones:

```c
void carrito_init(Carrito *c);
int  carrito_agregar(Carrito *c, Producto p);  /* 1=ok, 0=lleno */
int  carrito_contar(Carrito *c);
int  carrito_total(Carrito *c);
int  carrito_descuento(int total, int porcentaje);
```

Y estas estructuras (en `src/carrito.h`):

```c
typedef struct {
    char nombre[32];
    int  precio;    /* precio unitario en pesos */
    int  cantidad;
} Producto;

typedef struct {
    Producto items[MAX_ITEMS];  /* MAX_ITEMS = 4 */
    int      cantidad;
} Carrito;
```

---

## Parte 1: Correr el programa

Compilá y ejecutá el programa principal:

```bash
make programa
./programa
```

Vas a ver el carrito con tres productos y su total.

**P1** — Calculá a mano el total esperado: Leche ($350 x2) + Pan ($200 x3) + Queso ($1500 x1).

> R:2800

**P2** — ¿El total que imprime el programa coincide con lo que calculaste? Si no coincide, ¿cuánto muestra?

> R:NO

```
TOTAL_PROGRAMA=2050
```
_(escribí el número que imprimió el programa)_

---

## Parte 2: El framework de testing

Antes de seguir, abrí `tests/minunit/minunit.h`. Es el framework que usamos — apenas 60 líneas de C.

Los macros principales son:

```c
ASSERT_IGUAL(esperado, obtenido)  /* verifica que dos enteros sean iguales */
ASSERT_VERDADERO(condicion)       /* verifica que la condicion sea verdadera */
RESUMEN()                         /* imprime cuantos tests pasaron/fallaron */
```

Cuando todos los tests pasan, el programa termina con **exit code 0**. Si alguno falla, termina con **exit code 1**.

---

## Parte 3: Tests ya escritos

Compilá y corré los tests unitarios tal como están:

```bash
make test_unitarios
./test_unitarios
```

Deberías ver dos tests pasando:

```
=== Tests unitarios ===

[carrito nuevo]
  [OK]   carrito_contar(&c) == 0

[agregar un producto]
  [OK]   carrito_agregar(&c, p) == 1
  [OK]   carrito_contar(&c) == 1

--- 3/3 tests OK ---
```

Mirá el código en `tests/test_unitarios.c` para entender la estructura de un test.

**P3** — ¿Qué hace `carrito_init` y por qué es importante llamarla antes de usar el carrito?

> R:Inicializa el carrito sin productos para luego ir agregando dichos productos

---

## Parte 4: Agregar un test (copy-paste)

Ahora vas a agregar un test que verifica el cálculo del total para un producto con cantidad = 1.

Abrí `tests/test_unitarios.c` y pegá esta función en el lugar marcado como `/* PARTE A */`:

```c
void test_total_precio_unitario(void) {
    printf("\n[total: un producto, cantidad 1]\n");
    Carrito c;
    carrito_init(&c);
    Producto p = {"Leche", 350, 1};
    carrito_agregar(&c, p);
    ASSERT_IGUAL(350, carrito_total(&c));
}
```

Luego, en el `main()`, descomentá la línea:
```c
/* test_total_precio_unitario(); */
```

Compilá y corré:

```bash
make test_unitarios
./test_unitarios
```

**P4** — ¿El nuevo test pasa o falla?

> R:El nuevo test pasa y funciona

```
TEST_PRECIO_UNITARIO_PASA=SI
```
_(SI o NO)_

---

## Parte 5: Completar un test con blancos

El siguiente test verifica que el total tiene en cuenta la **cantidad** de cada producto.
Copialo en `tests/test_unitarios.c` (en el lugar `/* PARTE B */`) y completá los blancos:

```c
void test_total_con_cantidad(void) {
    printf("\n[total: un producto, cantidad 2]\n");
    Carrito c;
    carrito_init(&c);
    Producto p = {"Leche", 350, 2};  /* 350 x 2 = 700 */
    carrito_agregar(&c, p);
    ASSERT_IGUAL(____, carrito_total(&c));  /* <-- completar el valor esperado */
}
```

Descomentá `/* test_total_con_cantidad(); */` en el `main()`, compilá y corré.

**P5** — ¿Este test pasa o falla? ¿Qué valor esperaba y qué obtuvo?

> R:El test falla, porque espera 700 pero obtiene 350

```
TEST_TOTAL_CANTIDAD_PASA=NO
```
_(SI o NO)_

---

## Parte 6: Encontrar y corregir el bug

El test anterior encontró un bug en `carrito_total`. Abrí `src/carrito.c` y buscá esa función.

**P6** — ¿En qué línea está el bug y qué dice ese código?

> R:El bug esta en la linea 24. Lo que hace es sumar a total el precio de los productos

**P7** — ¿Qué debería hacer esa línea para calcular el total correctamente?

> R:Deberia multiplicar los precios de los productos por la cantidad

Corregí el bug. Luego volvé a compilar y correr:

```bash
make test_unitarios
./test_unitarios
```

Guardá la salida:

```bash
mkdir -p salidas
./test_unitarios > salidas/test_unitarios.txt 2>&1
cat salidas/test_unitarios.txt
```

```
TESTS_UNITARIOS_PASAN=SI
```
_(escribí SI si todos los tests pasan ahora)_

---

## Parte 7: Escribir un test propio

Ahora escribís un test completamente vos. El carrito tiene capacidad para `MAX_ITEMS = 4` productos.
Cuando está lleno, `carrito_agregar` debería devolver `0` (fallo) en vez de `1` (éxito).

Escribí una función `test_carrito_lleno()` en `tests/test_unitarios.c` (en el lugar `/* PARTE C */`) que:

1. Inicialice un carrito
2. Agregue 4 productos (usá el mismo `Producto` cuatro veces, está bien)
3. Intente agregar un 5to producto
4. Verifique con `ASSERT_IGUAL` que ese último intento devuelve `0`

Descomentá `/* test_carrito_lleno(); */` en el `main()`, compilá y corré.

**P8** — ¿El test pasa o falla? Si falla, ¿qué devuelve ese 5to `carrito_agregar`?

> R:El test falla, devuelve un 1 de exito mientras que lo esperado es un 0

Si el test falló, encontraste el segundo bug. Buscá en `src/carrito.c` la condición del `if` dentro de `carrito_agregar`.

**P9** — ¿Cuál es el operador incorrecto y cuál debería ser?

> R:El operador incorrecto es <= y deberia ser <

Corregí el bug, volvé a compilar y verificá que todos los tests pasan.

```
BUG_2_CORREGIDO=SI
```
_(SI o NO)_

---

## Parte 8: Test de integración guiado

Los tests de integración verifican que las funciones trabajan bien **en combinación**.

Abrí `tests/test_integracion.c`. El archivo ya tiene la estructura armada; vas a agregar una función.

Escribí `test_compra_con_descuento()` en el lugar `/* PARTE D */` siguiendo esta descripción:

- Inicializá un carrito
- Agregá Pan ($200, cantidad 3) → subtotal: $600
- Agregá Leche ($350, cantidad 2) → subtotal: $700
- Total esperado: **$1300**
- Con 10% de descuento: **$1170**
- Usá `ASSERT_IGUAL` para verificar tanto el total como el precio con descuento

Descomentá `/* test_compra_con_descuento(); */` en el `main()`, compilá y corré:

```bash
make test_integracion
./test_integracion
```

Guardá la salida:

```bash
./test_integracion > salidas/test_integracion.txt 2>&1
cat salidas/test_integracion.txt
```

```
TEST_INTEGRACION_PASA=SI
```
_(SI o NO)_

---

## Parte 9: Test de integración propio

Escribí `test_agregar_hasta_llenar()` en el lugar `/* PARTE E */`. Este test debe:

1. Llenar el carrito hasta su capacidad máxima (4 productos)
2. Verificar que el conteo es `MAX_ITEMS`
3. Verificar que intentar agregar uno más devuelve `0`
4. Verificar que el conteo sigue siendo `MAX_ITEMS` (no cambió)

Descomentá `/* test_agregar_hasta_llenar(); */` en el `main()`, compilá y corré.

```
TEST_LLENAR_PASA=SI
```
_(SI o NO)_

---

## Parte 10: Cobertura de código

¿Tus tests ejercitan todo el código que escribiste? `gcov` lo mide:

```bash
make cobertura
cat carrito.c.gcov
```

Las líneas con `#####` nunca se ejecutaron — no están cubiertas por los tests.

**P10** — ¿Hay alguna línea de `carrito.c` con `#####`? ¿Cuál y por qué no se ejecutó?

> R:Las lineas 29 y 30 donde se encuentra la funcion carrito_descuento. Esto pasa porque en los test unitarios no es usada la funcion carrito descuent

```
COBERTURA_COMPLETA=COBERTURA_COMPLETA=gcc -Wall -Wextra -std=c99 --coverage -c src/carrito.c -o carrito.o
gcc -Wall -Wextra -std=c99 --coverage -c tests/test_unitarios.c -o test_unitarios.o
tests/test_unitarios.c:57:1: warning: return type defaults to ‘int’ [-Wimplicit-int]
   57 | test_carrito_lleno() {
      | ^~~~~~~~~~~~~~~~~~
tests/test_unitarios.c: In function ‘test_carrito_lleno’:
tests/test_unitarios.c:67:1: warning: control reaches end of non-void function [-Wreturn-type]
   67 | }
      | ^
gcc --coverage carrito.o test_unitarios.o -o test_cobertura
./test_cobertura
=== Tests unitarios ===
[carrito nuevo]
  [OK]   carrito_contar(&c) == 0

[agregar un producto]
  [OK]   carrito_agregar(&c, p) == 1
  [OK]   carrito_contar(&c) == 1

[total: un producto, cantidad 1]
  [OK]   carrito_total(&c) == 350

[total: un producto, cantidad 2]
  [OK]   carrito_total(&c) == 700

[carrito lleno]
  [OK]   resultado == 0

--- 6/6 tests OK ---
gcov -o . carrito.o
File 'src/carrito.c'
Lines executed:88.89% of 18
Creating 'carrito.c.gcov'

Lines executed:88.89% of 18
        -:    0:Source:src/carrito.c
        -:    0:Graph:./carrito.gcno
        -:    0:Data:./carrito.gcda
        -:    0:Runs:1
        -:    1:#include "carrito.h"
        -:    2:#include <string.h>
        -:    3:
        5:    4:void carrito_init(Carrito *c) {
        5:    5:    c->cantidad = 0;
        5:    6:}
        -:    7:
        8:    8:int carrito_agregar(Carrito *c, Producto p) {
        8:    9:    if (c->cantidad < MAX_ITEMS) {   /* BUG: deberia ser < MAX_ITEMS */
        7:   10:        c->items[c->cantidad] = p;
        7:   11:        c->cantidad++;
        7:   12:        return 1;
        -:   13:    }
        1:   14:    return 0;
        -:   15:}
        -:   16:
        2:   17:int carrito_contar(Carrito *c) {
        2:   18:    return c->cantidad;
        -:   19:}
        -:   20:
        2:   21:int carrito_total(Carrito *c) {
        2:   22:    int total = 0;
        4:   23:    for (int i = 0; i < c->cantidad; i++) {
        2:   24:        total += c->items[i].cantidad * c->items[i].precio;  /* BUG: falta multiplicar por cantidad */
        -:   25:    }
        2:   26:    return total;
        -:   27:}
        -:   28:
    #####:   29:int carrito_descuento(int total, int porcentaje) {
    #####:   30:    return total - (total * porcentaje / 100);
        -:   31:}
```
_(SI si todas las líneas están cubiertas, NO si hay alguna con #####)_

---

## Preguntas de reflexión

**P11** — ¿Qué diferencia hay entre un test unitario y uno de integración? ¿Cuál de los dos detectó primero el bug de `carrito_total`?

> R:El test unitario evalua el resultados de funciones por separado, mientras que el de integracion utiliza todas las funciones y evalua con funciona en conjunto. El test unitario detecto primero el bug del carrito total.

**P12** — El bug de capacidad en `carrito_agregar` causa un **buffer overflow**: se escribe más allá del array. ¿Por qué esto es peligroso en C pero no ocurriría en un lenguaje como Python o Java?

> R:Esto es peligroso porque al escribir mas alla del array estamos modificando memoria a la cual no sabemos como acceder, es decir, no tenemos un puntero a esa momoria a diferencia de los espacios del array.

**P13** — En este laboratorio encontraste los bugs escribiendo tests. ¿Qué tiene de mejor este enfoque frente a mirar el código directamente?

> R:Mirar el código e intentar comprenderlo para encontrar el error puede ser abrumador y estresante, pero mediante estos tests ,si hay un error, nos indica donde se encuentra y el motivo por el cual se genero.

**P14** — El test `test_total_precio_unitario` (cantidad = 1) **pasó** a pesar del bug, mientras que `test_total_con_cantidad` (cantidad = 2) **falló**. ¿Por qué el primer test no detectó el bug?

> R:Porque el test unitario evaluava el valor, mientras que el test total evaluava el total del precio por la cantidad de ese producto. El bug era que faltaba multiplicar la cantidad por el precio.

```
BUG_EN_FUNCION_1=carrito_agregar
```
_(nombre de la función con el primer bug)_

```
BUG_EN_FUNCION_2=carrito_total
```
_(nombre de la función con el segundo bug)_

---

## Entrega

### Checklist

- [X] Ambos bugs corregidos en `src/carrito.c`
- [X] Tests en `tests/test_unitarios.c` y `tests/test_integracion.c` completos y pasando
- [X] Salidas guardadas en `salidas/`
- [X] Respuestas y claves completadas en este archivo
- [X] `make test` pasa localmente
- [X] Todo pusheado a `main`

### Verificación local

Antes de hacer push, verificá tu puntaje con:

```bash
make test
```

**Flujo recomendado:** hacé commits frecuentes mientras avanzás, usá `make test` para verificar tu progreso, y dejá el push para cuando una parte esté realmente lista.

Cuando estés listo/a:

```bash
git add .
git commit -m "Agrego tests y corrijo bugs en carrito"
git push
```

### Integración continua

Cuando pusheás cambios dentro de `labs/laboratorio-testing-c/`, el workflow central del monorepo ejecuta los mismos checks y valida el puntaje mínimo.

> ⚠️ **Evitá pushes innecesarios.** Cada ejecución consume cómputo en servidores de GitHub — un recurso compartido. `make test` te da el mismo resultado en tu terminal sin costo.

Para ver los resultados:

1. Entrá a tu repositorio en GitHub
2. Hacé click en la pestaña **Actions**
3. Hacé click en la ejecución más reciente → job **Grading · laboratorio-testing-c**
4. Al final del job vas a ver la tabla con el resultado de cada check y el puntaje total

---

## Ejercitación extra

**E1** — Agregá una función `int carrito_buscar(Carrito *c, char *nombre)` que devuelva el índice del producto con ese nombre, o `-1` si no existe. Escribí al menos 3 tests unitarios para ella.

**E2** — Modificá `carrito_total` para que ignore productos con `cantidad <= 0`. Agregá un test que lo verifique.

**E3** — Deliberadamente escribí un test que falle (ponés un valor incorrecto en `ASSERT_IGUAL`). Corré los tests y observá el mensaje de error. ¿Qué información da el framework? Luego revertí el cambio.

**E4** — Investigá qué es **TDD** (Test Driven Development). ¿En qué orden se escribe el código? ¿En qué se diferencia de lo que hiciste en este laboratorio?
