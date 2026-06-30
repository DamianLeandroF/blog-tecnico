# 📜 Blog Técnico: Desafíos en el Desarrollo de Software

Bienvenido a mi espacio de documentación técnica. En esta entrada detallo el proceso de resolución, los aprendizajes y la gestión de entornos de desarrollo basados en experiencias reales de trabajo en equipo.

---

## 🛠️ Entrada: Sincronización de Flujos en Backend y Git (Caso Ticket hph-366)

### 1. Contexto
Este desafío se llevó a cabo en el entorno de desarrollo de **HarryPotterHead**, un foro web basado en el motor de código abierto **SMF (Simple Machines Forum)**, implementado con PHP y bases de datos MySQL/MariaDB. El objetivo del ticket `dev/hph-366` era expandir las capacidades del sistema de BBCode para permitir que las menciones dinámicas de usuarios (con formato `@Nombre Completo`) funcionaran correctamente cuando se inyectaban bloques de código personalizados mediante la etiqueta `[html]`.

### 2. Problema
Durante las pruebas de integración en el entorno local, surgieron dos problemas críticos de distinta naturaleza:

* **Inconsistencia en el Backend:** Al renderizar los posts, las etiquetas de salto de línea explícitas (`<br>`) y ciertos estilos CSS inline dentro del bloque `[html]` se rompían o no aplicaban su color de fondo. El análisis del flujo reveló un problema de orden de operaciones: la función global nativa de SMF (`strtr`) transformaba los saltos de línea `\n` en `<br>` **antes** de que nuestro Hook personalizado pudiera aislar y escapar el bloque `[html]`, destruyendo el formateo manual del usuario.
* **Falla del Entorno Local (Crash de Base de Datos):** Debido a un cierre abrupto del sistema durante el workflow de Git, el motor InnoDB de MariaDB sufrió una corrupción en su secuencia de transacciones, arrojando el error `[ERROR] InnoDB: Missing MLOG_CHECKPOINT` y bloqueando por completo el arranque del servidor local en XAMPP.

### 3. Acciones Tomadas (y Post-Mortem)
Para solucionar ambos frentes de forma sistemática sin perder el progreso del código, se ejecutó el siguiente plan de acción:

1. **Recuperación del Servidor MySQL:** Se editó el archivo de configuración `my.ini` inyectando la directiva `innodb_force_recovery = 3` para forzar un inicio seguro del motor ignorando los índices corruptos. Una vez en verde, se purgaron los archivos de registro de transacciones dañados (`ib_logfile0` e `ib_logfile1`) y se retiró el modo recuperación, devolviendo la base de datos a un estado 100% operativo.
2. **Estrategia de Git Workflow:** Tras usar por error un comando de limpieza drástico (`git reset --hard`) que barrió los cambios del editor, se utilizó `git reflog` para ubicar el hash exacto del commit perdido (`edba8ac`) y restaurar la lógica desarrollada. Posteriormente, se aplicó un `git rebase origin/main` para mudar el trabajo sobre la última versión limpia de producción, resolviendo los conflictos del merge mediante el editor de VS Code.
3. **Refactorización de la Lógica de Escape (Solución al Backend):** Siguiendo las directivas de arquitectura del core de `preparsecode()`, se modificó el comportamiento del guardado para sustituir temporalmente los caracteres sensibles y saltos de línea por entidades numéricas (`&#13;`). Esto garantizó que el contenido sobreviviera intacto a los filtros globales antes de ser interpretado en el renderizado final de `parse_bbc()`.
4. **Entorno de Aislamiento (Staging Local):** Se creó un Topic de pruebas aislado en el `localhost` conteniendo múltiples tipos de tablillas CSS complejas utilizadas por la comunidad. Esto sirvió como un "Stress Test" para validar que la nueva función de menciones operara correctamente sin alterar el diseño de los posts antiguos.

### 4. Aprendizajes
* **Arquitectura de Software:** En sistemas basados en eventos o ganchos (Hooks), el orden exacto en el que se procesa, sanitiza y escapa una cadena de texto determina la integridad del producto. Modificar datos antes o después de los filtros globales del Core cambia drásticamente el comportamiento del renderizador.
* **Robustez en Control de Versiones:** El dominio de herramientas avanzadas de Git como `reflog` y el uso consciente de `git rebase` en lugar de merges tradicionales son indispensables para mantener un historial de commits limpio, profesional y auditable por los revisores de código (*Reviewers*).

### 5. Reflexión sobre Feedback Radicalmente Sincero
"Durante el ciclo de desarrollo de este ticket, experimenté la aplicación de un feedback radicalmente sincero por parte del líder técnico del proyecto (Aiden) al señalarme de forma directa e incontestable que estaba asumiendo la presencia de arquitecturas erróneas en mis explicaciones teóricas (como el uso del stack MERN o expresiones regulares en un entorno de lógica pura de backend en PHP).

Lejos de tomar la observación como una crítica personal, decidí aplicar una mentalidad de crecimiento enfocada en la resolución del problema: acepté el error de lectura del flujo, limpié el historial de Git para eliminar commits intermedios desprolijos y dediqué tiempo a estudiar minuciosamente el comportamiento de las funciones nativas del core del foro. Esta honestidad brutal dentro del equipo eliminó la complacencia, aceleró el entendimiento del bug real de los saltos de línea y elevó significativamente la calidad técnica del código final entregado."

---

## 🏁 Checklist de Entrega

* **[X]** Entrada de blog publicada y accesible públicamente.
* **[X]** Documentación clara y estructurada según la plantilla solicitada.
* **[X]** Evidencia de control de versiones integrada.
* **[X]** Reflexión sobre feedback radicalmente sincero incluida.

### 🔗 Enlaces del Proyecto
* **Repositorio del Blog:** [https://github.com/damianleandrof/blog-tecnico](https://github.com/damianleandrof/blog-tecnico)
* URL pública del Blog: [https://damianleandrof.github.io/blog-tecnico/](https://damianleandrof.github.io/blog-tecnico/)
### 📊 Evidencia de Control de Versiones (Snapshot de Git)
*Nota: Dado que el repositorio de desarrollo del foro es de carácter privado, se adjunta el registro real del historial de commits de la rama `dev/hph-366` como evidencia del flujo de trabajo:*

```text
57041e3 (HEAD -> dev/hph-366, origin/main) HEAD@{0}: reset: moving to origin/main
edba8ac HEAD@{1}: commit: hph-366: Implementación completa de menciones en bloques HTML
303d2f8 HEAD@{2}: reset: moving to HEAD~2
5897f86 HEAD@{3}: commit: hph-366 correccion de [html]
4d4f80c HEAD@{4}: commit: hph-366: Implementación de menciones en bloques [html]
