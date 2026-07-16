# Post-Mortem: Interrupción en la Carga de Contenido Dinámico

## Contexto
En el proyecto **Boomerang Institute**, una plataforma educativa diseñada para proporcionar recursos y cursos a estudiantes, la experiencia del usuario depende fuertemente de la navegación fluida. Utilizamos un script llamado `content-loader.js` para cargar dinámicamente diferentes secciones de la página (como los módulos de los cursos o el panel de administración) sin necesidad de recargar el sitio completo. Esto se logra mediante peticiones asíncronas (Fetch API) a nuestro servidor.

## Problema
Durante una fase de pruebas con usuarios que simulaban condiciones de red inestables (redes móviles 3G o conexiones con alta latencia), detectamos un desafío técnico crítico. Cuando la solicitud asíncrona para obtener el contenido tardaba demasiado o fallaba por interrupciones en la red, la aplicación no manejaba el error correctamente.

El impacto fue directo a la experiencia del usuario:
* **Pantalla en blanco o congelada:** Los usuarios hacían clic en un enlace y la página no mostraba ningún indicador visual de que estaba trabajando.
* **Falta de retroalimentación:** Si la carga fallaba definitivamente, el usuario nunca era notificado del error, asumiendo que el sitio estaba "roto" e impidiendo el acceso a los materiales del instituto.

Técnicamente, el problema radicaba en la ausencia de manejo de excepciones (`try/catch`) en el flujo de la Promesa y la falta de un componente visual de estado de carga (loading spinner).

## Acciones
Para resolver este incidente, aplicamos un enfoque de resolución estructurado sin buscar culpables, enfocándonos en la robustez del código:

1. **Análisis de Logs:** Revisamos los errores en la consola del navegador, confirmando que las peticiones HTTP estaban devolviendo errores de *Timeout* y *Network Error* que no estaban siendo capturados.
2. **Implementación de Manejo de Errores:** Refactorizamos el script `content-loader.js`. Envolvimos la lógica de las peticiones `fetch()` dentro de bloques `try/catch` para capturar cualquier fallo de red de manera elegante.
3. **Mejora de UX (Indicadores visuales):**
   * Añadimos un estado visual de *cargando* (un spinner animado) que se activa inmediatamente al hacer clic y se desactiva al completarse la petición.
   * En caso de error (dentro del bloque `catch`), implementamos una notificación en la interfaz que dice: *"Lo sentimos, hubo un problema al cargar el contenido. Por favor, verifica tu conexión e intenta nuevamente."*
4. **Validación:** Simulamos nuevamente entornos de baja conectividad usando las herramientas de desarrollador (Network Throttling), confirmando que ahora el usuario recibe retroalimentación inmediata y la aplicación no se congela.

## Aprendizajes
Este incidente nos dejó valiosas lecciones tanto a nivel técnico como de producto:

* **Diseñar para el peor escenario (Design for Failure):** Aprendimos que no debemos asumir que las condiciones de red del usuario siempre serán óptimas. Todo código asíncrono debe tener un plan de contingencia.
* **La UX técnica es clave:** Un simple bloque `try/catch` no solo previene que la aplicación colapse, sino que, acompañado de una buena interfaz, mantiene la confianza del usuario en la plataforma.
* **Cultura de Pruebas:** Incorporamos a nuestro flujo de trabajo la práctica de probar las aplicaciones bajo diferentes perfiles de red (Slow 3G, Offline) antes de cada despliegue, asegurando que la resiliencia sea una característica por defecto en Boomerang Institute.
