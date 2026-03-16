# SECCIÓN 1 :SMB

### 1. Vulnerabilidad o configuracion erronea encontrada

La falla principal identificada fue la configuración incorrecta del servicio NFS, 
especificamente la activación de la opción no_root_squash. 

En un entorno seguro, el servidor debería mapear cualquier solicitud proveniente 
de un usuario root remoto hacia un usuario sin privilegios (proceso conocido como root squashing). 

Al estar desactivada esta protección, el servidor permite que nuestra identidad 
de administrador en la maquina atacante fuera reconocida con autoridad total 
sobre los archivos del recurso compartido. 

Esto nos faculto para manipular propietarios y permisos especiales, 
como el bit SUID, que el sistema operativo de destino respeto integramos.

### 2. Herramientas utilizadas y justificación

Para llevar a cabo la intrusión, se emplearon las siguientes herramientas de forma sistematica:

* **Nmap:** Se utilizo en la fase de reconocimiento para detectar servicios activos, 
confirmando que el puerto 2049 estaba abierto y ejecutando el servicio NFS.

* **Showmount:** Se empleo para la enumeración de recursos compartidos, 
lo que permitió confirmar que el directorio /home era exportado de manera publica.

* **Mount:** Se utilizo para realizar el montaje del sistema de archivos remoto 
en un directorio local, permitiendo la interaccion directa con los datos almacenados en el servidor.

* **SSH:** Se utilizo como método de acceso inicial tras haber sustraido una llave privada 
id_rsa desde el recurso compartido, garantizando una conexión persistente y legitima al sistema.

### 3. Procedimiento para el acceso y obtención del dato oculto

El acceso y la posterior escalada de privilegios se lograron mediante la ejecucion 
de los siguientes pasos:

**Acceso Inicial:** Tras montar el recurso compartido NFS, se extrajeron las credenciales 
del directorio personal del usuario cappucino. Esto permitió establecer una sesion 
de usuario de bajos privilegios a traves de SSH.

**Inyeccion de binario:** Se transfirió un ejecutable de bash al sistema de archivos montado. 
Aprovechando la falta de restricciones en el servidor (no_root_squash), 
se cambio el propietario del archivo a root y se le asigno el bit SUID 
mediante el comando `chmod +s`.

**Escalada de privilegios:** Una vez en la terminal de la maquina victima, 
se ejecuto el binario modificado con el parametro -p. 
Debido a la presencia del bit SUID y a que el propietario era root, 
el proceso asumió los privilegios del administrador del sistema.

**Localizacion del dato:** Con acceso de superusuario, se procedió a inspeccionar 
el directorio /root, donde se identifico y extrajo la flag correspondiente al laboratorio.

---

# SECCIÓN 2 :SMTP

### 1. ¿Qué vulnerabilidad o configuración errónea encontramos?

Encontramos que el servidor de correo tenía una "fuga de información" en el puerto 25. 
Estaba configurado de tal manera que nos permitía preguntarle por nombres de usuarios 
y él nos confirmaba si existían o no.

Para aprovecharnos de este fallo, en Metasploit tuvimos que cargar la lista 
de nombres con este comando:
`set USER_FILE /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt`

Además, encontramos una segunda vulnerabilidad: una política de contraseñas débil. 
El usuario administrator usaba una contraseña tan sencilla que estaba 
en los diccionarios más comunes del mundo.

### 2. ¿Qué herramienta usaron y por qué?

Usamos un combo de tres herramientas porque cada una hace un trabajo diferente:

* **Nmap:** La usamos para "mapear" la casa y ver qué puertas estaban abiertas.
Comando: `nmap --min-rate 5000 -A -T4 -Pn -p- 10.64.154.188`

* **Metasploit:** La usamos para hablar con el correo (SMTP) y que nos diera 
el nombre del usuario automáticamente.
Comando: `use auxiliary/scanner/smtp/smtp_enum`

* **Hydra:** La usamos para "reventar" la puerta del SSH (puerto 22) 
probando miles de llaves por segundo.
Comando: `hydra -t 16 -l administrator -P /usr/share/wordlists/rockyou.txt 10.64.154.188 ssh`

### 3. ¿Cómo lograron el acceso o encontraron el dato oculto?

Lo logramos conectando los resultados de cada paso como si fuera una cadena:

Primero, gracias al escaneo de Nmap, supimos que el objetivo tenía servicios 
de correo y de entrada remota.

Segundo, usamos Metasploit para encontrar el "dato oculto" (el nombre de usuario). 
Al ejecutar el módulo, nos dio el premio:
Comando: `run` (dentro de smtp_enum, nos devolvió el usuario: administrator)

Tercero, con ese nombre, usamos Hydra para sacar la contraseña (alejandro).

Cuarto, con el nombre y la contraseña listos, simplemente entramos 
de forma legal por la puerta principal:
Comando: `ssh administrator@10.64.154.188`

---

# Sección 3: MYSQL

### 1. ¿Qué vulnerabilidad o configuración errónea encontramos?

La vulnerabilidad principal fue la reutilización de credenciales entre servicios 
(Credential Reuse) y la exposición de hashes de sistema en la base de datos. 

Aunque contábamos con las credenciales root:password por el escenario del laboratorio, 
el error crítico del administrador fue asignar una contraseña muy débil 
al usuario carl en el sistema operativo. 

Además, la configuración permitía al usuario root extraer los hashes de seguridad 
de otros usuarios, facilitando un ataque "fuera de línea" para comprometer 
cuentas con acceso directo al servidor mediante SSH.

### 2. ¿Qué herramienta usaron y por qué?

Usamos un conjunto de herramientas para transformar un acceso a la base de datos 
en un acceso al sistema operativo:

* **Nmap:** Fundamental para confirmar que, además de MySQL, el puerto 22 (SSH) 
estaba abierto, lo que sugería que podíamos intentar un salto de servicio.
Comando: `nmap -A -p- 10.67.147.110`

* **Metasploit (módulos de MySQL):** Los usamos para extraer la información sensible 
de las tablas internas de MySQL de forma estructurada. 
Específicamente, el módulo mysql_hashdump para obtener los secretos de los usuarios.
Comando: `use auxiliary/scanner/mysql/mysql_hashdump`

* **John the Ripper:** Fue la herramienta clave para el "cracking". 
La usamos porque los hashes de MySQL son vulnerables a ataques de diccionario 
si la contraseña no es compleja.
Comando: `john hash.txt`

### 3. ¿Cómo lograron el acceso o encontraron el dato oculto?

El acceso se logró mediante un movimiento lateral basado en la información extraída:

**Enumeración de Usuarios:** Una vez conectados a MySQL con las credenciales iniciales, 
buscamos el "dato oculto" dentro de la tabla mysql.user.

**Extracción del Hash:** Ejecutamos el volcado de hashes y detectamos al usuario carl, 
quien poseía un hash vulnerable.

**Descifrado de Contraseña:** Usamos John the Ripper para identificar que el hash 
del usuario carl correspondía a la contraseña "doggie".

**Conexión SSH:** Finalmente, validamos la reutilización de contraseñas intentando 
entrar por el servicio SSH. El éxito de la conexión nos dio acceso al sistema de archivos, 
donde leímos la bandera en el archivo MySQL.txt.
