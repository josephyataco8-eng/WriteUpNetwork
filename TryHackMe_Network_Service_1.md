# SECCIÓN 1: PROTOCOLO SMB

### ¿Qué vulnerabilidad o configuración errónea encontraron? 

Durante la fase de enumeración se identificaron varios puertos abiertos en el host objetivo. 

Entre ellos, los puertos 139 y 445, que corresponden al protocolo Server Message Block (SMB). 

Tras analizar el servicio, se descubrió una mala configuración en los recursos compartidos, 
específicamente en el share llamado "profiles", el cual permitía acceso sin autenticación adecuada. 

Esta configuración permitio listar y acceder al contenido del recurso compartido, 
lo que expuso información sensible del sistema. 

Esta debilidad facilitó el acceso a archivos privados de usuarios, 
incluyendo claves SSH que posteriormente permitieron obtener acceso al sistema.

### ¿Qué herramienta usaron y por qué?

* **Nmap:** Se utilizó para realizar el escaneo de puertos del sistema objetivo y detectar los servicios activos. 
Usamos el escaneo SYN (-sS) para descubrir puertos de forma rápida y la detección de versiones (-sV) 
para saber qué programas estaban corriendo exactamente.

* **Enum4linux:** Se utilizó para realizar una enumeración específica de SMB. 
Con esta herramienta pudimos extraer nombres de usuarios y recursos compartidos. 
Gracias a esto descubrimos que el recurso "profiles" permitía acceso sin contraseña.

* **Smbclient:** Esta herramienta se usó para conectarnos directamente al recurso compartido vulnerable. 
Fue útil para navegar por las carpetas como si estuviéramos en el sistema local y 
descargar los archivos encontrados.

### ¿Cómo lograron el acceso o encontraron el dato oculto? 

El paso clave fue la enumeración con Enum4linux, ya que reveló que el recurso compartido 
"profiles" tenía el listado de archivos abierto. 

Esto indicaba que el recurso podía ser montado sin credenciales. 

Una vez dentro con smbclient, exploramos los directorios y encontramos una carpeta oculta llamada .ssh. 
Dentro de ella estaban las claves de autenticación del usuario. 

Descargamos la clave id_rsa, le dimos los permisos correctos en nuestra máquina 
y la usamos para entrar por SSH al servidor, donde finalmente leímos la bandera.

---

# SECCIÓN 2: PROTOCOLO TELNET

### 1. ¿Qué vulnerabilidad o configuración errónea encontraron? 

Durante el análisis, identificamos una vulnerabilidad de Ejecución Remota de Comandos (RCE) 
en un servicio que corría en el puerto 8012. 

El problema principal es que este servicio se comporta como una puerta trasera 
que no pide ningún tipo de contraseña. 

Esto permite que cualquier persona que se conecte pueda enviar órdenes 
directamente a la consola del servidor.

### 2. ¿Qué herramienta usaron y por qué?

* **Nmap:** La usamos para el descubrimiento inicial y saber que el puerto 8012 estaba abierto.

* **Netcat y Telnet:** Estas herramientas las usamos para conectarnos manualmente al puerto 
y ver qué mensajes devolvía el servidor. Son útiles porque permiten interactuar directamente con el servicio.

* **Tcpdump:** Esta herramienta fue necesaria para confirmar que los comandos se estaban ejecutando. 
Como el servicio no nos mostraba el resultado de lo que escribíamos, usamos tcpdump 
para ver si el servidor respondía a nuestras peticiones de ping.

* **Msfvenom:** La usamos para crear el código de la "shell inversa". 
Es una herramienta que facilita la creación de comandos complejos que sirven 
para que el servidor se conecte de vuelta a nuestra computadora.

### 3. ¿Cómo lograron el acceso o encontraron el dato oculto? 

Logramos el acceso estableciendo una conexión inversa. 

Primero, pusimos nuestra computadora a esperar una conexión en el puerto 4444 usando Netcat. 

Luego, a través de la conexión de Telnet en el puerto 8012, ejecutamos el código 
que habíamos preparado con msfvenom. 

El servidor ejecutó la orden y nos devolvió una terminal. 

Con esa terminal ya pudimos movernos por las carpetas del sistema 
y encontrar el archivo con la bandera.

---

# SECCIÓN 3: PROTOCOLO FTP

### 1. ¿Qué vulnerabilidad o configuración errónea encontraron? 

Encontramos dos fallos de seguridad en el servicio FTP:

* **Acceso anónimo:** El servidor permitía entrar usando el usuario "anonymous" sin contraseña. 
Esto nos dejo ver archivos que no deberían ser públicos.

* **Contraseñas débiles:** Uno de los usuarios del sistema, llamado "mike", 
usaba una contraseña muy común que estaba en diccionarios de seguridad, 
lo que permitió adivinarla fácilmente.

### 2. ¿Qué herramienta usaron y por qué?

* **Nmap:** Se usó para encontrar el puerto 21 y para ver que el servicio permitía 
el acceso de usuarios anónimos mediante sus scripts automáticos.

* **Cliente FTP:** Se usó para entrar al servidor, navegar por las carpetas 
y bajar un archivo de texto que contenía nombres de posibles usuarios.

* **Hydra:** Esta herramienta se eligió para adivinar la contraseña de Mike. 
Es muy efectiva porque permite probar miles de combinaciones de forma automática 
hasta encontrar la correcta.

### 3. ¿Cómo lograron el acceso o encontraron el dato oculto? 

El acceso se logró en varios pasos. Primero entramos como anónimos 
y descargamos un archivo donde leímos el nombre de "mike". 

Después, usamos Hydra junto con una lista de contraseñas llamada rockyou.txt 
para encontrar que su clave era "password". 

Con esos datos, volvimos a entrar por FTP como el usuario Mike. 

Al tener sus permisos, pudimos entrar a su carpeta personal 
y leer el archivo de texto que guardaba la bandera del ejercicio.
