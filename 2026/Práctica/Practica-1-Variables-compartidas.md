**Programación Concurrente**

**Práctica 1 – Variables compartidas**

**1.** Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas:

a) En algún caso el valor de x al terminar el programa es 56.

b) En algún caso el valor de x al terminar el programa es 22.

c) En algún caso el valor de x al terminar el programa es 23.

```pseudocode
P1::
    If (x = 0) then
        y := 4*2;
        x := y + 2;

P2::
    If (x > 0) then
        x := x + 1;

P3::
    x := (x*3) + (x*2) + 1;
```

**Rta.:** primero, descomponer los procesos en acciones atómicas de grano fino, solo en aquellas partes en las que hay variables comprometidas por el uso de otros procesos.

```pseudocode
P1::
    If (x = 0) then
        y := 4*2;
        x := y + 2;
        1) Load Pos Memoria y, reg acumulador
        2) Add 2, reg acumulador
        3) Store reg acumulador, Pos Memoria x

P2::
    If (x > 0) then
        x := x + 1;
        4) Load Pos Memoria x, reg acumulador
        5) Add 1, reg acumulador
        6) Store reg acumulador, Pos Memoria x

P3::
    x := (x*3) + (x*2) + 1;
        7) Load Pos Memoria x, reg1
        8) Load Pos Memoria x, reg2
        9) Multi 3, reg1
        10) Multi 2, reg2
        11) Add reg2, reg1
        12) Add 1, reg1
        13) Store reg1, Pos Memoria x
```

en segundo lugar, identificar los diferentes resultados:

```pseudocode
1 - 2 - 3 - 4 - 5 - 6 - 7 - 8 - 9 - 10 - 11 - 12 - 13  →  x = 56
4 - 5 - 6 - 7 - 8 - 9 - 10 - 11 - 12 - 13 - 1 - 2 - 3  →  x = 1
1 - 2 - 7 - 8 - 9 - 10 - 11 - 12 - 13 - 3 - 4 - 5 - 6  →  x = 11
7 - 1 - 2 - 3 - 8 - 9 - 10 - 11 - 12 - 13 - 4 - 5 - 6  →  x = 22
7 - 8 - 9 - 10 - 11 - 12 - 13 - 1 - 2 - 3 - 4 - 5 - 6  →  x = 2
7 - 1 - 2 - 3 - 4 - 5 - 6 - 8 - 9 - 10 - 11 - 12 - 13  →  x = 23
```

∴ las sentencias a, b y c son verdaderas.

**2.** Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.

**Rta.:** precondiciones: se dispone de un arreglo de tamaño M. hay K procesos ContarN. M % K == 0.

```pseudocode
int total = 0;
int[] arreglo = arreglo[M];
int porciones = M / K;

Process ContarN[id: 0..K-1] {
    int base = id * porciones;
    int contador = 0;
    for i = base .. (base + porciones - 1)
        if (arreglo[i] == N) contador++;
    < total += contador >
}
```

**3.** Dada la siguiente solución de grano grueso:

a) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias.

```pseudocode
int cant = 0;
int pri_ocupada = 0;
int pri_vacia = 0;
int buffer[N];

Process Productor:: {
    while (true) {
        produce elemento
        <await (cant < N); cant++>
        buffer[pri_vacia] = elemento;
        pri_vacia = (pri_vacia + 1) mod N;
    }
}

Process Consumidor:: {
    while (true) {
        <await (cant > 0); cant-->
        elemento = buffer[pri_ocupada];
        pri_ocupada = (pri_ocupada + 1) mod N;
        consume elemento
    }
}
```

**Rta.:** dado el caso en el que Consumidor sea un proceso más veloz que Productor, existe una alta probabilidad de que Consumidor intente acceder a un elemento del buffer antes de que Productor lo cargue. se soluciona encerrando en exclusión mutua hasta la carga (en Productor) y el acceso (en Consumidor) del elemento.

```pseudocode
int cant = 0;
int pri_ocupada = 0;
int pri_vacia = 0;
int buffer[N];

Process Productor:: {
    while (true) {
        produce elemento
        < await (cant < N); cant++
          buffer[pri_vacia] = elemento; >
        pri_vacia = (pri_vacia + 1) mod N;
    }
}

Process Consumidor:: {
    while (true) {
        < await (cant > 0); cant--
          elemento = buffer[pri_ocupada]; >
        pri_ocupada = (pri_ocupada + 1) mod N;
        consume elemento
    }
}
```

b) Modificar el código para que funcione para C consumidores y P productores.

**Rta.:**

```pseudocode
int cant = 0;
int pri_ocupada = 0;
int pri_vacia = 0;
int buffer[N];

Process Productor[id: 0..P-1] {
    while (true) {
        produce elemento
        < await (cant < N); cant++
          buffer[pri_vacia] = elemento;
          pri_vacia = (pri_vacia + 1) mod N; >
    }
}

Process Consumidor[id: 0..C-1] {
    while (true) {
        < await (cant > 0); cant--
          elemento = buffer[pri_ocupada];
          pri_ocupada = (pri_ocupada + 1) mod N; >
        consume elemento
    }
}
```

