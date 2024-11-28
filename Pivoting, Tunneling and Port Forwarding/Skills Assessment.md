# Escenario

>Un miembro del equipo inició una prueba de penetración contra el entorno de Inlanefreight, pero fue trasladado a otro proyecto en el último minuto. Afortunadamente para nosotros, dejaron una **`web shell`** en su lugar para que podamos volver a ingresar a la red y poder continuar donde ellos lo dejaron. Necesitamos aprovechar el web shell para continuar enumerando los hosts, identificando servicios comunes y usando esos servicios/protocolos para pivotar hacia las redes internas de Inlanefreight. Nuestros objetivos detallados se encuentran a continuación:
## Objetivos

- Comience desde lo externo (Pwnbox o su propia máquina virtual) y acceda al primer sistema a través del shell web que quedó en su lugar.
- Use el acceso al shell web para enumerar y pivotar hacia un host interno.
- Continúe con la enumeración y el pivoteo hasta llegar al controlador de dominio de Inlanefreight y capture la bandera asociada.
- Use cualquier dato, credencial, script u otra información dentro del entorno para habilitar sus intentos de pivoteo.
- Tome todas las banderas que pueda encontrar.
### NOTA

>Tenga en cuenta las herramientas y tácticas que practicó a lo largo de este módulo. Cada uno puede proporcionar una ruta diferente hacia el siguiente punto de pivote. Puede que te resulte sencillo realizar un salto desde un grupo de hosts, pero es posible que esa misma táctica no funcione para llegar al siguiente. Mientras completas esta evaluación de habilidades, te recomendamos que tomes notas adecuadas, dibujes un mapa de lo que ya sabes y planifiques tu próximo salto. Intentar hacerlo sobre la marcha resultará difícil sin tener un elemento visual de referencia.


## Preguntas:

1. **Once on the webserver, enumerate the host for credentials that can be used to start a pivot or tunnel to another host in the network. In what user's directory can you find the credentials? Submit the name of the user as the answer.**
   
	- [ ] **`wedadmin`**

>Luego de haber acceder a la web shell que nos dejaron los anteriores pentester, podemos navegar por el host y logramos identificar en el directorio **`/home`** a dos usuarios, **administrador** y **webadmin**, en el segundo logramos conseguir una id_rsa que podemos usar como conexión para **SSH**, además de un archivo ***`for-admin-eyes-only`***, al cual luego le podemos dar un vistazo.

![[id_rsa_webadmin.png]]

---

2. **Submit the credentials found in the user's home directory. (Format: user:password)**
   
	- [ ] **`mlefay:Plain Human work!`**
   
>Al leer el archivo ***`for-admin-eyes-only`*** vemos las credenciales de la respuesta, dichas credenciales deben ser usadas para acceder al **server01** u otros servidores en la subnet.

![[Creds_SubNet_mlefay.png]]

---
3. **Enumerate the internal network and discover another active host. Submit the IP address of that host as the answer.**
   
	- [ ] **`172.16.5.35`**

>Copiamos la **id_rsa** que conseguimos del usuario **webadmin**, y nos logramos conecta por SSH, con esto ahora podemos utilizar dicho HOST como Host de pivoting, pero primero debemos identificar que otros hosts o servidores están en la red interna.

>Para lograr identificar los host en la red interna, debemos identificar la Subnet que debemos escanear, esto lo podemos hacer con el comando **`ifconfig`** y ver las interfaces activas. Con esto conseguimos la interfaz **ens192** que cuenta con la dirección **172.16.5.15**, por lo que utilizamos un **forloop** para identificar host en la red **172.16.5.x**, esto se puede hacer con el siguiente bucle de bash:

