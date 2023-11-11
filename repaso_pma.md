# PMA

## 1

se debe simular la atencion de un banco con 3 cajas para atender a N
clientes que pueden ser especiales o regulares. Cuando el cliente llega al banco se
dirige a la caja con menos personas esperando y se queda ahi hasta que lo terminan de
atender y le dan el comprobante de pago. Las cajas atienden a personas de acuerdo al
orden de llegada pero dando prioridad a los clientes epeciales.
Nota: maximizar la concurrencia.

```cs

Process cliente[id = 0 .. N - 1]{
    Integer idCaja;
    boolean prioridad = ObtenerPrioridad();

   
    send solicitarCaja(id);
    receive asignarCaja[id](idCaja);

    send agregarCola(id, prioridad)

    receive siguiente[id]();

    usarCaja(idCaja);

    send salirCaja[idCaja]();
}

Process admin{
    Integer[] clientesPorCaja = ([3] 0);
    Integer cajaConMenosClientes = 0;
    Integer idCliente;
    Integer min;

    while (true){
        receive solicitarCaja(idCliente);
        send asignarCaja[idCliente](cajaConMenosClientes);

        clientesPorCaja[cajaConMenosClientes]++;

        min = 99999;;
        for i in 0 .. 2{
            if (min > clientesPorCaja[i]){
                min = clientesPorCaja[i];
                cajaConMenosClientes = i;
            }
        }
    }
}

Process caja[id = 0 .. 2]{
    Cola clientes;
    Cola prioritarios;
    Integer idCliente;
    Boolean prioridad;
    Boolean libre = true;


    while (true){
        if libre and not empty(agregarCola) and not prioritarios.empty() and not clientes.empty(){
            receive agregarCola(idCliente, prioridad);
            send siguiente(idCliente);
            libre = false;

        } else if (not libre and not empty (agregarCola)){
            receive agregarCola(idCliente, prioridad);

            if (prioridad)
                prioritarios.push(idCliente);
            else
                cliente.push(idCliente);

        } else if (libre and not prioritarios.empty()){
            siguiente[prioritarios.pop()]();
            libre = false;

        } else if (libre and not clientes.empty()){
            siguiente[clientes.pop()];
            libre = false;

        } else if (not empty(salirCaja[id])){
            receive salirCaja[id];
            libre = true;

        }
    }
}
```

## 2

En un corralón se pueden pedir presupuestos por mail, para esto tiene 3 empleados para
atender los pedidos de N personas de acuerdo con el orden en que llegan los mismos.
Cada persona envía la lista de materiales a comprar y recibe el presupuesto para
dicha lista. Los empleados atienden los pedidos de a uno a la vez, cuando no hay
pedidos para pendientes ordenen materiales durante 10 minutos.
NOTA: los empleados no deben terminar.

```cs

chan siguiente[3](Integer, Text);
chan enviarLista(Integer, Text);
chan pedido(Integer);
chan presupuesto[N](Integer);

Process persona[id = 0 .. N - 1]{
   Integer presupuesto;
   Text lista;

   send enviarLista(id, lista)

   receive presupuesto[id](presupuesto);
}

Process coordinador{
    Cola personas;
    Integer idEmpleado;
    Integer idCliente;
    Text lista;

    while (true) {
        receive pedido(idEmpleado);

        if (empty enviarLista){
            send siguiente[idEmpleado](-1 , 'VACIO')
        } else {
            receive enviarLista(idCliente, lista);
            send siguiente[idEmpleado](idCliente, lista);
        }
    }
}

Process empleado[id = 0 .. 2]{
    Text lista;
    Integer presupuesto;
    Integer idCliente;

    while (true){
        send pedido(id);

        receive siguiente[id](idCliente, lista);

        if (lista !== 'VACIO'){
           presupuesto = calcularPresupuesto(lista);
           send presupuesto[idCliente](presupuesto);
        } else {
            delay(600);
        }
    }
}
```

## 3

