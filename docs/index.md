# Práctica: Configuración de Servidor Nginx como Proxy Inverso

## Introducción

Un **proxy** es un servidor que actúa como intermediario entre los clientes y los servidores web. Existen dos tipos principales:
- **Proxy de reenvío**: intercepta las solicitudes de los clientes antes de enviarlas a los servidores de origen.
- **Proxy inverso**: intercepta las solicitudes entrantes y las redirige a los servidores web, mejorando seguridad, balanceo de carga y rendimiento.

En esta práctica configuraremos un **servidor proxy inverso** con Nginx para redirigir las solicitudes hacia otro servidor Nginx que actúa como servidor web. Utilizaremos dos máquinas Debian virtuales: una como proxy inverso y la otra como servidor web.

## Configuración inicial

1. **Clonación de la máquina virtual**:
    - Clonamos la máquina virtual Debian ya configurada con Nginx (de la práctica anterior).
    - Durante la clonación, asegurarse de generar una nueva dirección MAC para evitar conflictos de red.

## Servidor Web (Webserver)

1. **Renombrar la configuración del servidor web**:
    - Cambiar el nombre del archivo de configuración de Nginx a `webserver`.

    ```bash
    sudo mv /etc/nginx/sites-available/Practica2Daw /etc/nginx/sites-available/webserver
    ```

    - Dentro del archivo, actualizar las referencias al nombre del sitio web.

    ```bash
    sudo nano /etc/nginx/sites-available/webserver
    ```

2. **Modificar el puerto del servidor web**:
    - Cambiar el puerto de escucha de 80 a 8080 en el archivo de configuración.

    ```nginx
    server {
        listen 8080;
        server_name webserver;
        root /var/www/webserver;
        index index.html;
    }
    ```

3. **Eliminar el enlace simbólico anterior y crear uno nuevo**:

    ```bash
    sudo unlink /etc/nginx/sites-enabled/Practica2Daw
    sudo ln -s /etc/nginx/sites-available/webserver /etc/nginx/sites-enabled/
    ```

4. **Reiniciar Nginx**:

    ```bash
    sudo systemctl restart nginx
    ```

## Servidor Proxy Inverso (Proxy)

1. **Crear un archivo de configuración para el proxy inverso**:
    - Crear un nuevo archivo de configuración en `/etc/nginx/sites-available/` llamado `Practica2Daw`.

    ```bash
    sudo nano /etc/nginx/sites-available/Practica2Daw
    ```

    ![comando1_edicionPractica2Daw](assets/images/Captura%20de%20pantalla%202024-10-18%20185509.png)

2. **Configuración del proxy inverso**:
    - Dentro del archivo, configurar el proxy para redirigir las solicitudes al servidor web (`webserver:8080`).

    ```nginx
    server {
        listen 80;
        server_name Practica2Daw;
        location / {
            proxy_pass http://webserver:8080;
        }
    }
    ```

    ![comando2_editor_proxy](assets/images/Captura%20de%20pantalla%202024-10-24%20222205.png)

3. **Crear un enlace simbólico para activar el proxy**:

    ```bash
    sudo ln -s /etc/nginx/sites-available/Practica2Daw /etc/nginx/sites-enabled/
    ```

    ![comando3_vincular](assets/images/Captura%20de%20pantalla%202024-10-18%20190422.png)

4. **Reiniciar Nginx en el servidor proxy**:

    ```bash
    sudo systemctl restart nginx
    ```

## Modificación del archivo hosts

- Para asociar el nombre de dominio del proxy con su dirección IP, modificar el archivo `/etc/hosts` en la máquina anfitriona:

    ```bash
    sudo nano /etc/hosts
    ```

    Añadir la siguiente línea, reemplazando `IP_DEL_PROXY` por la dirección IP real:

    ```bash
    IP_DEL_PROXY proxy
    ```

## Comprobaciones

1. **Comprobación en el navegador**:
    - Acceder a `http://proxy` desde el navegador. Si todo está configurado correctamente, se debería redirigir al servidor web en `webserver:8080`.

2. **Verificación de logs**:
    - Comprobar que las peticiones llegan tanto al servidor proxy como al servidor web revisando los archivos de logs:

    ```bash
    sudo tail -f /var/log/nginx/access.log
    ```

## Añadir cabeceras

1. **Configurar cabeceras en el proxy**:
    - En el archivo de configuración del proxy (`proxy`), agregar una cabecera personalizada.

    ```nginx
    server {
        listen 80;
        server_name Practica2Daw;
        location / {
            proxy_pass http://webserver:8080;
            add_header Host Practica2Daw;
        }
    }
    ```

2. **Configurar cabeceras en el servidor web**:
    - En el archivo de configuración del servidor web (`webserver`), agregar una cabecera personalizada:

    ```nginx
    server {
        listen 8080;
        server_name webserver;
        root /var/www/webserver;
        index index.html;
        location / {
            add_header Host Servidor_web_tunombre;
        }
    }
    ```

3. **Reiniciar Nginx en ambos servidores**:

    ```bash
    sudo systemctl restart nginx
    ```

4. **Comprobación de cabeceras**:
    - Utilizando las herramientas de desarrollo del navegador (F12 > Red), verificar que las cabeceras personalizadas están presentes en las respuestas del proxy inverso y del servidor web.

## Conclusión

Con esta configuración, se ha implementado un servidor Nginx como proxy inverso que redirige las solicitudes a otro servidor web Nginx. Se ha demostrado el paso de las solicitudes mediante logs y cabeceras personalizadas, garantizando que las peticiones pasan por el proxy antes de llegar al servidor web final.