```bash
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

>Con lo que conseguimos otro servidor interno con la dirección **172.16.5.35**.

![[Host-RedInterta.png]]

---
4. **Use the information you gathered to pivot to the discovered host. Submit the contents of C:\Flag.txt as the answer.**
   
	- [ ] **`S1ngl3-Piv07-3@sy-Day`**

>Ahora para esto, lo podemos hacer de distintas formas, en este caso utilizaremos la herramienta **`chisel`** para conseguir un Túnel Socks5, con el cual podamos conseguir retransmitir nuestros paquetes desde la máquina Linux y al servidor **172.16.5.35** que es un servidor Windows. Por lo que debemos transferir la herramienta al servidor Linux, que servirá como Host de Pivoteo. Esto se puede lograr con la herramienta **`scp`** dado que tenemos la **id_rsa** del usuario **webadmin**.

>Una vez lograda la transferencia, iniciamos la herramienta en modo servidor en el host de pivoteo, con el siguiente comando:

```bash
./chisel server -v -p 1234 --socks5
```

>Luego debemos iniciar en modo cliente, luego de haber verificado que el archivo **`/etc/proxychains.conf`** este configurado de forma correcta, debe verse de la siguiente forma:

```bash
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
# socks4 	127.0.0.1 9050 --> Debemos comentar esta lína de nuevo
socks5 127.0.0.1 1080 --> Esta línea debe ser añadida si no se encuentra
```

>Ahora debemos iniciar **Chisel**, en nuestra máquina de atacante, como clientes, conectándonos al puerto especificado del servidor **chisel**, esto se puede lograr con el siguiente comando:

```bash
./chisel client -v <Dirección IP - Servidor Chisel>:1234 socks
```

>Con esto logramos que los paquetes enviados a través de una herramienta como **proxychains** redirijan los paquetes a través del túnel SOCKS5, lo cual nos permitirá llegar a la máquina **172.16.5.35**, a la cual no tenemos acceso desde nuestra máquina atacante.

>Ahora luego de haber realizado un escaneo con **proxychain y nmap**, descubrimos el puerto **3389** de **RDP** abierto, por lo que intentamos acceder a través de **xfreedrp3** junto con **proxychain** recordando que conseguimos las credenciales **`mlefay:Plain Human work!`**.
	![[pivot-Server1.png]]

5. **In previous pentests against Inlanefreight, we have seen that they have a bad habit of utilizing accounts with services in a way that exposes the users credentials and the network as a whole. What user is vulnerable?**

	 - [ ] **`vfrank`**

>En este caso me costo un poco conseguir como obtener las credenciales, tuve que ir a mis notas del módulo **Password Attack**, donde pude leer y recordar que Windows, cuenta con el proceso **`LSASS`** **Servicio de Subsistema de Autoridad de Seguridad Local**, que encarga de verificar los intentos de inicio de sesión y demás temas de administración de seguridad de los usuarios.

>Para poder abusar de dicho proceso podemos utilizar la herramienta **`mimikatz`**, dado que con el siguiente comando podemos obtener información sobre los usuarios en el sistema, como sus nombres de usuarios, hash NTLM, a dominio pertenecen y más:

```Powershell
./mimikatz.exe privilege::debug  sekurlsa::logonpasswords
```

>La herramienta se logra transferir mediante el copiar y pegar, gracias a que contamos con una interfaz GUI a la máquina **172.16.5.35**.
	![[vfrank-Creds.png]]

>Al buscar entre los resultados vemos que el usuario **vfrank**, es el único que cuenta con una contraseña en texto claro, por lo que es el usuario vulnerable que estábamos buscando.

6. **For your next hop enumerate the networks and then utilize a common remote access solution to pivot. Submit the C:\Flag.txt located on the workstation.**
   
	- [ ] **``N3tw0rk-H0pp1ng-f0R-FuN``**

>Vemos que al ejecutar el comando **ipconfig**, la máquina cuenta con una segunda dirección IP **172.16.6.35**, que esta dentro del mismo segmento o sub red de **172.16.5.35**, dado que su máscara es **255.255.0.0**, por lo que ahora enumeraremos las direcciones IP's **172.16.6.1-255**. Intentando realizar un descubrimiento de host a través de trazas **ICMP** con la herramienta **PING** desde el usuario **mlefay**, no descubrimos nada, por lo que intentamos hacer el descubrimiento desde el usuario **vfrank**, y conseguimos dos dirección además de la nuestra, **172.16.6.25** y la **172.16.6.45**.

>Ahora dado que tenemos una interfaz gráfica, podemos intentar conectarnos a dichas máquinas con el **RDC** de Windows, iniciando con la conexión a la **172.16.6.25**, con la nuevas credenciales que encontramos del usuario **vfrank**, con éxito conseguimos la **flag**. 

![[flag-172.16.6.25.png]]

----
7. **Submit the contents of C:\Flag.txt located on the Domain Controller.**

	- [ ] **`3nd-0xf-Th3-R@inbow!`**

>En este punto conseguimos un recurso compartido al cual tenemos acceso.

![[Recurso compartido.png]]

>En dicho recurso conseguimos la ultima **Flag**.

![[flag-DC.png]]

---

## Conclusiones

>Esta práctica del módulo **`Pivoting, Tunneling, and Port Forwarding`** de [**Hack The Box Academy**](https://academy.hackthebox.com/module/details/158) es una excelente forma de poner en práctica como se mueven los atacantes en una red empresarial, que por lo general, se encuentra segmentada, encontrando un **Host de Pivoteo** el cual pueden usar como punto de apoyo para saltar a otros equipo de la red e ir escalando privilegios a medida que van comprometiendo los demás.


