# Cálculo de tamaño de muestra en R con la librería `pwr`

*Notas de clase de Bioestadística (Dr. Antonio Quispe) explicadas a mi manera.*

## ¿Por qué nos importa esto?

En clase vimos que el tamaño de muestra, de manera general tiene estos conceptos clave:

- Toda investigación parte de una **hipótesis nula (H0)** que asumimos verdadera "por defecto" (ej. la moneda es justa, el tratamiento nuevo no cambia nada) y una **hipótesis alterna (Ha)** que es la que en realidad queremos demostrar.
- Nunca "aceptamos" H0: solo podemos **rechazarla** o **no rechazarla**. Si el valor p no es significativo no significa que H0 sea verdadera, puede que simplemente nuestro estudio no tuviera **poder** suficiente para detectar el efecto.
- Ahí es donde entra el tamaño de muestra: si mi `n` es muy chico, aunque exista una diferencia real, mi estudio no va a tener el poder para verla. Si mi tamaño de muestra es de gran magnitud, se puede llegar a encontrar "diferencias estadísticamente significativas" que en la práctica no importan pero debido a la gran información recopilada se llega a tener esa presición de poder dar tendencias mínimas.
-La relación entre variables que se recalcó durante las clases (sobre todo en Semana 3 es):
  - **Poder de estudio** sube si sube el **tamaño de muestra**.
  - **Poder** baja si también disminuye la **variabilidad** de los datos.
  - El tamaño de muestra necesario baja cuando la **variabilidad** es baja (datos más "limpios", más fácil encontrar diferencias).

En R, el paquete que automatiza estas cuentas es `pwr` (y `pwr2`que descubrí que ahora sus funciones están integradas en pwr). Todo el cálculo siempre necesita un grupo de 4 variables que se relacionan entre sí y si conoces 3, la función indicada de `pwr` te calcula la cuarta:

| Pieza | Qué es | Valor típico que usamos |
|---|---|---|
| `n` | tamaño de muestra | lo que queremos hallar |
| `d` / `h` / `f` | tamaño del efecto (qué tan grande es la diferencia que queremos detectar, "estandarizada") | depende del escenario |
| `sig.level` | es cuan confiable llega a ser nuestro estudio, ($\alpha$) es la probabilidad de para un falso positivo | 0.05 o 5% según concenso|
| `power` | poder del estudio, 1 − $\beta$ es como lo llaman en la teoría pero significa en sí que tan confiado estás que no es un falso negativo | 0.80 |

Con eso ya podemos entender el patrón: **casi todas las funciones de `pwr` reciben 3 de estos 4 argumentos y calculan el que falta.**. Si consultas en la sección de RStudio para ver la documentación ahí mismo explican en las diferentes funciones que debes dejar una variable libre para que el programa te lo calcule

## Instalación y carga

```r
install.packages("pwr")

library(pwr)   # cálculos básicos de poder usando tamaños de efecto
```

`pwr` trabaja con lo que le dicen **tamaños de efecto**: simplifica la diferencia (una diferencia de medias o de proporciones) en una fórmula para que la fórmula matemática sea comparable entre estudios distintos. Por eso casi nunca metemos directamente "la diferencia en soles" o "la diferencia en mmHg": primero la convertimos a `d`, `h` o `f`. Esos nombres se usan para que directamente usemos la nomenclatura que tiene el código de `prw`

 Los 8 escenarios que vimos en clase es la que repetimos varias veces variando un poco en qué versión de estudio se usa:

1. ¿El estudio es para comparar **una media** o **una proporción**?
2. ¿El estudio usa **una sola muestra contra un valor fijo**, **dos muestras independientes**, **dos muestras pareadas** (mismo sujeto en dos momentos) o **más de dos grupos**?
El orden va así según lo que quieres comparar:
Una muestra: Media (caso 1) o proporción (caso 2)
Dos muestras independientes: Media (caso 1), proporción (caso 2). tamaño de muestra es POR grupo, dependiendo de cuantos grupos tienes multiplicas el n por la cantidad de grupos.
Dos muestras pareadas:

Cabe recalcar que las opciones two.sided, grater, less. Es dependiendo de si tu estudio quiere comparar si la nueva propuesta es superior, inferior o al menos no es inferior que un resultado de un estudio anterior.

Con esas dos preguntas puedes hallar fácil la función de `pwr` para usar. Pero tienes que tener cuidado porque el programa te devolvera el tamaño n de UNA muestra, más el estudio usa varias muestras o grupos dependiendo del caso, por lo que tendrías que multiplicar según la cantidad de grupos que tengas.

## Escenario 1: una sola media, contra un valor de referencia

Ejemplo de clase: quiero saber si el promedio de algo cambió respecto a un valor conocido `m0 = 20`, esperando que el nuevo promedio sea `30`, con una desviación estándar de `10`.

Primero calculamos el tamaño de efecto `d` (la diferencia entre los promedios, dividida entre la desviación estándar), esto se hace para brindar a la función esta forma estándar de tamaño de efecto, la mejor definición para entender el tamaño de efecto es que ve cuántas desviaciones estandar hay entre el resultado conocido en la literatura (que tomaste como hipótesis nula) y el resultado esperado ahora:

