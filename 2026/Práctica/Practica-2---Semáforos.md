**Programación Concurrente ATIC**

**Práctica 2 – Semáforos**

**CONSIDERACIONES PARA RESOLVER LOS EJERCICIOS:**
- Recuerde que en la práctica sólo se utilizan los semáforos GENERALES.
- Los semáforos deben estar declarados en todos los ejercicios.
- Los semáforos deben estar inicializados en todos los ejercicios.
- No se puede utilizar ninguna sentencia para setear o ver el valor de un semáforo.
- Debe evitarse hacer busy waiting en todos los ejercicios.
- En todos los ejercicios el tiempo debe representarse con la función delay.

**1. Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola. Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente.**

```pseudocode
Cola c;
sem mutex = 1;
sem instancias = 5;

Process Proceso[id: 1..P-1] {
    int i;
    while (true) {
        p(instancias);

        p(mutex);
        i = c.pop();
        v(mutex);

        // usando instancia

        p(mutex);
        c.push(i);
        v(mutex);

        v(instancias);
    }
}
```

**2. Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión.**

1. **Analice el problema y defina qué procesos, recursos y semáforos serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema.**
2. **Implemente una solución que modele el acceso de las personas a un detector (es decir, si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar).**
3. **Modifique su solución para el caso que haya tres detectores.**

2.
```pseudocode
Detector d;
sem mutex = 1;

Process Persona[id: 1..N-1] {
    p(mutex);
    d.pasar();
    v(mutex);
}
```

3.
```pseudocode
sem detectores = 3;
sem mutex = 1;

Process Persona[id: 1..N-1] {
    p(detectores);
    // usar detector
    v(detectores);
}
```

**3. Un sistema de control cuenta con 4 procesos que realizan chequeos en forma colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de tamaño N). De cada fallo, se conoce su número de identificación (ID) y su nivel de gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Para cada item realice una solución adecuada a lo pedido:**

**a. Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden).**

**b. Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global.**

**c. Ídem b) pero cada proceso debe ocuparse de contar los fallos de un nivel de gravedad determinado.**

**a.**
```pseudocode
Fallo historial[N];
int cantPorProceso = N DIV 4;

Process Proceso[id: 0..3] {
    int ini = id * cantPorProceso;
    int fin = ini + cantPorProceso - 1;
    for i: ini..fin
        if (historial[i].gravedad == 3)
            write(historial[i].id);
}
```

**b.**
```pseudocode
Fallo historial[N];
int cantPorProceso = N DIV 4;
int vecGlobal ([4] 0);
sem mutex = 1;

Process Proceso[id: 0..3] {
    int vectorGravedades ([4] 0);
    int ini = id * cantPorProceso;
    int fin = ini + cantPorProceso - 1;
    for i: ini..fin {
        vectorGravedades[historial[i].gravedad]++;
    }
    p(mutex);
    for i: 0..3 {
        vecGlobal[i] += vectorGravedades[i];
    }
    v(mutex);
}
```

**c.**
```pseudocode
Fallo historial[N];
int cantPorProceso = N DIV 4;
int vecGlobal[4];

Process Proceso[id: 0..3] {
    int ini = id * cantPorProceso;
    int fin = ini + cantPorProceso - 1;
    int cantFallos = 0;
    for i: ini..fin {
        if (historial[i].gravedad == id) cantFallos++;
    }
    vecGlobal[id] = cantFallos;
}
```

**4. En una empresa de logística de paquetes existe una sala de contenedores donde se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:**

**a. La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de preparar los paquetes y dejarlos en los contenedores; un empelado Entregador que se ocupa de tomar los paquetes de los contenedores y realizar la entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez.**

**b. Modifique la solución a) para el caso en que haya *P* Preparadores y *E* Entregadores.**

**a.**
```pseudocode
Paquete contenedores[N];
int indiceLibre = 0;
int indiceOcupado = 0;
sem vacios = N;
sem llenos = 0;

Process Preparador {
    Paquete p;
    while (true) {
        p = new Paquete();
        P(vacios);
        contenedores[indiceLibre] = p;
        indiceLibre = (indiceLibre + 1) % N;
        V(llenos);
    }
}

Process Entregador {
    Paquete p;
    while (true) {
        P(llenos);
        p = contenedores[indiceOcupado];
        indiceOcupado = (indiceOcupado + 1) % N;
        V(vacios);
        p.entregar();
    }
}
```

**b.**
```pseudocode
Paquete contenedores[N];
sem mutexP = 1;
sem mutexE = 1;
sem vacios = N;
sem ocupados = 0;
int indiceLibre, indiceOcupado = 0;

Process Preparador[id: 0..P] {
    Paquete p;
    while (true) {
        P(vacios);
        P(mutexP);
        contenedores[indiceLibre] = p;
        indiceLibre = (indiceLibre + 1) % N;
        V(mutexP);
        V(llenos);
    }
}

Process Entregador[id: 0..E] {
    Paquete p;
    while (true) {
        P(llenos);
        P(mutexE);
        p = contenedores[indiceOcupado];
        indiceOcupado = (indiceOcupado + 1) % N;
        V(mutexE);
        V(llenos);
        p.entregar();
    }
}
```

**5. Suponga que se tiene un curso con 50 alumnos. Cada alumno debe realizar una tarea y existen 10 enunciados posibles. Una vez que todos los alumnos eligieron su tarea, comienzan a realizarla. Cada vez que un alumno termina su tarea, le avisa al profesor y se queda esperando el puntaje del grupo, el cual está dado por todos aquellos que comparten el mismo enunciado. Cuando todos los alumnos que tenían la misma tarea terminaron, el profesor les otorga un puntaje que representa el orden en que se terminó esa.**

