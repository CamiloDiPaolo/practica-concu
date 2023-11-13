## 1

Simular el con funcionamiento de un Entrenamiento de Básquet donde hay 20 jugadores y UN entrenador. 
El entrenador debe distribuir a los jugadores en 2 canchas. 
Cuando un jugador llega el entrenador le indica la cancha a la cual debe ir para que se dirija a ella y espere hasta que lleguen los 10;
en ese momento comienzan a jugar el partido que dura 40 minutos.
Cuando ambos partidos han terminado, el entrenador les da a los 20 jugadores una charla de 10 minutos y luego todos se retiran. 
El entrenador asigna el número de cancha en forma cíclica 1, 2, 1, 2 y así sucesivamente.

```ada
procedure ejercicio1 is

    task entrenador is;
        entry llegada(idJugador: IN Integer);
        entry terminoPartido();
    end entrenador;

    task type jugador is
        entry asignarId(nuevaId: IN Integer);
        entry asignarCancha(id: IN Integer);
        entry terminoPartido();
        entry retirarse();
    end jugador;

    task type cancha is
       entry llegar(); 
    end cancha;

    task body entrenador is
        -- asignar la cancha a cada jugador
        for i in 0 .. 19 loop
            accept llegada(idJugador: IN Integer) do
                jugadores(idJugador).asignarCancha(i MOD 2);
            end llegada;
        end loop;

        -- esperar a que todos los partidos terminen
        for i in 0 .. 1 loop
            accept terminoPartido();
        end loop;

        -- da la charla de 10 minutos y los libera
        delay 60.0 * 10;

        for i in 0 .. 19 loop
            jugadores(i).retirar();
        end loop;
    end entrenador;

    task body jugador is
        id: Integer;
        idCancha: Integer;
    begin
        -- asignacion id
        accept asignarId(nuevaId: IN integer) do
            id:= nuevaId;
        end asignarId;

        -- avisa al entrenador que llego
        entrenador.llegada(id);

        -- se le asigna la cancha
        accept asignarCancha(id: IN Integer) do
            idCancha:= id;
        end asignarCancha;
    
        -- avisa la llegada
        canchas(idCancha).llegar();
        
        -- espera a que el partido termine
        accept terminoPartido();

        -- esperan por la charla para irse
        accept retirarse();
    end jugador;

    task body cancha is 
    begin;
        -- van llegando los jugadores
        for i in 0 .. 9 loop
            accept llegar(); 
        end loop;

        -- inicia el partido
        delay 60.0 * 40;

        -- se avisa que el partido termino
        for i in 0 .. 9 loop
           jugadores((i * 2) + id).terminoPartido(); 
        end loop;
        
        entrenador.terminoPartido()
    end cancha;


    jugadores: array(0..19) of jugador;
    canchas: array(0..1) of cancha;
begin
    for i in 0 .. 19 loop
        jugadores(i).asignarId(i);
    end loop;
end ejercicio1;
```


## 2
Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores
para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia;
a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente
manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores
para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más
se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el
código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores.
Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. 

Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde
recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el
código y el valor de similitud de la huella más parecida a test en la BD correspondiente.
Maximizar la concurrencia y no generar demora innecesaria.

```ada
procedure ejercicio2 is
    
    task especialista is
        entry respuestaAnalisis(codigo: IN Integer; valor: IN Integer);
    end especialista;

    task type seridor is
        entry analizar(huella: IN Text);
    end servidor;

    task body servidor is
        huella: Text;
        codigo, valor: Integer;
    begin
        loop
            accept analizar(huellaAnalizar: IN Text) do
                huella:= huellaAnalizar;
            end analizar;

            Buscar(huella, codigo, valor);

            especialista.respuestaAnalisis(codigo, valor);
        end loop;
    end servidor;

    task body especialista is
        huella: Text;
        porcentajeMax, codigo: Integer;
    begin
        loop
            huella = obtenerHuella();

            -- envia la huella a los servidores
            for (i in 0 .. 7) loop
                servidor(i).analizar(huella)
            end loop;

            -- espera la respuesta de los servidores
            porcentajeMax:= -1;
            codigo:= -1;
            for (i in 0 .. 7) loop
                accept respuestaAnalisis(codigoHuella: IN Integer; porcentajeHuella: IN Integer) do
                    if (porcentajeHuella > porcentajeMax) then
                        porcentajeMax := porcentajeHuella;
                        codigo:= codigoHuella;
                    fi
                end respuestaAnalisis;
            end loop;

            -- termina de procesar la huella
            procesar(codigo);
        end loop;
    end especialista;

    task body servidor is
    begin
        body;
    end servidor;

    servidores: array(0..7) of servidor;
begin
    body;
end ejercicio2;
```