```r
d <- (30 - 20) / 10
d
```

Con ese `d`, le pedimos a `pwr` el tamaño de muestra `n` mínimo para detectarlo con 80% de poder y con 95% de probabilidad de que el estudio esté en lo correcto:

```r
pwr.t.test(d = d, sig.level = 0.05, power = 0.80, type = "one.sample")
```

Y como mencioné arriba, la misma función sirve para "despejar" cualquiera de las otras piezas. Si ya tengo un `n` fijo (por ejemplo, porque así lo permite mi presupuesto) y quiero saber qué poder me da simplemente coloco 3 parámetros y dejo el de `power` sin aparecer:

```r
pwr.t.test(n = 20, d = d, sig.level = 0.05, type = "one.sample")
```

O, al revés, si tengo `n` y `power` fijos y quiero saber qué tan grande tendría que ser el efecto para que mi estudio funcione:

```r
pwr.t.test(n = 20, sig.level = 0.05, power = .80, type = "one.sample")
```

Es la misma fórmula, solo cambia cuál de los 4 argumentos dejamos vacío para que R lo calcule.

## Escenario 2: una sola proporción con una sola muestra

Es exactamente igual al Escenario 1, pero cuando lo que medimos es una **proporción/prevalencia** en vez de un promedio (ej. "¿la prevalencia real es 15% o sigue siendo 10%?").

Aquí el tamaño de efecto se llama `h`, y se calcula con la función `ES.h()` (en la definición técnica dentro de RStudio dice la documentacio´n que Cohen definió esta transformación con arcosenos para que las proporciones se comporten bien matemáticamente):

```r
h <- ES.h(0.15, 0.10)
h
```

Como referencia rápida que dieron en el documento plantilla: `h ≈ 0.2` es un efecto pequeño, `0.5` mediano, `0.8` grande.

```r
# n necesario
pwr.p.test(h = h, power = 0.80, sig.level = 0.05)

# poder si ya tengo n = 400
pwr.p.test(h = h, n = 400, sig.level = 0.05)

# efecto detectable si tengo n = 400 y quiero 80% de poder
pwr.p.test(n = 400, sig.level = 0.05, power = .80)
```

## Escenario 3: comparar medias de dos poblaciones independientes (multiplicar n por 2)

Este es el caso típico de "grupo A vs grupo B" (por ejemplo, expuestos vs no expuestos), donde las dos muestras **no tienen ninguna relación entre sí** (no están pareadas). En clase lo resumimos con el ejemplo de comparar Lima vs Iquitos, o Perú vs China: bolsas de datos completamente independientes. En este caso se usa la prueba t de Student.

En un estudio generalmente no tienes el efecto que se va a generar por ello defrente en la función hecha en base teórica de Cohen(1982), `cohen.ES()` nos da valores de referencia ya tabulados (pequeño/mediano/grande) para el tipo de prueba que indiquemos:

```r
efecto_u2_u1 <- cohen.ES(test = "t", size = "medium")
efecto_u2_u1
```

Y para `n`, poder o efecto:

```r
pwr.t.test(d = .5, sig.level = 0.05, power = 0.80,
           type = "two.sample", alternative = "two.sided")

pwr.t.test(d = .5, n = 70, sig.level = 0.05,
           type = "two.sample", alternative = "two.sided")

pwr.t.test(n = 70, sig.level = 0.05, power = 0.80,
           type = "two.sample", alternative = "two.sided")
```

`alternative = "two.sided"` es porque nuestra Ha es "son diferentes" (2 colas), no "uno es mayor que el otro" (1 cola). Esto también quedó claro en clase: si tu hipótesis es de superioridad o inferioridad, usas 1 cola; si es de "hay diferencia" en cualquier sentido, usas 2 colas.

## Escenario 4: comparar dos proporciones independientes (también multiplicar n por 2)

Igual que el Escenario 3, pero con proporciones. Ejemplo: `p1 = 10%` vs `p2 = 50%`.

```r
efecto_p2_p1 <- ES.h(0.5, 0.1)
efecto_p2_p1

pwr.2p.test(h = efecto_p2_p1, sig.level = 0.05, power = 0.80,
            alternative = "two.sided")

pwr.2p.test(h = efecto_p2_p1, n = 20, sig.level = 0.05,
            alternative = "two.sided")

pwr.2p.test(n = 20, sig.level = 0.05, power = 0.80,
            alternative = "two.sided")
```

## Escenario 5: comparar medias en muestras pareadas (estudios longitudinales)

Aquí cambia la naturaleza del diseño: ya no son dos grupos independientes, sino **el mismo sujeto medido dos veces** (basal vs seguimiento). Como vimos en clase, en un estudio longitudinal se compara la "diferencia de las diferencias" por sujeto, por eso el diseño se llama `paired`.
esto básicamente dice que solo necesitas un n. Igual para el caso 6 solo que aplicado a encontrar proporciones
```r
efecto_u2p_u1p <- (0 - 5) / 5
efecto_u2p_u1p

pwr.t.test(d = 1, power = 0.8, sig.level = 0.05,
           type = "paired", alternative = "two.sided")

pwr.t.test(d = 1, n = 40, sig.level = 0.05,
           type = "paired", alternative = "two.sided")

pwr.t.test(n = 40, power = 0.8, sig.level = 0.05,
           type = "paired", alternative = "two.sided")
```

