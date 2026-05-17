# Taller: Integración de RabbitMQ con Spring Boot

## 1. Introducción

RabbitMQ es un sistema de mensajería basado en el protocolo AMQP (Advanced Message Queuing Protocol) que permite a las aplicaciones comunicarse entre sí mediante mensajes asíncronos. Esto desacopla la producción y el consumo de mensajes, lo cual es fundamental en arquitecturas de microservicios y aplicaciones que requieren alta disponibilidad y escalabilidad.

Spring Boot simplifica la integración con RabbitMQ proporcionando autoconfiguración y componentes listos para usar. En este taller se creó una aplicación Spring Boot desde cero que actúa como **productor** y **consumidor** de mensajes a través de RabbitMQ.

---

## 2. Diferencias con el tutorial original

El tutorial original fue diseñado en 2015 con tecnologías de esa época. Para este taller se actualizaron las versiones manteniendo exactamente la misma lógica y estructura de código.

| Aspecto | Tutorial original | Nuestra implementación |
|---|---|---|
| **Spring Boot** | 1.2.6.RELEASE (2015) | 3.2.5 (2024) |
| **Java** | 1.8 | 21 |
| **RabbitMQ** | Instalación nativa en Mac | Docker container (`rabbitmq:3-management`) |
| **IDE** | IntelliJ IDEA 14 | VS Code / terminal |
| **Sistema operativo** | Mac OS X Yosemite | Windows 11 |

**¿Por qué se cambiaron las versiones?**  
Spring Boot 1.2.6 es incompatible con Java 21. Las versiones actuales permiten ejecutar el proyecto en entornos modernos sin modificar la lógica del código.

---

## 3. Qué se aprendió

### Conceptos de RabbitMQ
- **Queue (Cola)**: Donde se almacenan los mensajes hasta que un consumidor los procesa.
- **Exchange**: Recibe mensajes del productor y los enruta a las colas correctas según reglas de enrutamiento.
- **Binding**: Enlace entre un exchange y una cola que define qué mensajes llegan a qué cola.
- **Routing Key**: Clave que determina cómo se enruta un mensaje desde el exchange hacia las colas.

### Conceptos de Spring Boot
- **`@SpringBootApplication`**: Equivale a combinar `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`. Activa la autoconfiguración de Spring.
- **`@Configuration`**: Indica que una clase contiene definiciones de beans.
- **`@Bean`**: Declara un objeto que será gestionado por el contenedor de Spring.
- **`@Autowired`**: Inyección automática de dependencias.
- **`CommandLineRunner`**: Interface que permite ejecutar código cuando la aplicación arranca.

### Componentes de Spring AMQP
- **`RabbitTemplate`**: Plantilla para enviar y recibir mensajes de forma sencilla.
- **`SimpleMessageListenerContainer`**: Contenedor que mantiene un listener escuchando una cola.
- **`MessageListenerAdapter`**: Adaptador que conecta un POJO (como `Receiver`) con el sistema de mensajería, indicando qué método procesa los mensajes.

---

## 4. Estructura del proyecto

```
tutorial-rabbitmq-spring/
├── pom.xml
└── src/
    └── main/
        ├── java/tutoriales/rabbitmq/spring/
        │   ├── TutorialRabbitmqSpringApplication.java  ← Main: envía el mensaje
        │   ├── RabbitMqConfig.java                     ← Configuración: queue, exchange, binding, listener
        │   └── Receiver.java                           ← Consumidor: recibe e imprime el mensaje
        └── resources/
            └── application.properties                  ← Configuración de conexión a RabbitMQ
```

---

## 5. Cómo ejecutar

### Prerrequisitos
- **Java 21** o superior
- **Maven 3.6+**
- **Docker** (para ejecutar RabbitMQ)

### Paso 1: Levantar RabbitMQ con Docker

```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

Esto inicia un contenedor con:
- Puerto `5672` para conexiones AMQP
- Puerto `15672` para el panel de gestión web (accesible en `http://localhost:15672`)
- Usuario por defecto: `guest` / Contraseña: `guest`

### Paso 2: Compilar y ejecutar

```bash
cd tutorial-rabbitmq-spring
mvn spring-boot:run
```

### Paso 3: Verificar la salida

Deberías ver en la consola:

```
[Application] Enviando el mensaje "Hello world!"...
[Receiver] ha recibido el mensaje "Hello world!"
```

### Detener RabbitMQ

```bash
docker stop rabbitmq
```

---

## 6. Explicación del flujo

### 6.1. Arranque de la aplicación

Cuando `SpringApplication.run()` ejecuta la aplicación:

1. **Autoconfiguración de RabbitMQ**: Spring detecta la dependencia `spring-boot-starter-amqp` y configura automáticamente un `ConnectionFactory` usando las propiedades de `application.properties`.

2. **Creación de beans**: La clase `RabbitMqConfig` (anotada con `@Configuration`) declara los beans necesarios:
   - Una cola llamada `queue_name` (no durable)
   - Un exchange de tipo `TopicExchange` llamado `exchange_name`
   - Un binding que enlaza la cola al exchange con la routing key `routing_key`
   - Un `SimpleMessageListenerContainer` que escucha la cola
   - Un `MessageListenerAdapter` que conecta el `Receiver` con el listener

3. **Declaración en el broker**: `RabbitAdmin` declara automáticamente el exchange, la cola y el binding en RabbitMQ al iniciar la conexión.

### 6.2. Envío del mensaje

La clase `TutorialRabbitmqSpringApplication` implementa `CommandLineRunner`, por lo que su método `run()` se ejecuta automáticamente al arrancar:

```java
rabbitTemplate.convertAndSend(RabbitMqConfig.EXCHANGE_NAME, RabbitMqConfig.ROUTING_KEY, MESSAGE);
```

Esto:
1. Envía el mensaje `"Hello world!"` al exchange `exchange_name`
2. El exchange usa la routing key `routing_key` para enrutar el mensaje a la cola `queue_name`
3. El mensaje queda almacenado en la cola

### 6.3. Recepción del mensaje

El `SimpleMessageListenerContainer` está escuchando constantemente la cola `queue_name`. Cuando llega el mensaje:

1. El contenedor lo toma de la cola
2. Lo pasa al `MessageListenerAdapter`
3. El adaptador invoca el método `receiveMessage()` del bean `Receiver`
4. El `Receiver` imprime el mensaje en consola

```
[Receiver] ha recibido el mensaje "Hello world!"
```

---

## 7. Resumen

En este taller se logró:

1. **Configurar RabbitMQ** usando Docker como servidor de mensajería
2. **Crear un proyecto Maven** con Spring Boot 3.2.5 y la dependencia `spring-boot-starter-amqp`
3. **Definir la infraestructura de mensajería**: cola, exchange y binding mediante beans de Spring
4. **Enviar un mensaje** usando `RabbitTemplate.convertAndSend()`
5. **Recibir el mensaje** mediante un listener configurado con `SimpleMessageListenerContainer` y `MessageListenerAdapter`
6. **Verificar el funcionamiento** con la salida exitosa en consola

La integración de RabbitMQ con Spring Boot es directa gracias a la autoconfiguración, lo que permite enfocarse en la lógica de negocio en lugar de la configuración de infraestructura.
