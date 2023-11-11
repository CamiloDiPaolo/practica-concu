# PMS

# Envio
destino!port(mensaje)
destino[i]!port(mensaje)

# Recepcion
origen?port(variable)
origen[i]?port(variable)
origen[*]?port(variable)

# Do
do [CONDICION]; PROCESO?PUERTO() => 
    {CUERPO QUE SE EJECUTA SI SE ESTABLECE CONEXION}
* PROCESO?PUERTO() =>
    {CUERPO QUE SE EJECUTA SI SE ESTABLECE CONEXION}
od

# If
if [CONDICION]; PROCESO?PUERTO() => 
    {CUERPO QUE SE EJECUTA SI SE ESTABLECE CONEXION}
* PROCESO?PUERTO() =>
    {CUERPO QUE SE EJECUTA SI SE ESTABLECE CONEXION}
fi;

# Diferencia entre If y Do

IF: si todas las guardas tienen el estado NO ELEGIBLE (la condicion es falsa) se sale sin hacer nada;
    si alguna guarda esta en BLOQUEADA(se esta esperando que se establezca la comunicacion) es queda esperando;

DO: itera de la misma manera que el IF hasta que todas las guardas sean NO ELEGIBLES;
    ESTA ESTRUCTURA ITERA, es decir que puede quedarse loop mientras alguna sea elegible.

## 1
Resolver con PASAJES DE MENSAJES SINCRÓNICOS (PMS) el siguiente problema. Existe un sitio
de algebra lineal especializado en resolver multiplicaciones de matrices que atiende
pedidos de N clientes, el sitio tiene un Servidor encargado de resolver los pedidos de los
clientes de acuerdo con el orden en que se hacen los mismos. Cada cliente hace un solo pedido.
NOTA: todas las tareas deben terminar.

```cs
Process cliente[id = 0..N-1]{
    Matriz m = new Matriz();
    Intege response;

    encolador!req(m, id);
    servidor?res(response);
}

Process encolador{
    Cola cola = new Cola();
    Integer idCliente;
    Matriz matrizCliente;

    for i in 0..2* (N - 1) {
        do cliente[*]?req(m, id) => 
            cola.push(m , id);
        * not empty(cola); servidor?pedido() => 
            cola.pop(matrizCliente, idCliente);
            servidor!req(matrizCliente, matrizCliente);j
    } 
}

Process servidor{
    Integer response;
    Integer idCliente;
    Matriz matrizCliente;

    for i in 0..(N - 1) {
        encolador!pedido();
        encolador?res(matrizCliente, idCliente);
        response = resolver(matrizCliente);
        cliente[idCliente]!res(response);
    }
}
```

## 2
Resolver con Pasaje de Mensajes Sincrónicos (PMS) el siguiente problema. En un torneo de programación
hay 1 organizador, N competidores y S supervisores. El organizador comunica el desafio a resolver a cada
competidor. Cuando un competidor cuenta con el desafio a resolver, lo hace y lo entrega para ser evaluado.
A continuación, espera a que alguno de los supervisores lo corrija y le indique si está bien. En caso de
tener errores, el competidor debe corregirlo y volver a entregar, repitiendo la misma metodología hasta
que llegue a la solución esperada. Los supervisores corrigen las entregas respetando el orden en que los
competidores van entregando.
Nota: maximizar la concurrencia y no generar demora innecesaria.

```cs
Process organizador{
    Text enunciado = obtenerEnunciado();

    for i in 0..(N-1){
        competidor[id]!comenzar(enunciado);
    }
}

Process competidor[id = 0..(N-1)]{
    Text enunciado, respuesta;
    Boolean correcto;

    organizador?comenzar(enunciado);

    correcto = false;

    while (not correcto){
        respuesta = resolver(enunciado);
        entrega!entregar(respuesta, id);
        supervisor[*]?correccion(correcto);
    }
}

Process entrega{
    Cola cola = new Cola();
    Text respuesta;
    Integer idCompetidor;

    while(true){
        do entrega?entregar(res, id) => 
            cola.push(res, id);
        * not empty(cola); supervisor?pedido() => 
            cola.pop(respuesta, idCompetidor);
            supervisor!entregar(respuesta, idCompetidor);
        od;
    }
}

Process supervisor[id = 0..(S-1)]{
    Text res;
    Integer idCompetidor;

    while(true){
        entrega!pedido();
        entrega?entregar(res, idCompetidor);

        correcto = corregir(res);
        competidor[idCompetidor]!correccion(correcto);
    }
}
```

