## HTB Academy: Pivoting, Tunneling & Port Forwarding Skill Assessment

Este repositorio contiene el WriteUp del Skill Assessment del módulo Pivoting, Tunneling & Port Forwarding de Hack The Box Academy. Aquí se detallan los pasos seguidos para resolver el desafío, junto con explicaciones de las herramientas y técnicas utilizadas.
### 🎯 Objetivo

El objetivo del Skill Assessment es aplicar técnicas avanzadas de pivoteo, tunelización y redirección de puertos para acceder a redes internas y explotar servicios restringidos.
### 🚀 Herramientas y Protocolos Utilizados

En la resolución de este desafío se emplearon las siguientes herramientas y protocolos:

    Chisel: Herramienta para crear túneles basados en TCP/UDP.
    SCP: Protocolo seguro para la transferencia de archivos entre hosts.
    SSH: Protocolo para acceso remoto y redirección de puertos.
    Proxychains: Herramienta para enrutar tráfico de red a través de proxies configurados.
    Mimikatz: Herramienta para la extracción de credenciales en sistemas Windows.
    RDP: Protocolo de escritorio remoto.
    Ping (ICMP): Para enumeración de hosts en la red interna.

### 📝 Contenido del WriteUp

El WriteUp cubre los siguientes aspectos clave:

    Reconocimiento inicial: Identificación del punto de entrada y mapeo básico de la red.
    Pivoteo mediante túneles:
        Configuración de túneles con Chisel.
        Uso de redirección de puertos con SSH.
    Transferencia de archivos sensibles:
        Uso de SCP para mover herramientas a sistemas internos.
    Enumeración avanzada:
        Escaneo de la red interna mediante ping y herramientas auxiliares.
    Explotación:
        Uso de Mimikatz para la extracción de credenciales.
        Conexión mediante RDP a sistemas comprometidos.
    Evasión de restricciones de red:
        Configuración de Proxychains para enrutar tráfico hacia servicios internos.

### ⚠️ Nota Importante

Este WriteUp es para fines educativos. La reproducción de estas técnicas en entornos no autorizados está estrictamente prohibida y podría violar leyes locales e internacionales.

¡Si tienes alguna sugerencia o mejora, no dudes en enviar un Pull Request o abrir un Issue!
