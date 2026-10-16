# Bitácora - Laboratorio 1.1: Spring Boot, REST, Actuator, Perfiles y DevTools

## 1. Entorno
* **Salida de `java -version`:**
  ```text
  java version "25.0.1" 2025-10-21 LTS
  Java(TM) SE Runtime Environment (build 25.0.1+8-LTS-27)
  Java HotSpot(TM) 64-Bit Server VM (build 25.0.1+8-LTS-27, mixed mode, sharing)
  ```
* **Salida de `./mvnw -v`:**
  ```text
  Apache Maven 3.9.9 (8e8579a9e76f7d015ee5ec7bfcdc97d260186937)
  Maven home: C:\Users\PC-MASTER\.m2\wrapper\dists\apache-maven-3.9.9-bin\33b4b2b4\apache-maven-3.9.9
  Java version: 25.0.1, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk-25
  Default locale: es_EC, platform encoding: UTF-8
  OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
  ```
* **Compilación inicial (`.\mvnw.cmd clean package -DskipTests`):**
  ```text
  [INFO] ------------------------------------------------------------------------
  [INFO] BUILD SUCCESS
  [INFO] ------------------------------------------------------------------------
  [INFO] Total time:  01:08 min
  [INFO] Finished at: 2026-10-16T11:44:08-05:00
  [INFO] ------------------------------------------------------------------------
  ```

---

## 2. Reinicio con DevTools
* **Cambio realizado:**  
  Se modificó la propiedad `app.mensaje` en el archivo `src/main/resources/application.properties` de `"Aplicación Spring Boot en funcionamiento - ESPE"` a `"Aplicación Spring Boot actualizada con DevTools - ESPE"` y se ejecutó la recompilación del proyecto (`.\mvnw.cmd compile` / guardar archivo).

* **Evidencia del reinicio (registro de consola):**
  ```text
  2026-10-16T11:57:00.044-05:00  INFO 4228 --- [lab] [   File Watcher] rtingClassPathChangeChangedEventListener : Restarting due to 1 class path change (0 additions, 0 deletions, 1 modification)
  2026-10-16T11:57:00.082-05:00  INFO 4228 --- [lab] [tomcat-shutdown] o.s.b.w.e.tomcat.GracefulShutdown        : Graceful shutdown complete
  ...
  2026-10-16T11:57:00.233-05:00  INFO 4228 --- [lab] [  restartedMain] com.espe.lab.LabApplication              : Starting LabApplication using Java 25.0.1 with PID 4228
  2026-10-16T11:57:01.597-05:00  INFO 4228 --- [lab] [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
  2026-10-16T11:57:01.626-05:00  INFO 4228 --- [lab] [  restartedMain] com.espe.lab.LabApplication              : Started LabApplication in 1.44 seconds (process running for 713.348)
  ```

* **Respuesta antes y después (GET `http://localhost:8080/api/saludo`):**
  * **Antes:**
    ```json
    {"mensaje":"Aplicación Spring Boot en funcionamiento - ESPE","perfil":"default"}
    ```
  * **Después:**
    ```json
    {"mensaje":"Aplicación Spring Boot actualizada con DevTools - ESPE","perfil":"default"}
    ```

---

## 3. Registro de Incidencias

### Incidencia 1: Conflicto de puerto en uso (Port 8080 already in use)
* **Síntoma observado:**  
  Al intentar ejecutar una segunda instancia de la aplicación con `.\mvnw.cmd spring-boot:run` en una nueva terminal mientras la primera seguía activa, el proceso de arranque se interrumpe y la aplicación termina inmediatamente.
* **Mensaje de error:**
  ```text
  ***************************
  APPLICATION FAILED TO START
  ***************************

  Description:

  Web server failed to start. Port 8080 was already in use.

  Action:

  Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
  ```
* **Causa identificada:**  
  Dos procesos del servidor embebido Apache Tomcat intentaron escuchar simultáneamente en el mismo socket de red (`localhost:8080`), el cual ya estaba reservado por la primera instancia en ejecución.
* **Solución aplicada:**  
  Se detuvo la instancia conflictiva (o alternativamente se inició la segunda instancia utilizando el perfil `dev` mediante `.\mvnw.cmd spring-boot:run "-Dspring-boot.run.profiles=dev"` para que opere en el puerto 8081).
* **Verificación posterior:**  
  La aplicación inició correctamente mostrando `Tomcat started on port 8081` (con perfil dev) y respondió con estado `{"status":"UP"}` en `/actuator/health`.

---

## 4. Evidencias de Ejecución por Perfiles

### Perfil default (Puerto 8080):
* `GET http://localhost:8080/api/saludo`
  ```json
  {"mensaje":"Aplicación Spring Boot en funcionamiento - ESPE","perfil":"default"}
  ```
* `GET http://localhost:8080/actuator/health`
  ```json
  {"status":"UP"}
  ```
* `GET http://localhost:8080/actuator/info`
  ```json
  {"app":{"perfil":"default"}}
  ```

### Perfil dev (Puerto 8081):
* `GET http://localhost:8081/api/saludo`
  ```json
  {"mensaje":"Aplicación Spring Boot en entorno de desarrollo - ESPE","perfil":"dev"}
  ```
* `GET http://localhost:8081/actuator/health`
  ```json
  {"status":"UP"}
  ```
* Comprobación de que el puerto 8080 no responde:
  ```text
  curl: (7) Failed to connect to localhost port 8080: Connection refused
  ```

---

## 5. Uso de herramientas de inteligencia artificial
* **Herramienta utilizada:** Antigravity AI Assistant.
* **Consulta realizada:** Generación de estructura Maven para Spring Boot 3, configuración de endpoints Actuator e integración de perfiles `default` y `dev`.
* **Verificación:** Se contrastó la configuración de `management.info.env.enabled=true` con la documentación oficial de Spring Boot 3 y se verificó en tiempo de ejecución mediante peticiones cURL a `/actuator/info` y `/api/saludo`.

---

## 6. Conclusiones técnicas
1. **Configuración por perfiles:** La segregación de archivos (`application.properties` y `application-dev.properties`) permite desacoplar los parámetros de entorno (como el puerto y mensajes) del código fuente, facilitando despliegues limpios y seguros.
2. **Supervisión con Actuator:** Spring Boot Actuator expone métricas de salud (`/health`) e información (`/info`) esenciales para la observabilidad en entornos de producción y contenedores.
3. **Agilidad con DevTools:** El reinicio rápido en el hilo `restartedMain` optimiza el ciclo de desarrollo al evitar reconstruir y reiniciar la JVM manualmente en cada cambio.
