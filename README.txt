Práctica 1: Conectar servidores MCP a un agente

1. Descripción
Esta práctica consistió en configurar y utilizar servidores del Model Context Protocol (MCP) para dotar de herramientas a un agente de programación. En mi caso, utilicé Google Antigravity CLI (versión 1.5.16) con el modelo Gemini. El objetivo fue resolver tareas combinando las capacidades de lectura/escritura de archivos locales y la navegación web automatizada.

2. Requisitos y reproducción
Para reproducir esta práctica se necesita:
* Node.js 18 o superior.
* Google Chrome instalado (usado por Playwright).
* Git y una cuenta de GitHub.
* Antigravity instalado.

Pasos para reproducir:
1. Clona este repositorio y abre la carpeta en la terminal.
2. Crea la carpeta .agents en la raíz del proyecto.
3. Crea el archivo .agents/mcp_config.json con la configuración de los servidores.
Importante: Debes cambiar la ruta absoluta en el archivo por la ruta donde tengas clonado este repositorio en tu equipo.
4. Ejecuta agy en la terminal y verifica que los servidores filesystem y playwright aparezcan conectados usando el comando /mcp.

3. Servidores MCP
Se configuraron dos servidores locales ejecutados vía npx:
	filesystem (@modelcontextprotocol/server-filesystem): Proporciona herramientas para listar directorios, leer y escribir archivos (ej. list_directory, read_text_file, write_file).
	playwright (@playwright/mcp): Proporciona herramientas para controlar un navegador web, navegar, evaluar JavaScript en la página y tomar capturas (ej. browser_navigate, browser_evaluate, browser_take_screenshot).

![Servidores conectados](evidencias/01_servidores_conectados.png)

4. Actividades

Actividad A: Lectura y escritura de archivos
Instrucción dada: Usa únicamente las herramientas del servidor MCP filesystem. Lista el contenido de la carpeta datos, lee el archivo datos/calificaciones.csv, calcula el promedio de cada alumno (media de tres parciales, dos decimales) indicando si aprueba (>= 6.0), y escribe el resultado ordenado de mayor a menor en `salida/reporte.md`, agregando el promedio del grupo.

Secuencia de herramientas usadas:
1. list_allowed_directories (para conocer la ruta permitida).
2. list_directory (para ver el contenido de la carpeta datos).
3. read_text_file (para leer calificaciones.csv).
4. write_file (para crear salida/reporte.md con la tabla final).

Resultado: Se generó el archivo correctamente con 8 alumnos, 6 aprobados, 2 reprobados y un promedio grupal de 7.50. El agente tecleó el directorio de salida previamente usando create_directory.

![Permiso Actividad A](evidencias/02_actividad_a_permiso.png)
![Llamadas Actividad A](evidencias/03_actividad_a_secuencia.png)
![Reporte final](evidencias/04_actividad_a_reporte.png)

Actividad B: De una página web a un CSV
Instrucción dada: Usar herramientas de playwright y filesystem. Abrir la página de Wikipedia de las entidades federativas de México, localizar la tabla principal (32 entidades), extraer entidad, superficie, población y densidad. Guardar en salida/entidades.csv, tomar captura de pantalla y cerrar el navegador.

Secuencia de herramientas usadas:
1. browser_navigate (para abrir la URL).
2. browser_evaluate (para extraer los datos de la tabla usando JavaScript).
3. filesystem/write_file (para guardar los datos extraídos en el CSV).
4. browser_take_screenshot (para guardar la imagen de la página).
5. browser_close (para cerrar el navegador).

Verificación manual: Revisé tres filas del CSV generado (salida/entidades.csv) contra la página web:
* Chihuahua: 247455 km2 (Correcto).
* Sonora: 179355 km2 (Correcto).
* Ciudad de México: 1495 km2 (Correcto).
El año de población que usa la tabla según el agente es 2020.

Cambio de servidor: El agente utilizó el servidor playwright (ejecutando JavaScript con browser_evaluate) para obtener los datos de la tabla, y luego cambió al servidor filesystem (con write_file) para guardar esa información extraída en el archivo local CSV.

