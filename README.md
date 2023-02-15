


# Modelo vista controlador (MVC)
Modelo Vista Controlador (MVC) es un estilo de arquitectura de software que separa los datos de una aplicación, la interfaz de usuario, y la lógica de control en tres componentes distintos.

Se trata de un modelo muy maduro y que ha demostrado su validez a lo largo de los años en todo tipo de aplicaciones, y sobre multitud de lenguajes y plataformas de desarrollo.

#### El Modelo que contiene una representación de los datos que maneja el sistema, su lógica de negocio, y sus mecanismos de persistencia.
#### La Vista, o interfaz de usuario, que compone la información que se envía al cliente y los mecanismos interacción con éste.
#### El Controlador, que actúa como intermediario entre el Modelo y la Vista, gestionando el flujo de información entre ellos y las transformaciones para adaptar los datos a las necesidades de cada uno.

## ¿Por qué deberías usar MVC?
Tres palabras: separación de preocupaciones (separation of concerns), o SoC para abreviar.

El patrón MVC te ayuda a dividir el código frontend y backend en componentes separados. De esta manera, es mucho más fácil administrar y hacer cambios a cualquiera de los lados sin que interfieran entre sí.

Pero esto es más fácil decirlo que hacerlo, especialmente cuando varios desarrolladores necesitan actualizar, modificar o depurar una aplicación completada simultáneamente.
 
### El modelo es el responsable de:
*Acceder a la capa de almacenamiento de datos. Lo ideal es que el modelo sea independiente del sistema de almacenamiento.

*Define las reglas de negocio (la funcionalidad del sistema). Un ejemplo de regla puede ser: "Si la mercancía pedida no está en el almacén, consultar el tiempo de entrega estándar del proveedor".

*Lleva un registro de las vistas y controladores del sistema.

*Si estamos ante un modelo activo, notificará a las vistas los cambios que en los datos pueda producir un agente externo (por ejemplo, un fichero por lotes  que actualiza los datos, un temporizador que desencadena una inserción, etc.).
 

### El controlador es responsable de:
 *Recibe los eventos de entrada (un clic, un cambio en un campo de texto, etc.).

*Contiene reglas de gestión de eventos, del tipo "SI Evento Z, entonces Acción W". Estas acciones pueden suponer peticiones al modelo o a las vistas. Una de estas peticiones a las vistas puede ser una llamada al método "Actualizar()". Una petición al modelo puede ser "Obtener_tiempo_de_entrega ( nueva_orden_de_venta )". 
 

### Las vistas son responsables de:
*Recibir datos del modelo y los muestra al usuario.

*Tienen un registro de su controlador asociado (normalmente porque además lo instancia).

*Pueden dar el servicio de "Actualización()", para que sea invocado por el controlador o por el modelo (cuando es un modelo activo que informa de los cambios en los datos producidos por otros agentes).

### Ejemplo
#### Modelo:
En la aplicación Car Clicker, el objeto modelo contiene un arreglo de objetos car con toda la información (datos) necesaria para la aplicación.

También gestiona el carro actual que se muestra con una variable que se establece inicialmente en null.

    currentCar: null,
    cars: [
        {
            clickCount: 0,
            name: 'Coupe Maserati',
            imgSrc: 'img/black-convertible-coupe.jpg',
        },
        {
            clickCount: 0,
            name: 'Camaro SS 1LE',
            imgSrc: 'img/chevrolet-camaro.jpg',
        },
        {
            clickCount: 0,
            name: 'Dodger Charger 1970',
            imgSrc: 'img/dodge-charger.jpg',
        },
        {
            clickCount: 0,
            name: 'Ford Mustang 1966',
            imgSrc: 'img/ford-mustang.jpg',
        },
        {
            clickCount: 0,
            name: '190 SL Roadster 1962',
            imgSrc: 'img/mercedes-benz.jpg',
        },
    ]
};
 
#### Vista(UI)
La aplicación "Car Clicker" tiene dos vistas: carListView y CarView.

Ambas vistas tienen dos funciones críticas que definen lo que cada vista quiere inicializar y renderizar.

Estas funciones son donde la aplicación decide lo que el usuario verá y cómo.


#### carListView 

