# Descripción de la Situación Actual y Solución Propuesta

Actualmente me desempeño como desarrollador de software dentro de la Agencia de Aduanas Capricornio S.A., empresa dedicada a la gestión de operaciones aduaneras. Durante mi permanencia en la organización he podido conocer de primera mano la evolución de los sistemas que soportan los procesos operativos de la empresa, así como las limitaciones que han surgido a medida que el negocio ha crecido y los requerimientos tecnológicos se han vuelto más complejos.

La operación diaria de la empresa se encuentra soportada principalmente por ASINSA, una solución especializada para el sector aduanero que ha acompañado a la organización durante muchos años. Debido a las necesidades particulares del negocio y a las limitaciones de personalización de dicha plataforma, con el tiempo se desarrolló una solución complementaria denominada Capricornio Net (CapriNet), construida utilizando PHP con CodeIgniter para la capa de presentación y Slim Framework para la capa de servicios.

Este sistema fue concebido con el objetivo de abstraer gran parte de las operaciones realizadas sobre ASINSA, simplificar ciertos procesos para los usuarios y agregar funcionalidades que no estaban contempladas originalmente en la solución principal. Para ello, Capricornio Net interactúa directamente con las bases de datos existentes de ASINSA y además incorpora una base de datos complementaria que almacena información propia de los nuevos procesos implementados.

A medida que la empresa continuó creciendo, surgió la necesidad de modernizar la plataforma tecnológica. Como respuesta a esta necesidad se inició un proceso progresivo de migración hacia una arquitectura moderna basada en Angular para la capa de presentación y microservicios desarrollados en Java Spring Boot para la capa de negocio. Esta estrategia sigue el patrón conocido como Strangler Fig Pattern o Patrón de Estrangulamiento, permitiendo reemplazar gradualmente componentes legacy sin afectar la continuidad operativa de la organización.

Actualmente ambas plataformas conviven dentro del ecosistema tecnológico de la empresa. Por un lado, continúa operando el sistema legacy desarrollado en PHP y, por otro, se siguen incorporando nuevas funcionalidades en la plataforma moderna basada en Angular y microservicios. Adicionalmente, existen procesos automatizados implementados mediante robots desarrollados en Python que participan en distintos flujos operativos.

Durante el análisis de la operación diaria se identificó una problemática importante relacionada con el mecanismo de alertas y notificaciones utilizado por los sistemas corporativos.

Originalmente, el sistema PHP contaba con un conjunto de alertas destinadas a informar a los usuarios sobre actividades pendientes, eventos importantes y acciones requeridas dentro de diversos módulos de negocio. Sin embargo, el diseño de esta funcionalidad presentaba múltiples limitaciones.

La generación de alertas dependía directamente de consultas ejecutadas desde cada pantalla del sistema. Cada vez que un usuario ingresaba a una opción determinada, el sistema realizaba consultas específicas para identificar situaciones que requerían atención. Este comportamiento se repetía constantemente para múltiples usuarios y múltiples pestañas abiertas simultáneamente.

Además, la lógica de generación y visualización de alertas se encontraba implementada de forma rígida mediante código JavaScript, definiendo de manera manual qué usuarios o perfiles podían visualizar determinados mensajes. Esto provocaba una alta dependencia del código fuente, dificultando el mantenimiento y limitando la capacidad de adaptación ante nuevos requerimientos.

Como consecuencia de este diseño, el sistema comenzó a generar una carga considerable sobre la base de datos. Las consultas necesarias para construir las alertas se ejecutaban repetidamente desde diferentes módulos y sesiones activas, provocando degradación del rendimiento y afectando la estabilidad general de la plataforma.

Debido al impacto generado sobre los recursos de base de datos, la funcionalidad terminó siendo deshabilitada parcialmente, dejando a los usuarios sin un mecanismo centralizado y eficiente para recibir información relevante sobre los procesos operativos.

A partir de esta problemática se plantea el desarrollo de una Plataforma Centralizada de Gestión de Alertas Operacionales, cuyo objetivo es desacoplar completamente la gestión de alertas de los sistemas de negocio existentes.

La solución propuesta consiste en la implementación de un servicio centralizado desarrollado en Java Spring Boot encargado de administrar el ciclo de vida completo de las alertas. Este servicio será responsable de la creación, consulta, actualización, lectura, eliminación y programación de alertas dentro de la organización.

A diferencia del modelo anterior, las alertas dejarán de generarse mediante consultas ejecutadas desde las interfaces de usuario. En su lugar, los distintos sistemas de la empresa, incluyendo microservicios, aplicaciones web y procesos automatizados, podrán registrar eventos relevantes directamente en la plataforma centralizada de alertas.

Para garantizar la distribución inmediata de la información se implementará un servidor WebSocket especializado que permitirá la comunicación en tiempo real con los usuarios conectados. Gracias a este mecanismo, las alertas podrán aparecer instantáneamente en pantalla sin necesidad de recargar páginas ni ejecutar consultas periódicas a la base de datos.