## 3
En una exposición de aeronautica N personas quieren usar un simulador, solo pueden acceder de a uno
Hay un unico empleado. Las personas lo usan un rato y luego se retiran

```
Process persona[id = 0..N-1]{
    empleado!solicitar(id);
    empleado?permiso();

    usarSimulador();

    empleado!irse();
    
}

Process empleado{
    Integer id;
    
    for i in 0..N-1 {
       persona[*]?solicitar(id);
       persona[id]!permiso();
       persona[id]?irse();
    }
}
```

## 4
En un banco se tiene un sistema que administra el uso de una sala de reuniones
por parte de N clientes. Los clientes se clasifican en habituales o temporales.
La sala puede ser usada por un unico cliente a la vez
Y cuando esta libre se debe determinar a quien permitirle su uso siempre priorizando a los clientes habituales
Dentro de cada clase de cliente se debe respetar el orden de llegada
Nota: suponga que existe una funcion tipo() que le indica al cliente de que tipo esta

```
Process cliente[id = 0..N-1]{
    Boolean habitual = tipo();
    
    sala!encolar(id, habitual);

    sala?entrar();

    hacerCosasEnLaSala();

    sala!salir();
}

Process admin{
    Cola colaHabitual = new Cola();
    Cola colaTemporal = new Cola();

    Integer idCliente;
    Boolean habitual;

    do cliente[*]?encolar(idCliente, habitual) => 
        if (habitual)
            colaHabitual.push(idCliente);
        else
            colaTemporal.push(idCliente);
    * sala?siguiente() =>
        if (not colaHabitual.empty){
            colaHabitual.pop(idCliente);
        } else {
            colaTemporal.pop(idCliente);
        }
        sala!entrar(idCliente);
    od;
}

Process sala{
    Integer idCliente;

    while (true){
        sala!siguiente();
        sala?entrar(idCliente);

        cliente[idCliente]!entrar();
        cliente[*]?salir();
    }
}
```

## 4 Otra solucion que no esta mal pero tiene busy wating

```
process Cliente[id: 0..N-1] {
    String tipo = Tipo();
    SalaDeReuniones!Encolar(id, tipo);
    SalaDeReuniones!Pedido();
    SalaDeReuniones?Usar();
    delay(random());
    SalaDeReuniones!Liberar();
}

process SalaDeReuniones {
    queue colaHabituales, colaTemporales;
    int idC,
    String tipo;
    bool libre = true;
    
    do  Cliente[*]?Encolar(idC, tipo) ->
            if (tipo == "habitual") push(colaHabituales, idC);
            else push(colaTemporales, idC);
    * (libre && (!empty(colaHabituales))) ; Cliente[*]?Pedido() ->
            pop(colaHabituales, idC);
            Cliente[idC]!Usar();
            libre = false;
    * (libre && (!empty(colaTemporales)) && (empty(colaHabituales))) ; Cliente[*]?Pedido() ->
            pop(colaTemporales, idC);
            Cliente[idC]!Usar();
            libre = false;
    * Cliente[*]?Liberar ->
            libre = true;
    od
}
```

## 5
Resolver con PMS el siguiente problema: Se debe administrar el acceso para usar en
determinado Servidor donde no se permite más de 10 usuarios trabajando al mismo tiempo,
por cuestiones de rendimiento. Existen N usuarios que solicitan acceder al Servidor,
esperan hasta que se les de acceso para trabajar en él y luego salen del mismo.
Nota: suponga que existe una función TrabajarEnServidor() que llaman los usuarios para
representar que están trabajando en el Servidor

```cs
Process usuario[id = 0 .. N - 1]{
    servidor!acceso(id);
    servidor?habilitar();
    TrabajarEnServidor();
    servidor!salida();
}

Process servidor{
    Integer usuarios = 0;
    Integer idCliente;
    
    do (usuarios < 10) ; cliente[*]?acceso(idCliente) =>
        usuarios++;
        usuario[idCliente]!habilitar();
    * cliente[*]?salida() => 
        usuarios--;
    od;
}
```