N alumnos rinden un examen de promocion, M rinden normal, los alumnos realizan 
el examen (suponga que ya lo tienen al empezar) y luego lo entregan. Hay 3 profesores
que corrigen los examenes priorizando los de promocion, luego le dan la nota al alumno y este se retira.

```cs

chan entregaPromo(Integer, Examen);
chan entregaNormal(Integer, Examen);
chan entrega();
chan correccion[N + M](Integer);
chan pedidoProfesor(Integer);
chan entregaProfesor[3](Integer, Examen)


Process alumno[id = 0 .. N + M - 1]{
    Boolean promocion = promocion(id);
    Examen examen;
    Integer nota;

    examen = resolver();

    send entrega();

    if promocion 
        send entregaPromo(id, examen)
    else
        send entregaNormal(id, examen);

    receive correccion[id](nota);
}

Process coordinador{
    Integer idAlumno, idProfesor;
    Examen examen;

    while (true){
        receive pedidoProfesor(idProfesor);
        receive entrega();

        if(not empty entregaPromo)
            receive entregaPromo(idAlumno, examen);
        else 
            receive entregaNormal(idAlumno, examen);

        send entregaProfesor[idProfesor](idAlumnom examen); 
    }
}

Process profesor[id = 0 .. 2]{
    Integer idAlumno;
    Integer nota;
    Examen examen;

    while(true){
        send pedidoProfesor(id);

        receive entregaProfesor[id](idAlumno, Examen);
        nota = corregir(Examen);

        send correccion[idAlumno](nota)
    }
}
```

## 4

Se debe simular la atención en un peaje con 7 cabinas para atender a N vehículos (algunos de ellos son ambulancias).
Cuando el vehículo llega al peaje se dirige a la cabina con menos vehículos esperando y se queda
ahí hasta que lo terminan de atender y le dan el ticket de pago. Las cabinas atienden a los
vehículos que van a ella de acuerdo al orden de llegada pero dando prioridad a las ambulancias;
cuando terminan de atender a un vehículo le dan el ticket de pago.
Nota: maximizar la concurrencia.

```cs

chan ingresarCola(Integer, Boolean);
chan recibirTicket[N](ticket);

chan autoEntra[7]();
chan siguiente[7](Integer);
chan siguienteAmbulancia[7](Integer);
chan autoSalio(Integer);

Process vehiculo[id = 0 .. N -1]{
    Boolean ambulancia = esAmbulancia();
    Ticket ticket;

    send ingresarCola(id, ambulancia);
    receive recibirTicket[id](ticket);
}

Process coordiandor{
    Cola vehiculosPorCabina[7] = ([7] 0);

    while true {
        Boolean ambulancia;
        Integer idVehiculo, idCabina;
        Integer cabinaConMenosVehiculos = 0;
        Integer min = 99999;

        if not empty(ingresarCola) and empty(autoSalio) {
            receive ingresarCola(idVehiculo, ambulancia); 

            for (i in 0 .. 6){
                if (vehiculosPorCabina[i] < min){
                    min = vehiculosPorCabina[i];
                    idCabina = i;
                }
            }

            vehiculosPorCabina[idCabina]++;

            send autoEntro[idCabina]();
            if ambulancia
                send siguienteAmbulancia[idCabina](idVehiculo)
            else
                send siguiente[idCabina](idVehiculo)
        }

        while (not empty(autoSalio)){
            receive autoSalio(idCabina);
            vehiculosCabina[idCabina]--;
        }
        
    }
}

Process cabina[id = 0 .. 6]{
    Integer idVehiculo;

    while true {
        receive autoEntro[id]();

        if( not empty siguienteAmbulancia[id])
            receive siguienteAmbulancia[id](idVehiculo);
        else
            receive siguiente[id](idVehiculo);
        
        Ticket ticket = new Ticket(idVehiculo);

        send recibitTicket[idVehiculo](ticket);
        send autoSalio(id);
        
    }
}





```

























