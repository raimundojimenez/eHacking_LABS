# Enumeración NFS.

Requisitos:
1. Máquina ***Router-Ubu***.
2. Máquina ***Kali Linux***.
3. Máquina ***Ubu_srv_01***.


***NFS*** (Network File System) es el servicio de compartición de archivos característico de entornos Linux (Windows Server también puede ofrecerlo).

Las versiones más recientes presentan menos vulnerabilidades que las más antiguas, pero eso no nos interesa aún. En la mayoría de las ocasiones, la información se fuga porque el administrador ha realizado una implementación de NFS errónea, que permite ser atacada.

En este laboratorio vamos a instalar ***NFS*** y aprender a enumerarlo desde la máquina de ataque.

## Ejercicio 1: Configuración de un share de NFS.

Iniciamos sesión la máquina ***ubu_srv_01*** con el usuario
```
antonio
```

y el password
```
Pa55w.rd
```

Observarás que es una versión sin interfaz gráfica. No hay soporte de portapapeles, por esta razón tendrás que copiar los comandos directamente. Como alternativa, puedes hacer un **SSH** desde **Kali*** para disponer de interfaz gráfica.

Procedemos a instalar ***NFS***. En la terminal, escribimos.
```
sudo apt update
```

```
sudo apt install -y nfs-kernel-server
```

El archivo ***/etc/exports*** mantiene un registro por cada directorio que se va a compartir en la red. Existen diferentes opciones que definirán el tipo de privilegio que tendrán los clientes sobre cada ***share***.

* *rw*: Permiso de lectura y escritura.
* *ro*: Permiso de solo lectura.
* *root_squash*: Previene peticiones de archivo hechas por el usuario ***root*** en la máquina cliente.
* *no_root_squash*: Permite al usuario ***root*** de la máquina cliente acceder al share.
* *async*: Mejora la velocidad de transferencia pero puede conducir a corrupción de los datos.
* *sync*: Garantiza la integridad de los datos a expensas de la velocidad de transferencia.

En bastantes ocasiones, los administradores crean ***shares*** que no requieren autenticación, por diversas razones:

* No poseen los conocimientos para habilitar la autenticación.
* La información que se comparte no es sensible.
* Es una necesidad temporal y el share se retirará en breve.
* Las aplicaciones que acceden al share no pueden usar autenticación.
* Etc.

Vamos a crear un ***Share*** en el que cometeremos un error muy grave de seguridad. ***NFS*** se puede usar también para hacer las migraciones, y por consiguiente, suele ser muy cómodo compartir el directorio ***home*** del usuario en cuestión.

Primero creamos una carpeta en la ruta ***/datos*** a la que daremos permisos de lectura y escritura a todos los usuarios de dicha máquina. Posteriormente compartiremos esa carpeta por NFS para que se pueda acceder desde la red.
```
sudo mkdir -p /datos
```
```
sudo chmod 777 /datos
```

Procedemos a crear el share. Para ello editamos el archivo ***/etc/exports***.
```
sudo nano /etc/exports
```

y añadimos una nueva línea, al final del archivo con el contenido siguiente, tal y como muestra la imagen.
```
/datos *(rw,no_root_squash)
``` 

![Crear share](../img/lab-04-D/202209101205.png)

Guardamos con ***CTRL+X***, ***Y*** y ***ENTER***.

Lo que hemos hecho es compartir el directorio ***/datos***, permitiendo acceder al usuario ***root*** de los clientes, en forma de ***lectura*** y ***escritura***. El '*' indica que la conexión se puede hacer desde cualquier ***IP***.

Reiniciamos el servicion ***NFS***.
```
sudo service nfs-kernel-server restart
```

## Ejercicio 2: Enumerar los shares con nmap.

Lo primero que debemos hacer es localizar los servidores ***NFS*** de la red. Para ello debemos saber que el puerto de servicio de ***NFS*** es el ***2049***.

En la máquina ***Kali***, ejecutamos el siguiente comando.
```
nmap -sV -p 2049 192.168.20.60
```

Como puede observarse en la siguiente imagen, en la IP ***192.168.20.60*** existe un servidor ***NFS*** porque el puerto está ***open***.

![NFS open](../img/lab-04-D/202209101359.png)

Lo primero que va a hacer el actor de la amenaza es ***enumerar*** las shares de ese servidor nfs. Si es afortunado encontrará alguna que no requiera autenticación.

Nota: Los ***exports*** se exponen en el puerto ***111***.
```
nmap -sV -p 111 --script=nfs-showmount 192.168.20.60
```

La imagen muestra como aparece listado el share que hemos creado anteriormente.

![NFS Share](../img/lab-04-D/202209101503.png)

Con esto concluye la práctica de enumeración ***NFS***. Es muy importante recordar que una mala práctica por parte del administrador puede conducir al robo de información, e incluso a tomar el control del sistema.

Como adelanto a la parte del curso centrada en los ataques, te recomiendo que hagas el siguiente laboratorio: ***4. Laboratorio 30-D:  Escalado de privilegio explotando no_root_squash***.

***FIN DEL LABORATORIO***