## Escenario 6: comparar más de dos medias (ANOVA de una vía)

Cuando ya no son 2 grupos sino 3 o más (ej. dengue leve / dengue severo / sin dengue), la H0 pasa a ser "todas las medias son iguales" y la Ha es "al menos una es diferente". Aquí el tamaño de efecto se llama `f`:

```r
efecto_anova <- cohen.ES(test = "anov", size = "medium")
efecto_anova

pwr.anova.test(f = 0.25, k = 3, sig.level = 0.05, power = 0.80)

pwr.anova.test(f = 0.25, k = 3, n = 60, sig.level = 0.05)

pwr.anova.test(k = 3, n = 60, sig.level = 0.05, power = 0.80)
```

`k` es el número de grupos que estamos comparando.

## Escenario 7: comparar medias con `power.anova.test` (usando valores reales de los grupos). Acá tienes más de un grupo, incluso más de 2 porque ya se habla de un estudio bastante grande y que comparara muchos grupos. Por lo que n se multiplica según el número de grupos que se quiera evaluar. Casi nunca usas este tipo de cálculo de muestra porque es muy dificil que se de la oportunidad de hacer un estudio tan ambicioso.

Esta variante usa la función base de R `power.anova.test()` en vez de `pwr.anova.test()`, útil cuando ya tenemos las medias reales de cada grupo y podemos calcular directamente su varianza:

```r
grupos <- c(550, 598, 610)

p <- power.anova.test(groups = length(grupos),
                       between.var = var(grupos),
                       within.var = 6400,
                       power = 0.8,
                       sig.level = 0.05,
                       n = NULL)
p
```

Esto nos dice, por ejemplo, que se necesitan **32 sujetos por grupo** (96 en total con 3 grupos) para un poder de 0.8.

También podemos fijar `n` y ver qué poder obtenemos:

```r
n <- 40
p_anova <- power.anova.test(groups = 3,
                             between.var = var(grupos),
                             within.var = 6400,
                             power = NULL,
                             sig.level = 0.05,
                             n = n)
p_anova
```

Y hasta graficar cómo cambia el poder según el `n`:

```r
plot(n, p_anova$power)
```

El tamaño del efecto en este caso es simplemente la diferencia entre el grupo más alto y el más bajo, dividida entre la desviación estándar común:

```r
(610 - 550) / 80
```

## Escenario 8: comparar proporciones con más de dos grupos (diseño factorial / ANOVA de dos vías)

Este es el caso de un ensayo con **varios factores a la vez** (por ejemplo, en clase: nutricionista sí/no cruzado con app móvil sí/no → 4 brazos). Aquí usamos `pwr2`, porque ya no es un solo factor sino dos (`A` y `B`), cada uno con su propio tamaño de efecto.

```r
pwr.2way(a = 2, b = 2, alpha = 0.05, size.A = 40, size.B = 40,
         f.A = 0.25, f.B = 0.55)
```

Esto da el poder para cada efecto principal por separado (en este ejemplo: ~88% para el efecto A y prácticamente 100% para el efecto B, que es un efecto grande).

Y para hallar el `n` necesario por grupo dado el poder que queremos (`beta = 0.20` equivale a poder = 0.80):

```r
ss.2way(a = 2, b = 2, alpha = 0.05, beta = 0.20, f.A = 0.25, f.B = 0.55,
        delta.A = NULL, delta.B = NULL, sigma.A = NULL, sigma.B = NULL, B = 40)
```

## En resumen: cómo elegir la función correcta

| ¿Qué mido? | ¿Cuántos grupos / momentos? | Función de `pwr` |
|---|---|---|
| Media | 1 grupo vs valor fijo | `pwr.t.test(..., type="one.sample")` |
| Proporción | 1 grupo vs valor fijo | `pwr.p.test()` |
| Media | 2 grupos independientes | `pwr.t.test(..., type="two.sample")` |
| Proporción | 2 grupos independientes | `pwr.2p.test()` |
| Media | 2 momentos, mismo sujeto (pareado) | `pwr.t.test(..., type="paired")` |
| Media | 3+ grupos independientes | `pwr.anova.test()` / `power.anova.test()` |
| Proporción | 3+ grupos, diseño factorial (2 factores) | `pwr.2way()` / `ss.2way()` |

Y en cualquier función, el truco es siempre el mismo: **le das 3 de los 4 ingredientes (n, tamaño de efecto, α, poder) y R te calcula el que te falta.**

---

*El código y los apuntes de clase vienen a partir de lo indicado por el Dr. Antonio Quispe, con el código base del archivo `Muestra.qmd` como plantilla. Este post fue hecho primeramente en un word y luego consultando a Claude cómo mejorar el post y cómo llegar a escribirla para un post de Github, cosa que aprendí mientras elaboraba la versión final (de word a Github) del post*
