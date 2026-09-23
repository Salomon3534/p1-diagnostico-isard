# P1 - Publica y diagnostica una web en Isard

## Datos de mi VM

- Usuario: ikasle
- Hostname: daw-lubuntu-base
- IP de la VM: 192.168.123.201
- Puerto que he utilizado: 8000

## Esquema

PC del aula -> red de Isard -> VM (192.168.123.201) -> puerto 8000 -> proceso python3 -> archivos (index.html, productos.html, css/estilos.css)

En mi VM he arrancado un servidor de Python con `python3 -m http.server 8000`. Ese proceso escucha en el puerto 8000 y sirve los archivos de la carpeta `catalogo-web`.

## Diferencia entre 127.0.0.1, 0.0.0.0 e IP real

- **127.0.0.1**: es la dirección de la propia VM hablando consigo misma. Si el servidor escucha solo aquí, únicamente se puede entrar desde dentro de la VM. Lo comprobé porque con la IP real no me funcionaba.
- **0.0.0.0**: significa "escucha por todas las entradas de la VM". La usé al arrancar el servidor para que aceptara conexiones por cualquier dirección. Con `ss` me salía como `0.0.0.0:8000`.
- **192.168.123.201**: es la dirección real de mi VM en la red. Es la que usaría otro equipo para conectarse a ella.

## Tabla de diagnóstico

| Caso | Qué observé | Qué falla | Causa comprobada y solución |
|---|---|---|---|
| A Puerto 8080 | Conexión rechazada | Puerto | No hay ningún programa escuchando en el 8080. Uso el 8000 |
| B producto.html | Error 404 | Nombre del archivo | El archivo se llama productos.html, no producto.html. Lo escribo bien |
| C Servidor detenido | Conexión rechazada | Proceso | Al parar el servidor con Ctrl+C no queda nada escuchando. Lo vuelvo a arrancar |
| D IP real con escucha local | Conexión rechazada | Dirección de escucha | El servidor solo escuchaba en 127.0.0.1. Lo arranco con 0.0.0.0 |
| E Acceso desde el PC | No he podido conectar | Red | La red de Isard tiene la VM aislada. No es un fallo del servidor |
| F Hoja de estilos | Error 404 en css/style.css | Nombre del archivo | index.html pedía style.css pero el archivo se llama estilos.css. Lo corregí en la rama fix/css |

## Diferencia entre conexión rechazada y error 404

- **Conexión rechazada**: en esa dirección y puerto no hay ningún programa escuchando.
- **Error 404**: el servidor sí está funcionando y me contesta, pero me dice que el archivo que pido no existe.

## Resultado del acceso desde el PC del aula

No he podido acceder desde el PC del aula, porque la red de Isard tiene las VM aisladas. Dentro de la VM la web funciona bien: el servidor escucha en 0.0.0.0:8000 y responde con código 200.

## Salida de git log

```
* 09d56e7 (HEAD -> main, origin/main, fix/css) fix: css route
* 1642863 feat: crea catalogo web inicial
```

## Conclusión individual

### Qué comprobaciones haría y en qué orden si una web no se abre

Iría de lo más básico a lo más concreto, para descartar cosas por el camino:

1. Primero comprobaría que el nombre existe (`getent hosts`), porque si el nombre no se traduce a una IP, el navegador ni sabe a dónde ir.
2. Después miraría que la máquina tiene IP y que llego a ella (`ip -br address` y `ping`). Si no llego, el problema es de red y no tiene sentido seguir mirando el servidor.
3. Luego comprobaría que el servidor está arrancado y en qué dirección y puerto escucha (`ss -ltnp`). Aquí veo si hay algo en el puerto y si escucha en 127.0.0.1 o en 0.0.0.0.
4. Después probaría con `curl` desde dentro de la VM, primero a 127.0.0.1 y luego a la IP real. Si funciona con la primera y no con la segunda, el fallo está en la dirección de escucha o en la red, no en la web.
5. Por último miraría el código que devuelve el servidor. Un 404 me dice que el archivo o el nombre están mal, y si la página sale pero sin estilos, miro que los archivos que enlaza (como el CSS) existan.

Sigo este orden porque cada paso descarta un problema y me dice si seguir hacia delante o parar y arreglar ahí.

### Por qué un código 404 demuestra que parte de la infraestructura funciona

Para que me llegue un 404 han tenido que funcionar muchas cosas: el nombre o la IP se han resuelto bien, la red ha llegado hasta la VM, el puerto estaba abierto, el programa estaba arrancado y escuchando, y el servidor web ha entendido mi petición y me ha contestado. Lo único que falla es que el archivo que pedí no existe. Por eso un 404 me ayuda a descartar los problemas de red y de servidor, y me indica que tengo que revisar el nombre o la ruta del recurso.