const carListView = {

    init() {
        // almacene el elemento DOM para un fácil acceso más tarde
        this.carListElem = document.getElementById('car-list');

        // renderizar esta vista (actualizar los elementos DOM con los valores correctos)
        this.render();
    },

    render() {
        let car;
        let elem;
        let i;
        // obtener los carros para ser renderizados desde el controlador
        const cars = controller.getCars();

        // para asegurarse de que la lista está vacía antes de renderiz
        this.carListElem.innerHTML = '';

        // bucle sobre el arreglo de carros
        for(let i = 0; i < cars.length; i++) {
            // este es el carro que tenemos en bucle
            car = cars[i];

            // hacer un nuevo elemento de la lista de carros y establecer su texto
            elem = document.createElement('li');
            elem.className = 'list-group-item d-flex justify-content-between lh-condensed';
            elem.style.cursor = 'pointer';
            elem.textContent = car.name;
            elem.addEventListener(
                'click',
                (function(carCopy) {
                    return function() {
                        controller.setCurrentCar(carCopy);
                        carView.render();
                    };
                })(car)
            );
            // finalmente, agregua el elemento a la lista
            this.carListElem.appendChild(elem);
        }
    },
};
#### CarView
const carView = {

    init() {
        // almacene punteros a los elementos DOM para un fácil acceso más tarde
        this.carElem = document.getElementById('car');
        this.carNameElem = document.getElementById('car-name');
        this.carImageElem = document.getElementById('car-img');
        this.countElem = document.getElementById('car-count');
        this.elCount = document.getElementById('elCount');


        // al hacer clic, aumentar el contador del carro actual
        this.carImageElem.addEventListener('click', this.handleClick);

        // renderizar esta vista (actualizar los elementos DOM con los valores correctos)
        this.render();
    },

    handleClick() {
    	return controller.incrementCounter();
    },

    render() {
        // actualizar los elementos DOM con valores del carro actual
        const currentCar = controller.getCurrentCar();
        this.countElem.textContent = currentCar.clickCount;
        this.carNameElem.textContent = currentCar.name;
        this.carImageElem.src = currentCar.imgSrc;
        this.carImageElem.style.cursor = 'pointer';
    },
};

#### Controlador
A través de las funciones getter y setter, el controlador extrae datos del modelo e inicializa las vistas.

Si hay alguna actualización desde las vistas, modifica los datos con una función setter.

const controller = {
    
    init() {
        // establecer el carro actual como el primero en la lista
        model.currentCar = model.cars[0];

        // indicar a las vistas que inicialicen
        carListView.init();
        carView.init();
    },

    getCurrentCar() {
    	return model.currentCar;
    },

    getCars() {
    	return model.cars;
    },

    // establecer el carro seleccionado actualmente en el objeto que se pasa en
    setCurrentCar(car) {
    	model.currentCar = car;
    },

    // incrementar el contador para el coche seleccionado actualmente
    incrementCounter() {
        model.currentCar.clickCount++;
        carView.render();
    },
};

controller.init();

# Infraestructura Almacenaje y Recuperacion de datos
Un datacenter es una infraestructura física o virtual utilizada para alojar sistemas informáticos que puedan procesar, servir o almacenar datos. Data Center dan servicio de almacenamiento de datos, respaldo o backup, recuperación de datos y gestión de la información para empresas

 

### Componentes y funcionamiento
#### Servidores: 
El propósito principal de un datacenter o CPD es alojar los servidores necesarios para soportar los servicios ofrecidos a los clientes. El personal cualificado se encarga de que todos los servidores esten actualizados. Para que tengan un perfecto funcionamiento tanto software (Sistemas operativos, actualizaciones críticas, aplicaciones, copias de seguridad, parches) como hardware(memorias, discos duros, cpu´s etc). Estos servidores se colocan en grandes armarios denominados rack. El proveedor del alojamiento proporciona el ancho de banda, la seguridad, refrigeración e instalaciones. Para tener en condiciones de uso y rendimiento optimo los servidores.
 
 

### Sistemas de funcionamiento
#### Conectividad de red: 
Mediante switches todos los servidores reciben y entregan información desde la red y hacia la red según la demanda y el trabajo al que estén destinados.

#### Energía: 
Se necesita una fuente de alimentación para mantener todo este conjunto en marcha. Normalmente se usan fuentes redundantes y electro-generadores diésel para abastecer a todo el sistema en caso de fallo eléctrico. Los sistemas eléctricos deben de mantenerse constantes y sin fluctuaciones de voltaje o intensidad los cuales pueden perjudicar a todo el conjunto

#### Climatización:
 La carga de trabajo a la que se someten los sistemas de un datacenter o CPD generan unas condiciones de calor muy elevadas. Para evitar sobrecalentamientos existen uno o varios sistemas de ventilación que pueden utilizar aire frio o líquidos refrigerantes para mantener una temperatura adecuada. También se tiene en cuenta la disposición de los servidores para que la evacuación natural del aire sea la mejor posible.

#### Monitorización:
 La información y procesos que alberga un datacenter o Centro de procesamiento de datos es en la mayoría de los casos crítica, un fallo en el servidor. Por ejemplo se dedique al procesamiento de los datos de tarjetas de crédito puede dejar en jaque a miles o millones de personas. Ir siempre un paso por delante de estos fallos o atajarlos inmediatamente es la labor de personal altamente cualificado. Que se dedica segundo a segundo a velar porque todo funcione correctamente.

#### Sistemas de seguridad:
 Sistemas contraincendios, edificios con construcciones anti-seismos, vigilantes de seguridad, sistemas de accesos restringidos etc. Según el contenido de sus servidores las empresas que gestionan los datacenters o Centro de procesamiento de datos velan por la seguridad e integridad de todo el sistema.

 