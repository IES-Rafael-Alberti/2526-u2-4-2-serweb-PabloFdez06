# DESPLIEGUE — Evidencias y respuestas

Este documento recopila todas las evidencias y respuestas de la practica.

---

# Práctica P4.1 — Despliegue de una web estática (Nginx + SFTP + Docker)

## Objetivo
Montar un entorno con Docker Compose donde:

- Un contenedor **Nginx** sirva contenido estático.
- Un contenedor **SFTP** permita subir/actualizar esos archivos.
- Ambos compartan **el mismo volumen/carpeta**, de forma que lo que subo por SFTP se vea al momento en el navegador.
- Añadir **HTTPS** con certificado autofirmado y **redirigir HTTP → HTTPS**.

---

## Estructura del repo
- `www/` → carpeta que se compartirá entre Nginx y SFTP (aquí irá la web).
- `evidencias/` → capturas de pantalla para el checklist.
- `docker-compose.yml` → lo crearé según los requisitos.
- `default.conf` → configuración de Nginx para HTTPS + redirección.
- certificados `nginx-selfsigned.crt` y `nginx-selfsigned.key`.

---

### 1) Docker Compose con 2 servicios
**Requisitos obligatorios del enunciado**:

- Servicio Nginx con **imagen oficial**.
- Mapear **puerto 80 del contenedor → 8080 del host**.
- Servicio SFTP (por ejemplo `atmoz/sftp`).
- Mapear **puerto 22 del contenedor → 2222 del host**.
- Configurar usuario/contraseña en SFTP.
- **Volumen compartido**: ambos servicios deben apuntar a la misma carpeta/volumen.

Comprobación esperada:
- Entrando a `http://localhost:8080` se ve la web.
- Subiendo archivos por SFTP al puerto 2222, se actualizan sin reiniciar.

### 2) Subida de las dos webs por SFTP
Debo desplegar:

1) Web principal: `https://github.com/cloudacademy/static-website-example`
   - Debe verse en la raíz: `http://localhost:8080`

2) Web secundaria: `https://github.com/ArchiDep/static-clock-website`
   - Debe estar en una carpeta `reloj/` y verse en: `http://localhost:8080/reloj`

#### Cómo lo hago con FileZilla (lo que pide el checklist)

Importante (para que conecte):

- En FileZilla, la **Conexión rápida** suele intentar FTP/HTTP y no SFTP.
- Para SFTP, lo más fiable es ir a **Archivo → Gestor de sitios…** y crear un sitio nuevo.

Datos de conexión (con la configuración actual):

- Protocolo: **SFTP - SSH File Transfer Protocol**
- Host: `localhost` (si falla, probar `127.0.0.1`)
- Puerto: `2222`
- Usuario: `daw`
- Contraseña: `daw`

Si sale un error tipo “protocolo incorrecto”, revisar que NO se este usando:

- `http://localhost`
- FTP normal

Porque el servidor del contenedor escucha **SSH/SFTP**, no HTTP.

Ruta remota importante:

- Al conectar, la carpeta remota `upload/` es la que está compartida con Nginx.
- Todo lo que suba a `upload/` se verá en el navegador.

Qué carpetas locales subo (ya descargadas en este repo):

- Web principal (Cloud Academy): subir el contenido de `downloads/static-website-example/` directamente dentro de `upload/`.
   - Resultado esperado: al abrir `http://localhost:8080` (o `https://localhost:8443`) se ve la web.
- Web secundaria (Reloj): crear dentro de `upload/` una carpeta `reloj/` y subir dentro el contenido de `downloads/static-clock-website/`.
   - Resultado esperado: `http://localhost:8080/reloj` (o `https://localhost:8443/reloj`).

### 3) HTTPS + redirección
Tengo que:

- Generar certificado autofirmado (`.crt` y `.key`) en el host.
- Montarlo dentro del contenedor Nginx.
- Montar un `default.conf` propio en Nginx.
- Configurar dos `server`:
  - Uno en 80 que redirija a 443.
  - Otro en 443 que sirva el contenido.

---

## Checklist de evaluación + dónde pongo cada captura


### Fase 1: Instalación y Configuración

1. ✅ Servicio Nginx activo
   - Captura requerida: salida de `docker compose ps` mostrando el servicio activo (o equivalente).
   ![01-nginx-activo](evidencias/01-nginx-activo.png)

2. Configuración cargada
   - Captura requerida: listado de `/etc/nginx/conf.d/` (o ruta equivalente) donde se vea tu `.conf`.
   ![02-conf-cargada](evidencias/02-conf-cargada.png)

3. Resolución de nombres (hosts)
   - Captura requerida: navegador mostrando `http://nombre_web` (no IP) y página cargada.
   ![03-hosts](evidencias/03-hosts.png)

4. ✅ Contenido Web (Cloud Academy)
   - Captura requerida: se ve la web importada, no la página por defecto de Nginx.
   ![04-cloudacademy](evidencias/04-cloudacademy.png)

### Fase 2: Transferencia SFTP (Filezilla)

5. ✅ Conexión SFTP exitosa
   - Captura requerida: Filezilla con “Connected to …” y listado remoto.
   ![05-sftp-conectado](evidencias/05-conexion_exitosa.png)

6. ✅ Permisos de escritura (sin Permission denied)
   - Captura requerida: transferencia completada o archivos visibles en remoto.
   ![06-sftp-permisos](evidencias/06-sftp-permisos.png)