## 3

Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3
camiones. Hay P personas que hacen continuos reclamos hasta que uno de los camiones
pase por su casa. Cada persona hace un reclamo, espera a lo sumo 15 minutos a que llegue
un camión y si no vuelve a hacer el reclamo y a esperar a lo sumo 15 minutos a que llegue
un camión y así sucesivamente hasta que el camión llegue y recolecte los residuos; en ese
momento deja de hacer reclamos y se va. Cuando un camión está libre la empresa lo envía a
la casa de la persona que más reclamos ha hecho sin ser atendido. Nota: maximizar la
concurrencia.

task

- 3 camiones
- P personas
- 1 Administrador

ESTO NO HACE LO QUE PIDE EL ENUNCIADO, REHACER

```ada
procedure ejercicio3 is

    -- entrys
    task type camion is
        body;
    end camion;

    task type persona is
        body;
    end persona;

    task administrador is
        body;
    end administrador;

    -- bodys
    task body camion is
        id, idPersonaRecolectar: Integer;
    begin
        -- seteamos la id
        accept setId(n: IN Integer) do
            id:= n;
        end setId;

        -- logic
        loop
            administrador.camionLibre(id);

            accept asignarPersona(idPersona: IN Integer) do
                idPersonaRecolectar:= idPersona;
            end asignarPersona;

            recolectar(idPersonaRecolectar);
            personas(idPersonaRecolectar).finRecoleccion(); 
        end loop;
    end camion;

    task body persona is
        id: Integer;
        solicitar: Boolean;
        reclamos: Integer;
    begin
        -- seteamos la id
        accept setId(n: IN Integer) do
            id:= n;
        end setId;

        -- logic
        solicitar:= true;
        reclamos:= 0;
        while (solicitar) loop
            select 
                administrador.reclamo(id, reclamos);
                accept reclamoAceptado();
                accept finRecoleccion();
                solicitar:= false;
            or delay 60.0 * 15;
                reclamos:= reclamos + 1;
            end select;
        end loop;
    end persona;

    task body administrador is
        idCamionLibre, idPersonaReclamo, reclamos, maxReclamos, idMaxReclamos: Integer;

    begin
        loop
            accept camionLibre(idCamion: IN Integer) do
                idCamionLibre:= idCamion;
            end camionLibre;

            maxReclamos:= -1;
            while reclamo'count > 0 loop
                accept reclamo(idPersona: IN Integer, reclamos: IN Integer) do
                    if (reclamos > maxReclamos) then
                        maxReclamos = reclamos;
                        idMaxReclamos:= idPersona;
                    fi;
                end camionLibre;
            end loop;
            
            personas(idMaxReclamos).reclamoAceptado();
            camiones(idCamionLibre).asignarPersona(idMaxReclamos);
        end loop;
    end administrador;
    
    -- arrays
    camiones: array(0..2) of camion;
    personas: array(0..P-1) of persona;

begin
    for i in 0..2 loop
        camiones(i).setId(i);
    end loop;

    for i in 0..P-1 loop
        personas(i).setId(i);
    end loop;
end ejercicio3;
```

## 4

Existe un sitio de algebra lineal especializado en resolver
multiplicaciones de matrices que atiende pedidos de N clientes, el sitio tiene un Servidor encargado
de resolver los pedidos de los clientes de acuerdo con el orden en que se hacen los mismos. Cada
cliente hace un sólo pedido e imprime el resultado (suponga que existe una función ImprimirMatriz
para eso).