La arquitectura propuesta contempla además un mecanismo denominado Mapeo Conceptual, mediante el cual cada alerta será asociada a tipos, subtipos y claves de negocio. Este enfoque permitirá administrar las alertas de manera desacoplada respecto de las tablas operativas existentes, facilitando la eliminación individual o masiva de registros sin alterar la estructura de los sistemas de negocio.

La plataforma incluirá una bandeja centralizada de alertas desarrollada en Angular, desde la cual los usuarios podrán visualizar y administrar toda la información pendiente de atención. Las alertas podrán clasificarse como informativas o de acción.

Las alertas informativas permitirán al usuario consultar información relevante, marcar elementos como leídos y configurar mecanismos de eliminación automática luego de un periodo determinado.

Por otro lado, las alertas de acción actuarán como puntos de entrada a procesos específicos del negocio. Al seleccionar una alerta, el usuario será redirigido automáticamente al módulo correspondiente, ya sea dentro de la plataforma moderna o dentro del sistema legacy. Una vez completada la actividad asociada, la alerta podrá ser eliminada automáticamente, reflejando que el proceso fue atendido satisfactoriamente.

Con la finalidad de evitar duplicidad de esfuerzos de desarrollo y garantizar una experiencia unificada para los usuarios, la misma bandeja de alertas desarrollada en Angular será reutilizada dentro del sistema legacy PHP mediante integración a través de iframe y mecanismos seguros de comunicación entre aplicaciones utilizando autenticación JWT.

Adicionalmente, la solución incorporará un módulo administrativo que permitirá gestionar dinámicamente la distribución de alertas. Los administradores podrán configurar qué áreas, perfiles, roles o usuarios específicos tendrán acceso a cada tipo y subtipo de alerta, eliminando la necesidad de realizar modificaciones en el código fuente para adaptar el comportamiento del sistema.

Como parte de la evolución de la plataforma, también se contempla la generación de métricas e indicadores operacionales. Gracias a la centralización de eventos y al seguimiento del ciclo de vida de las alertas, será posible obtener información relacionada con tiempos de atención, volumen de incidencias, cumplimiento de procesos y otros indicadores relevantes para la toma de decisiones.

La implementación inicial se realizará mediante un entorno sandbox que simulará las condiciones reales de operación. Se usará el módulo CRM como plan piloto para mostrar la solución a traves de un proceso completo simulado(no necesariamente real y exacto al existente), permitiendo validar el funcionamiento de la solución antes de extender progresivamente la integración hacia otros procesos de la organización.

Con esta propuesta se busca reducir significativamente la carga generada sobre la base de datos, centralizar la gestión de alertas, mejorar la experiencia de los usuarios mediante información en tiempo real, facilitar la evolución tecnológica de la organización y establecer una base sólida para futuras iniciativas de monitoreo y gestión de procesos operacionales.

Soy Estudiante de la carrera Técnica de Desarrollo de Sistemas de Información en el instituto IDAT en Perú, actualmente me encuentro en el último ciclo de la carrera. Y estoy haciendo un TAP, Trabajo de Aplicación Profesional dando esta solución a la problemática que se ha identificado en la empresa donde me encuentro trabajando.

El contexto servira para el Curso de Proyecto Integrador 2 y proyecto Integrador 3, el primero se hara en metodologia RUP y el segundo se hara en metodologia SCRUM, ambos con la misma problemática y solucion propuesta.

El ciclo completo durará 4 meses, iniciando en el mes de mayo día 11 y finalizando el 11 de septiembre del 2026. Durante este periodo se llevarán a cabo las distintas fases del proyecto, desde la planificación y análisis inicial, pasando por el diseño, desarrollo, pruebas e implementación de la solución propuesta. El objetivo es entregar una plataforma robusta y eficiente que permita mejorar significativamente la gestión de alertas operacionales dentro de la Agencia de Aduanas Capricornio S.A., contribuyendo así al éxito y crecimiento continuo de la organización.

## Arquitectura y Contexto Técnico de Integración

El proyecto implementa el patrón Strangler Fig para desacoplar el módulo de alertas de los sistemas legados (CapriNet y CapriNet 2.0), eliminando la degradación de rendimiento en la base de datos principal mediante las siguientes especificaciones:

Capa de Persistencia (Heredada): Se utiliza una base de datos independiente en MySQL 8 operando bajo el motor MyISAM (mantenido por compatibilidad y arrastre con la infraestructura preexistente de la empresa). Al no requerir lógica transaccional compleja para las alertas, este motor ofrece una lectura masiva de alta velocidad con un consumo mínimo de CPU.

Ingesta de Eventos (API REST): Toda la comunicación y envío de alertas desde los diferentes componentes del ecosistema (el monolito PHP en CodeIgniter/Slim, los daemons en Python y el Sandbox del CRM) hacia el backend de Java Spring Boot se realiza de forma estandarizada mediante peticiones a una API REST compartida emitiendo payloads en formato JSON.

Tiempo Real y Frontend: El procesamiento centralizado distribuye las notificaciones instantáneamente a una SPA en Angular a través de WebSockets, permitiendo que la interfaz embebida (vía iframe en la plataforma PHP actual) muestre eventos en tiempo real sin necesidad de recargas de pantalla ni consultas cíclicas a la base de datos