### Fase 3: Infraestructura Docker

7. ✅ Contenedores activos (Nginx + SFTP)
   - Captura requerida: `docker compose ps` con ambos Up y puertos `8080->80` y `2222->22`.
   ![07-compose-ps](evidencias/07-contenedores_activos.png)

8. ✅ Persistencia (lo que subo por SFTP se ve en web)
   - Captura requerida: evidencia cruzada (Filezilla + navegador) mostrando los mismos archivos.
   ![08-persistencia](evidencias/08-persistencia.png)

9. ✅ Multi-sitio (reloj en subcarpeta)
   - Captura requerida: navegador en `http://localhost:8080/reloj` con el reloj funcionando.
   ![09-reloj](evidencias/09-reloj.png)

### Fase 4: Seguridad HTTPS

10. ✅ Cifrado SSL
   - Captura requerida: navegador en `https://…` mostrando candado o aviso de certificado y el puerto usado (ej. 8443).
   ![10-https](evidencias/10-conexion_https.png)

11. ✅ Redirección forzada HTTP → HTTPS
   - Captura requerida: pestaña Network (F12) mostrando `301 Moved Permanently` al entrar por HTTP.
   ![11-redirect-301](evidencias/11-redireccion_forzada.png)

---

## Comentarios de errores y aprendizaje durante la practica
- Nginx suele servir desde `/usr/share/nginx/html` (depende de imagen/config).
- En `atmoz/sftp`, normalmente se sube a una carpeta tipo `/home/<usuario>/upload`.
- Si hay “Permission denied”, tengo que revisar:
  - la ruta exacta del volumen compartido en ambos contenedores
  - permisos del directorio en el host
  - si el contenedor SFTP está escribiendo en el mismo sitio que Nginx lee

  Trabajo originalmente realizado en este repositorio: https://github.com/IES-Rafael-Alberti/Web_Estatica_PFF.git

## Parte 2 — Evaluacion RA2 (a–j)

### a) Parametros de administracion
- Respuesta:
- Evidencias:
  - evidencias/a-01-grep-nginxconf.png
  - evidencias/a-02-nginx-t.png
  - evidencias/a-03-reload.png

### b) Ampliacion de funcionalidad + modulo investigado
- Opcion elegida (B1 o B2):
- Respuesta:
- Evidencias (B1 o B2):
  - evidencias/b1-01-gzipconf.png
  - evidencias/b1-02-compose-volume-gzip.png
  - evidencias/b1-03-nginx-t.png
  - evidencias/b1-04-curl-gzip.png
  - evidencias/b2-01-defaultconf-headers.png
  - evidencias/b2-02-nginx-t.png
  - evidencias/b2-03-curl-https-headers.png

#### Modulo investigado: <NOMBRE>
- Para que sirve:
- Como se instala/carga:
- Fuente(s):

### c) Sitios virtuales / multi-sitio
- Respuesta:
- Evidencias:
  - evidencias/c-01-root.png
  - evidencias/c-02-reloj.png
  - evidencias/c-03-defaultconf-inside.png

### d) Autenticacion y control de acceso
- Respuesta:
- Evidencias:
  - evidencias/d-01-admin-html.png
  - evidencias/d-02-defaultconf-auth.png
  - evidencias/d-03-curl-401.png
  - evidencias/d-04-curl-200.png

### e) Certificados digitales
- Respuesta:
- Evidencias:
  - evidencias/e-01-ls-certs.png
  - evidencias/e-02-compose-certs.png
  - evidencias/e-03-defaultconf-ssl.png

### f) Comunicaciones seguras
- Respuesta:
- Evidencias:
  - evidencias/f-01-https.png
  - evidencias/f-02-301-network.png

### g) Documentacion
- Respuesta:
- Evidencias: enlaces a todas las capturas

### h) Ajustes para implantacion de apps
- Respuesta:
- Evidencias:
  - evidencias/h-01-root.png
  - evidencias/h-02-reloj.png

### i) Virtualizacion en despliegue
- Respuesta:
- Evidencias:
  - evidencias/i-01-compose-ps.png

### j) Logs: monitorizacion y analisis
- Respuesta:
- Evidencias:
  - evidencias/j-01-logs-follow.png
  - evidencias/j-02-metricas.png

---

## Checklist final

### Parte 1
- [ ] 1) Servicio Nginx activo
- [ ] 2) Configuracion cargada
- [ ] 3) Resolucion de nombres
- [ ] 4) Contenido Web (Cloud Academy)
- [ ] 5) Conexion SFTP exitosa
- [ ] 6) Permisos de escritura
- [ ] 7) Contenedores activos
- [ ] 8) Persistencia (Volumen compartido)
- [ ] 9) Despliegue multi-sitio (/reloj)
- [ ] 10) Cifrado SSL
- [ ] 11) Redireccion forzada (301)

### Parte 2 (RA2)
- [ ] a) Parametros de administracion
- [ ] b) Ampliacion de funcionalidad + modulo investigado
- [ ] c) Sitios virtuales / multi-sitio
- [ ] d) Autenticacion y control de acceso
- [ ] e) Certificados digitales
- [ ] f) Comunicaciones seguras
- [ ] g) Documentacion
- [ ] h) Ajustes para implantacion de apps
- [ ] i) Virtualizacion en despliegue
- [ ] j) Logs: monitorizacion y analisis