**Nota: para elegir la tarea suponga que existe una función elegir que le asigna una tarea a un alumno (esta función asignará 10 tareas diferentes entre 50 alumnos, es decir, que 5 alumnos tendrán la tarea 1, otros 5 la tarea 2 y así sucesivamente para las 10 tareas).**

```pseudocode
int cantElegidos = 0;
sem elegir = 1;
sem espera_profesor = 0;
sem barrera = 0;
sem sCola = 1;
sem espera_tarea = 0;
Cola c;
double puntajes ([10] 0);
sem espera_puntaje[10] = 0;

Process Alumno[id: 1..50] {
    int i;
    double mi_nota;
    Tarea tarea = elegir();
    P(elegir);
    cantElegidos++;
    if (cantElegidos == 50) {
        for i = 1..50: V(barrera);
        V(espera_profesor);
    }
    V(elegir);
    P(barrera);
    // hacer tarea
    P(sCola);
    push(c, tarea);
    V(espera_tarea);
    V(sCola);
    P(espera_puntaje[tarea.grupo]);
    mi_nota = puntajes[tarea.grupo];
}

Process Profesor {
    int i, j;
    int grupos_entregados ([10] 0);
    int siguiente = 10;
    P(espera_profesor);
    for i = 1..50 {
        P(espera_tarea);
        P(sCola);
        pop(c, tarea);
        V(sCola);
        if (++grupos_entregados[tarea.grupo] == 5) {
            puntajes[tarea.grupo] = siguiente;
            siguiente--;
            for j: 1..5: V(espera_puntaje[tarea.grupo]);
        }
    }
}
```

6. Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción:
   - no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD.
   - no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD.

Indique si la solución presentada es la más adecuada. Justifique la respuesta.

```pseudocode
Var
    sem: semaphoro := 6;
    alta: semaphoro := 4;
    baja: semaphoro := 5;

Process Usuario-Alta[I: 1..L]:: {
    P(sem);
    P(alta);
    // usa la BD
    V(sem);
    V(alta);
}

Process Usuario-Baja[I: 1..K]:: {
    P(sem);
    P(baja);
    // usa la BD
    V(sem);
    V(baja);
}
```

7. Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos:

a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función *Imprimir(documento)* llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las *Personas*.

b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

c) Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1).

d) Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.

e) Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar.

8. Una fábrica de piezas metálicas debe producir T piezas por día. Para eso, cuenta con E empleados que se ocupan de producir las piezas de a una por vez. La fábrica empieza a producir una vez que todos los empleados llegaron. Mientras haya piezas por fabricar, los empleados tomarán una y la realizarán. Cada empleado puede tardar distinto tiempo en fabricar una pieza. Al finalizar el día, se debe conocer cuál es el empleado que más piezas fabricó.

a. Implemente una solución asumiendo que T > E.

b. Implemente una solución que contemple cualquier valor de T y E (incluso T<E).

9. Resolver el funcionamiento en una fábrica de ventanas con 7 empleados (4 carpinteros, 1 vidriero y 2 armadores) que trabajan de la siguiente manera:

- Los carpinteros continuamente hacen marcos (cada marco es armando por un único carpintero) y los deja en un depósito con capacidad de almacenar 30 marcos.
- El vidriero continuamente hace vidrios y los deja en otro depósito con capacidad para 50 vidrios.
- Los armadores continuamente toman un marco y un vidrio (en ese orden) de los depósitos correspondientes y arman la ventana (cada ventana es armada por un único armador).

10. A una cerealera van T camiones a descargarse trigo y M camiones a descargar maíz. Sólo hay lugar para que 7 camiones a la vez descarguen, pero no pueden ser más de 5 del mismo tipo de cereal. **Nota:** no usar un proceso extra que actué como coordinador, resolverlo entre los camiones.

11. En un vacunatorio hay **un empleado** de salud para vacunar a **50 personas**. El empleado de salud atiende a las personas de acuerdo con el orden de llegada y de a 5 personas a la vez. Es decir, que cuando está libre debe esperar a que haya al menos 5 personas esperando, luego vacuna a las 5 primeras personas, y al terminar las deja ir para esperar por otras 5. Cuando ha atendido a las 50 personas el empleado de salud se retira. **Nota:** todos los procesos deben terminar su ejecución; asegurarse de no realizar *Busy Waiting*; suponga que el empleado tienen una función *VacunarPersona()* que simula que el empleado está vacunando a UNA persona.

12. Simular la atención en una Terminal de Micros que posee 3 puestos para hisopar a **150 pasajeros**. En cada puesto hay **una Enfermera** que atiende a los pasajeros de acuerdo con el orden de llegada al mismo. Cuando llega un pasajero se dirige al puesto que tenga menos gente esperando. Espera a que la enfermera correspondiente lo llame para hisoparlo. Cuando terminaron de hisoparlo se retira.

a. Implemente una solución considerando que además de los pasajeros y las enfermeras hay un **Coordinador** que es quien le indica al pasajero cual es el puesto que tiene menos gente esperando (el puesto al cual debe ir).

b. Implemente una solución en las que sólo existan los pasajeros y las enfermeras, siendo los pasajeros quienes determinan qué puesto tiene menos personas esperando. **Nota:** suponga que existe una función *Hisopar()* que simula la atención del pasajero por parte de la enfermera correspondiente.