**4.** Resolver con SENTENCIAS AWAIT (<> y <await B; S>). Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.

**Rta.:**

```pseudocode
colaRecursos cola;

Process UsarRecurso[id: 0..N-1] {
    Recurso r;
    < await !cola.empty(); r = cola.dequeue(); >
    r.use();
    < cola.enqueue(r); >
}
```

**5.** En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.

a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).

d) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).

e) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

**Rta.:**

a)
```pseudocode
Process Persona[id: 0..N-1] {
    < Imprimir(documento); >
}
```

b)
```pseudocode
int numero = 0;
int siguiente = 0;

Process Persona[id: 0..N-1] {
    < int miNumero = numero;
      numero++; >
    < await siguiente == miNumero; >
    Imprimir(documento);
    < siguiente++; >
}
```

c)
```pseudocode
int actual = -1;
ColaOrdenada cola;

Process Persona[id: 0..N-1] {
    < if (actual == -1)
        siguiente = id;
      else
        cola.enqueue(id); >
    < await siguiente == id; >
    Imprimir(documento);
    < if (cola.isEmpty())
        siguiente = -1;
      else
        siguiente = cola.dequeue(); >
}
```

d)
```pseudocode
int siguiente = 0;

Process Persona[id: 0..N-1] {
    < await siguiente == id; >
    Imprimir(documento);
    siguiente++;
}
```

e)
```pseudocode
int siguiente = -1;
boolean termine = false;

Process Coordinador {
    while (true) {
        < await !cola.isEmpty();
          siguiente = cola.dequeue(); >
        < await termine; termine = false; >
    }
}

Process Persona[id: 0..N-1] {
    cola.enqueue(id);
    < await siguiente == id; >
    Imprimir(documento);
    termine = true;
}
```

**6.** Realice una solución concurrente de grano grueso (utilizando <> y/o ) para el siguiente problema. En un examen final hay P alumnos y 3 profesores. Cuando todos los alumnos han llegado comienza el examen. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respectando el orden en que los alumnos van entregando.

**Rta.:**

```pseudocode
int presentes = 0;
int total = P;
ColaEspecial cola;
double[P] notas = 0;

Process Alumno[id: 0..P-1] {
    < presentes++; >
    < await presentes == p; >
    // suponiendo que los profesores ya están
    // resolver examen
    < cola.entregarExamen(id); >
    < await notas[id] != 0; >
}

Process Profesor[id: 0..2] {
    int alumno;
    while (true) {
        < await !cola.isEmpty(); Examen e = cola.tomarExamen(alumno); >
        // correjir examen e
        notas[alumno] = e.nota();
    }
}
```

**7.** Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:

```pseudocode
int turno = 1;

Process SC1:: {
    while (true) {
        while (turno == 2) skip;
        SC;
        turno = 2;
        SNC;
    }
}

Process SC2:: {
    while (true) {
        while (turno == 1) skip;
        SC;
        turno = 1;
        SNC;
    }
}
```

**Rta.:**

- exclusión mutua: sí. la variable turno funciona como un semáforo: cuando vale 1, solo SC1 estará en SC, y cuando vale 2, solo SC2 estará en SC.
- livelock: sí. como la variable turno solo vale 1 o 2, siempre habrá uno de dos procesos ejecutando algo.
- ausencia de demora innecesaria: no. por ejemplo, si SC2 pone turno en 1 y hace rápida su SNC pero la SNC de SC1 tarda demasiado, SC2 se quedará esperando en el while que SC1 termine su SNC.
- eventual entrada: sí. solo porque tanto SC como SNC son segmentos de código finitos (como lo dice en el enunciado).

**8.** Desarrolle una solución de grano fino usando sólo variables compartidas (no se puede usar las sentencias await ni funciones especiales como TS o FA). En base a lo visto en la clase 2 de teoría, resuelva el problema de acceso a sección crítica usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le dé permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. **Nota**: puede basarse en la solución para implementar barreras con Flags y Coordinador vista en la teoría 2.

**Rta.:**

```pseudocode
int pedido[1:N] = ([N]0);
int permiso[1:N] = ([N]0);
int salida[1:N] = ([N]0);

Process SC[i: 1..N] {
    while (true) {
        pedido[i] = 1;
        while (permiso[i] == 0) skip;
        permiso[i] = 0;
        salida[i] = 1;
    }
}

Process Coordinador {
    while (true) {
        for (i := 1 to N) {
            if (pedido[i] == 1) {
                pedido[i] = 0;
                permiso[i] = 1;
                while (salida[i] == 0) skip;
                salida[i] = 0;
            }
        }
    }
}
```
