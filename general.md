## Casos generales

- N clientes que acceden a N recursos
    * con distinta prioridad
    * ordenado
    * no ordenado
    * asignando recurso con una condicion extra
    * el recurso hace algo si nadie lo solicita

- N clientes que acceden a 1 recurso 
    * con distinta prioridad
    * ordenado
    * no ordenado
    * asignando recurso con una condicion extra

## Consideraciones

- en ADA y PMA los mensajes se ENCOLAN
- en PMS NO se encolan

## PMA

N clientes N recursos - con distinta prioridad
N clientes N recursos - ordenado (implicito)
N clientes N recursos - no ordenado

en este caso se le manda solicitud() porque el recurso no tiene que hacer nada hasta que reciba la solicitud, sino habria que agregar un ELSE al if 
para pasarle 'VACIO' al recurso

```c
chan recursoListo(Integer); 

chan solicitud(Integer);
chan solicitudCliente(Integer);
chan solicitudPrioritaria(Integer);

chan asignacionRecurso[N](Integer);

chan peticion[P](Any);
chan respuesta[N](Any);

Process cliente[id = 0 .. N]{
    Integer idRecurso;
    Boolean prioridad = getPrioridad();

    send solicitud();
    if (prioridad) {
        send solicitudCliente(id);
    } else {
        send solicitudPrioritaria(id);
    }

    receive asignacionRecurso(idRecurso);

    send peticion[idRecurso]()
    receive respuesta[id]()
}

Process coordinador{
    Integer idRecursoLibre, idCliente;

    while (true){
        receive recursoListo(idRecursoLibre);
        receive solicitud();

        if (not empty(solicitudPrioritaria)) => 
            receive solicitudPrioritaria(idCliente);
        * (empty(solicitudPrioritaria) and not empty(solicitudCliente)) =>
            receive solicitudCliente(idCliente);
        fi;

        asignacionRecurso[idCliente](idRecursoLibre);
    }
}

Process recurso[id = 0 .. P]{
    Integer idCliente;

    while (true){
        send recursoLibre(id);
        receive peticion[id](idCliente);
        // HACE ALGO
        send respuestaCliente[idCliente]();
    }
}
```
N clientes N recursos - asignando recursos con una condicion extra

supongamos que se asigna recurso en base a la cantidad de clientes que estan esperando para usar el recurso

```c

chan liberacion(Integer);

chan solicitudRecurso(Integer);
chan asignacionRecurso[N](Integer);

chan peticion[P](any);
chan respuesta[N](any);


Process cliente[id = 0 .. N]{
    Integer idRecurso;

    send solicitudRecurso(id);
    receive asignacionRecurso(idRecurso);

    send peticion[idRecurso](id);
    receive respeusta[id]();
}

Process coordinador{
    Integer cantEsperando[P] = ([P] 0);
    Integer idRecurso, idCliente;
    Integer idRecursoMin, min;

    while (true){
        if (empty(liberacion) and not empty(solicitudRecurso)){
            receive solicitudRecurso(idCliente);
            
            min:= 99999;
            for i in (0..P - 1){
                if (cantEsperando[i] < min){
                    min:= cantEsperando[i];
                    idRecursoMin:= i;
                }
            }
            
            cantEsperando[idRecursoMin]++;

            send asignacionRecurso[idCliente](idRecursoMin);
        } * (not empty(liberacion)){
            receive liberacion(idRecurso);
            cantEsperando[idRecurso]--;
        }
    }
}

Process recurso[id = 0 .. P]{
    Integer idCliente;

    while (true){
        receive peticion[id](idCliente);
        // HACE ALGO
        send respuesta[idCliente]();
        send liberacion(id);
    }
}
```


## PMS

N clientes N recursos - con distinta prioridad
N clientes N recursos - ordenado (no implicito)

```c
Process cliente[id = 0..N]{
    Boolean prioridad = getPrioridad();
    Integer idRecurso;

    if (prioridad)
        coordinador!solicitarRecurso(id);
    else
        coordinador!solicitarRecursoPrioritario(id);

    coordinador?asignarRecurso(idRecurso);
    recurso[idRecurso]!peticion(id);
    recurso[idRecurso]?respuesta();
}

Process coordinador{
    Cola normal;
    Cola prioritarios;
    Integer idCliente, idRecurso;

    do recurso[*]?solicitarRecurso(idCliente) {
        normal.push(idCliente);
    } * recurso[*]?solicitarRecursoPrioritario(idCliente){
        prioritarios.push(idCliente);
    } * (prioritarios.empty() and not normal.empty()); recurso[*]?disponible(idRecurso){
        idCliente = normal.pop();
        cliente[idCliente]!asignarRecurso(idRecurso);
    } * (not prioritarios.empty()); recurso[*]?disponible(idRecurso){
        idCliente = prioritarios.pop();
        cliente[idCliente]!asignarRecurso(idRecurso);
}

Process recurso[id = 0..P]{
    Integer idCliente;

    while (true){
        coordinador!disponible(id);

        cliente[*]?peticion(idCliente);
        // HACE ALGO
        cliente[idCliente]!respuesta();
    }
}

```

