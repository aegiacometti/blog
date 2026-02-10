---
title: "El ABC de Ansible"
date: 2020-04-11
categories: 
  - "uncategorised"
tags: 
  - "ansible"
---

![](/images/image-2.png)

[Ansible](https://www.ansible.com/) es la plataforma de automatización más adoptada en IT. Esto se debe a que podes utilizarlo sobre prácticamente todos los sistemas operativos Windows, Linux, Redes, etc. Si te podes conectar remotamente al sistema entonces seguramente puedas utilizar Ansible. Y ademas no necesitas instalar un agente en los equipos remotos!

Ansible es un producto de RedHat, asique cuenta con un buen desarrollo y soporte, muy buena documentación, y ademas es Open Source, es decir, es gratis, y todos pueden leer y colaborar con el código.

Sin ser académico voy a darte mi interpretación de lo que conozco de Ansible al momento, no pretendo ser perfecto, solo darte la idea básica y masticada sin que tengas que pasar por esos duros momentos donde no entendés porque no anda. De ahí en más esta en vos profundizar donde te interese.

Ansible es un wrapper (envoltorio) sobre muchos pedazos de código que al modo de plug-in se insertan para hacer el todo. Tenes plug-ins (dependiendo del contexto/función a veces llamados módulos) para distintas funciones como: inventario, recursos, comandos Cisco, comandos Windows, etc.

Y lo más importante a tener en cuenta es que corre sobre Linux/Unix y derivados. Es decir, no corre sobre Windows, aunque SI podemos gestionar remotamente un Windows vía PowerShell/WinRM, mientras que también soporta SSH y APIs HTTP/S.

Lo que hace básicamente es conectarse al sistema remoto, ejecutar un comando, parsearlo en base a plantillas definidas en ese plugin/modulo y devolver una respuesta ordenada en base a lo que hace el modulo/plugin.

El código se escribe en formato [YML](https://es.wikipedia.org/wiki/YAML) que es muy fácil de leer, no necesitas saber programar. Aunque si te gusta programación, Ansible está escrito en Python, que es un lenguaje de programación muy claro y fácil de aprender.

La gracia de Ansible es que en base al inventario que definas, podes loopear sobre el mismo y de esa manera 1 simple comando lo ejecutas en todo inventario en sin tener que conectarte equipo por equipo. Aquí es donde empezas a automatizar en este caso la ejecución de 1 comando en múltiples equipos.

De ahí en adelante, cada output lo podes pasar a otra tarea (**Task**), y esa otra tarea a otra y así sucesivamente. Todo ese conjunto de tareas para lograr un objetivo X se lo llama **Playbook**.

Bueno arranquemos. Primero unos tips, y al final la instalación.

1. Archivo de inventario tipico: tiene nombre, IP, OS, user y password.

![](/images/image.png)

El nombre del host que aparece primero, es el nombre que va a utilizar Ansible para referise a ese equipo y no tiene nada que ver con el DNS name.

2. Un playbook hiper sencillo, guardar la configuración en un archivo.

![](/images/image-1.png)

- Con "_**hosts: all**_" le estas diciendo que ejecute todo ese Playbook una vez por cada host del inventario.
- "**_ios\_commands_**" hace referencia al modulo de IOS, y el comando a ejecutar es el mismo del CLI de IOS
- "**_register: output_**" le está diciendo que el output lo guarde en una variable con el nombre output
- "**_ignore\_errors: yes_**" es para que no corte la ejecución si no se pudo conectar a alguno de los equipos, sino que siga igual con el resto.
- cada vez que ves "_**{{ xxx }}**_" le estas diciendo que reemplace **_xxx_** con el valor de la variable
- "_**delegate\_to: localhost**_" es para decirle que esa tarea (TASK - name) no la ejecute con los hosts del inventario, sino que la haga en el host linux desde donde estas ejecutando el Playbook, y la va a ejecutar una vez por cada host de invetario

4. Revisa y aplica las "notas especiales de SSH" de éste [link](https://github.com/aegiacometti/netconf-backup#special-ssh-connectivity-notes). Sino la conexion SSH a los equipos remotos no te va a funcionar con los IOS tradicionales.

5. En algún momento, lee sobre Ansible Vault en éste [link](/posts/2020-04-05-quick-start-ansible-vault/). Es para encriptar las passwords y no ponerlas sin protección en el inventario de Ansible. Si bien al principio no importa y es mejor enfocarse en aprender a hacer lo básico. Ahora cuando leas como encriptarlas y mantenerlas, vas a ver que es muy fácil y no lleva nada de tiempo. Ahora o después, lo vas a tener que hacer igualmente.

6. Y finalmente, en éste [link](https://www.digitalocean.com/community/tutorials/como-instalar-y-configurar-ansible-en-ubuntu-18-04-es) tenes una guía rápida muy buena para instalar y comenzar a usar Ansible.

7. Si queres ver en detalle que esta haciendo un Playbook como para debug, agrega al final de la linea `-vvvv`

8. Si queres limitar la ejecucion de un Playbook a uno/varios equipos podes agregar al fina de la linea `-l xxx*` donde xxx es parte del nombre.

Buena suerte y que disfrutes de la automatización. Si tenes alguna duda o problema, avisame.

Saludos