NOTA: todas las tareas deben terminar.

```ada
procedure ejercicio4 is
        
    task servidor is 
        entry req(matriz: IN Matriz; res: OUT Integera);
    end servidor;

    task type cliente;

    task body servidor is
    begin
        for i in 0 .. N - 1 loop
            accept req(matriz: IN Matriz; res: OUT Integer) do
                res = resolver(matriz);
            end req;
        end loop;
    end servidor;

    task body cliente is 
        matriz: array(0..10) of array(0..10) of Integer;
        res: Integer;
    begin
        servidor.req(matriz, res);
        ImprimirMatriz(res);
    end cliente;

    clientse: array(0.. N - 1) of cliente;
begin
    body;
end ejercicio4;
```

## 5

Se debe modelar el funcionamiento de un banco donde asisten
N clientes que van a realizar un depósito y que son atendidos por un empleado. Los clientes pueden ser
Regulares o Premium. Los clientes Regulares solicitan atención y esperan a lo sumo 1 hora a ser atendidos
y si no se retiran. Los clientes Premium solicitan atención y, si en ese momento no los pueden atender,
esperan 5 minutos para solicitar atención nuevamente; si luego de 5 intentos no fueron atendidos, se
retiran del banco. El empleado atiende los pedidos dando prioridad a los clientes Premium. El cliente le
indica el monto a depositar, y el empleado le devuelve un comprobante de depósito. 

Nota: supongo que existe una función RealizarDeposito(Monto) que realiza el depósito y retorna el comprobante del mismo.

```ada
procedure ejercicio5 is 
    
    task empleado is
        entrys;
    end empleado;

    task type cliente is
        entrys;
    end cliente;

    task body cliente is
    begin
        loop
            select 
                when (atenderPremium'count == 0) => 
                accept atenderPremium(monto: IN Integer, comprobante: OUT Text) do
                    comprobante:= RealizarDeposito(monto);
                end atenderPremium;
                or accept atender(monto: IN Integer, comprobante: OUT Text) do
                    comprobante:= RealizarDeposito(monto);
                end atenderPremium;
            end select;
        end loop;
    end cliente;

    task body cliente
        premium: Boolean;
        monto: Integer;
        comprobante: Text;
        intentos: Integer;
    begin
        premium:= esPremium();

        monto:= getMonto();
        if (not premium) then
            select
                empleado.atender(monto, comprobante);
            or delay 60.0 * 60
                null;
            end select;
        else
            intentos:= 0;
            while (intentos < 5) loop
                select
                    empleado.atenderPremium(monto, comprobante);
                or delay 60.0 * 5
                    intentos:= intentos + 1;
                end select;
            end loop;
        end if;
    end empleado;

    clientes: array(0..N-1) of cliente;
begin
    null;
end ejercicio5;
```

## 6

N usuarios y M directores quieren usar una impresora de uno a la vez, deben pasar en orden
dando prioridad siempre a los directores. Los usuarios esperan hasta imprimir luego se retiran.
Los directores esperan hasta 5' si no lo logran se retiran sin imprimir.

```ada
procedure ejercicio6 is
    
    task impresora is
        entry;
    end impresora;

    task type usuario is
        entry;
    end usuario;

    task type director is 
        entry;
    end director;

    task body impresora is
    begin
        loop 
        select 
            when (impresionDirector'count == 0) => 
            accept impresion(documento: IN Text; impresion: OUT Text) do
                impresion = imprimir(documento);
            end impresion;
            or 
            accept impresionDirector(documento: IN Text; impresion: OUT Text) do
                impresion = imprimir(documento);
            end impresionDirector;
        end select;
        end loop;
    end impresora;

    task body usuario is
        documento, impresion: Text;
    begin
        impresora.impresion(documento, impresion)
    end usuario;

    task body director is
        documento, impresion: Text;
    begin
        select
            impresora.impresionDirector(documento, impresion)
        or delay 60.0 * 5
            null;
        end select;
    end director;

    usuarios: array(0..N-1) of usuario;
    directores: array(0..N-1) of directores;
begin
    null;
end ejercicio6;

```


