![Llamadas Actividad B](evidencias/07_actividad_b_llamadas.png)
![Archivo CSV](evidencias/08_actividad_b_csv.png)
![Captura de la página](evidencias/09_actividad_b_screenshot.png)

Actividad C: Los límites del servidor
Instrucción dada: Usar únicamente las herramientas del servidor MCP filesystem. Llamar a `list_allowed_directories`, intentar leer `C:/Windows/win.ini` e intentar listar el directorio padre, reportando textualmente la respuesta del servidor.

Resultados obtenidos:
1. list_allowed_directories: Confirmó que el único directorio permitido es la carpeta de la práctica.
2. read_text_file (C:/Windows/win.ini): *Error:* Access denied - path outside allowed directories: C:\Windows\win.ini not in C:\Users\Admin\Documents\Universidad\Ago - Dic 2026\Desarrollo de Agentes Inteligentes\Unidad 3\practica1-mcp
3. list_directory (directorio padre): *Error:* Access denied - path outside allowed directories: C:\Users\Admin\Documents\Universidad\Ago - Dic 2026\Desarrollo de Agentes Inteligentes\Unidad 3 not in C:\Users\Admin\Documents\Universidad\Ago - Dic 2026\Desarrollo de Agentes Inteligentes\Unidad 3\practica1-mcp

¿Quién impuso el límite?
El límite lo impuso el servidor MCP (`server-filesystem`). El modelo (el agente) intentó hacer las llamadas (incluso envió la ruta correcta en los argumentos), pero el servidor, por razones de seguridad, validó la ruta contra los directorios permitidos que se le pasaron al arrancar (el parámetro "$PWD" en la configuración) y rechazó la petición antes de intentar leer o listar. Ni el modelo ni el host (Antigravity) bloquearon la acción directamente, fue el propio programa del servidor el que devolvió el mensaje de error.

![Límites del servidor](evidencias/10_actividad_c_limites.png)

Actividad D: Un tercer servidor (Memory)
Servidor elegido: @modelcontextprotocol/server-memory.
Quién lo publica: Es un servidor oficial mantenido por el equipo de Model Context Protocol.
Herramientas que expone: Permite crear entidades, agregar observaciones y relaciones en un grafo de conocimiento persistente (ej. create_entities, read_graph, search_nodes).
Tarea realizada: Le pedí al agente que creara una entidad con mi nombre, le asignara el tipo "estudiante" y guardara la observación de que estoy realizando esta práctica. Luego el agente leyó la entidad para confirmar que la memoria se guardó.
Justificación de confianza: Considero confiable este servidor porque está publicado en el repositorio oficial de Model Context Protocol en GitHub, lo que garantiza que es mantenido por los mismos creadores del protocolo. Además, se ejecuta de forma local (stdio) y no requiere enviar mis datos a ningún servicio externo.

![Servidores MCP con Memory](evidencias/12_actividad_d_servidores.png)
![Llamadas a Memory](evidencias/14_actividad_d_llamadas.png)
![Resumen Memory](evidencias/15_actividad_d_resumen.png)

5. Problemas encontrados
* Problema: Al inicio se intentó configurar Claude Code, pero la cuenta no contaba con una suscripción activa para usar el CLI.
**Solución: Se optó por la Ruta B de la práctica, utilizando Google Antigravity, el cual funciona correctamente con una cuenta personal de Google.
*Problema: Configuración inicial de los servidores MCP en Antigravity.
**Solución: En lugar de usar la interfaz, se creó manualmente el archivo .agents/mcp_config.json en el directorio del proyecto, asegurando el uso de barras dobles (`\\`) en las rutas de Windows para que el JSON fuera válido.
*Problema: El servidor memory no aparecía en la lista de MCP a pesar de estar en el archivo JSON.
**Solución: Se copió la configuración al archivo global (`~/.gemini/config/mcp_config.json`) y se reinició la terminal. Antigravity reconoció el servidor correctamente.

6. Fuentes
* Wikipedia: [Anexo:Entidades federativas de México por superficie, población y densidad](https://es.wikipedia.org/?curid=5897912)
* Documentación oficial de Model Context Protocol (MCP) y Google Antigravity.
