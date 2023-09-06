---
title: "Btrfs y respaldo de sistemas"
layout: post
author:
  name: Xavier Góngora
  email: xaviergongora.contacto@gmail.com
---
Nunca había dedicado mucho tiempo a pensar sobre el [sistema de archivos](https://www.freecodecamp.org/news/file-systems-architecture-explained/) de mi computadora. 
Desde el punto de vista de un usuario, los detalles de este tipo de tecnología son irrelevantes hasta que algo falla;
se encuentra en un muy bajo nivel y es manejado por el sistema operativo. Su función es definir el uso de la memoria en disco. 
Hasta hace poco, lo único que había investigado sobre sistemas de archivos era en relación a temas de compatibilidad.[^exfat]

## Btrfs en Garuda Linux

Fue al instalar [Garuda Linux](https://garudalinux.org) en mi _desktop_ que empecé a indagar en el tema. Una de sus características emblemáticas es el uso de _snapshots_ de [Btrfs](https://btrfs.readthedocs.io/en/latest/index.html) como estrategia de recuperación del sistema:[^btrfs] En caso de que el sistema se rompa, lo cual sucede en ocasiones al actualizarlo, se puede acceder a una lista imágenes previas del sistema desde un menu durante el arranque ([GRUB](https://en.wikipedia.org/wiki/GNU_GRUB)). Estas imágenes (las _snapshots_) se toman de manera automática antes y después de cada actualización del sistema.
Esta característica específica permite utilizar [Arch Linux](https://wiki.archlinux.org/title/Arch_Linux), en la que se basa Garuda,[^garuda] de manera _confiable_. Arch es una distribución
que (valga la redundancia) distribuye el software directo desde los repositorios de sus desarrolladores (_upstream_) de manera continua.[^rolling] La ventaja: siempre se tienen los ultimos
arreglos, parches de seguridad y características. La desventaja: las cosas pueden salir mal y un programa o todo el sistema puede dejar de funcionar.
Ambas son razones por las que Arch Linux es una distribución considerada exclusiva para usuarios de Linux expertos o entusiastas.[^arch] Btrfs tiene la capacidad de crear _snapshots_ con un costo computacional prácticamente nulo.[^bemoles] Explicar que es exactamente una _snapshot_ en Btrfs excede la intención de este _post_;[^snap] baste decir que se trata de una imagen (de alguna parte) del sistema que permite revertir los cambios en caso necesario. Así, en caso de que algo se rompa al actualizar el sistema, podemos revertir los cambios sin mayor problema.

[^exfat]: Cuando tuve que formatear un disco externo para respaldo de video de alta definición descubrí que exFAT es LA opción de sistema de archivos para soporte multiplataforma. FAT32 es mejor en cuanto a compatibilidad, pero limita los archivos a un tamaño máximo de 4 GB.

 [^btrfs]: Garuda es una de las primeras distribuciones que ofreció Btrfs como sistema de archivos por defecto. Este es un sistema de archivos _copy-on-write_(CoW), lo que permite la creación ágil de _snapshots_. Otro sistema de archivos CoW muy querido por los administradores de sistemas es [ZFS](https://en.wikipedia.org/wiki/ZFS), creado originalmente por Sun Microsystems.

[^garuda]: Quizás sería más claro pensar Garuda Linux como una configuración empaquetada de Arch Linux. Sin embargo hay características que me la hacen merecer el título de distribución derivada: 
    * Incluye dos repositorios adicionales: garuda (desde donde distribuyen sus propias configuraciones y _scripts_) y [chaotic-aur](https://aur.chaotic.cx/).
    * Utilizan [Calamares](https://calamares.io/) para instalación con interfaz gráfica.
    * Además de la curaduría de paquetes pre-instalados, mantienen varios programas propios para la administración del sistema.
    * Su script de instalación, `garuda-update`, envuelve al comando regular de actualización de Arch (`sudo pacman -Syu`). Esto les permite incluir arreglos desde la actualización.
    * Mientras no se utilizen las herramientas de Garuda, la experiencia de uso y administración es identica a la de Arch.


[^bemoles]: Las ventajas de Btrfs tienen sus costos: este sistema de archivos requiere de más acciones de mantenimiento por parte del usuario, en comparación a ext4 (el estándar en Linux). Por ejemplo, es necesario borrar las _snapshots_ viejas, ya que con el paso del tiempo sus diferencias con el sistema en operación crecen y llegan a ocupar mucho espacio en disto. En Garuda el mantenimiento está automatizado utilizando herramientas como [btrfsmaintenance](https://github.com/kdave/btrfsmaintenance) y [snapper](http://snapper.io/).

[^arch]: Otra razón es que su instalación, sin la mediación de una distribución preconfigurada como Garuda Linux o EndeavorOS, involucra la elección e instalación de todos los componentes del sistema desde una terminal.

[^rolling]: A esto se le conoce como modelo [_rolling release_](https://itsfoss.com/rolling-release/).

[^snap]: En Btrfs, una _snapshot_ es esencialmente un _subvolumen_. Ver [Fedora Magazine: Workging with Btrfs - Snapshots](https://fedoramagazine.org/working-with-btrfs-snapshots/)

Es importante mencionar que estas _snapshots_ **no son una una solución de respaldo** para la integridad de los archivos (aunque se puede implementar con ellas). Dada la naturaleza de Btrfs, y los sistemas de archivos CoW, si un archivo está corrupto entonces lo está para todas las _snapshots_ que lo comparten. 

Tuve que investigar estas cosas debido a que cuando instalé Garuda cometí un error inocente. Decidí hacerle caso a un [video tutorial](https://youtu.be/iBDIj-J3U28?si=NQ3gv1MOjY_fqmPj) de instalación en que se creaba una partición Btrfs separada para guardar los archivos personales (`/home`). Experiencias previas con otras distribuciones de Linux me habián enseñado la ventaja de tener particiones separadas para los archivos del sistema (`/`) y los archivos personales. 
Sin embargo esto no aplicaba de la misma manera en Garuda. 
Al hacer la partición, evité que el instalador de Garuda ubicara `/home` en su propio _subvolumen_. Los subvolúmenes de Btrfs son similares a una partición, pues
permiten administrarla de manera independiente gracias a que establecen un tipo de división lógica, con la capacidad extra de acceder al almacenamiento disponible de forma dinámica.[^subvol]
En Garuda se crean por defecto snapshots de `/`, cada vez que se hace una actualización del sistema. Para ilustrar como funciona esto hay que considerar el arreglo de subvolúmenes que hace Garuda al instalar el sistema:[^suse]

| Nombre del subvolumen | Punto de montaje |
|-----------------------|------------------|
| @                     | /                |
| @root                 | /root            |
| @srv                  | /srv             |
| @cache                | /var/cache       |
| @log                  | /var/log         |
| @tmp                  | /var/tmp         |
| @home                 | /home            |
    
La función de este arreglo de subvolúmenes es omitir los directorios correspondientes de las snapshots. Estos contienen archivos que no conviene revertir al volver a un estado anterior del sistema. `/home` contiene los archivos y configuraciones personales de los usuarios. `/root` es análogo a `/home` para el usuario _root_. `/var/cache` contiene archivos transitorios de las aplicaciones. `/var/log` es para los registros del sistema. `/var/tmp` contiene archivos temporales que las aplicaciones preservan entre reinicios. `/srv` contiene archivos relacionados con servicios de red proporcionados por el sistema.[^fhs]

[^fhs]: Estos roles están especificados en la [Filesystem Hierarchy Standard (FHS)](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.html) a la que se apega Arch Linux.


Tener `/home` en una partición separada me impidía administrarla con las herramientas de Garuda que dependen de este arreglo, aunque la partición fuera también Btrfs. Además me empecé a quedar con espacio limitado en el disco para mis archivos. La solución que encontré a esto, evitando reinstalar todo el sistema, está documentada en el [foro de Garuda](https://forum.garudalinux.org/t/moving-home-partition-to-a-btrfs-subvolume/26336/10).

 [^subvol]: Por ejemplo, resulta trivial añadir un nuevo dispositivo de almacenamiento al sistema y redistribuir los archivos entre todos los dispositivos disponibles: simplemente se lo conecta, se añade al pozo de dispositivos y luego se da la instrucción de balancear el sistema de archivos. Para detalles ver este [tutorial](https://www.techrepublic.com/article/how-to-add-a-device-on-btrfs-system/).
 
 [^suse]: En Garuda el acomodo de subvolúmenes, utilizado por snapper, es [plano](https://archive.kernel.org/oldwiki/btrfs.wiki.kernel.org/index.php/SysadminGuide.html#Flat) y está basado en el [acomodo de openSUSE](https://en.opensuse.org/SDB:BTRFS).
   

## Respaldar Doom Emacs

Lograr utilizar lo aprendido sobre Btrfs para idear una simple estrategia para respaldar mi editor de texto me inspiró a escribir este _post_. Utilizó [Doom Emacs](https://github.com/doomemacs/doomemacs): una esquema de configuración para el clásico editor de texto hacker [Emacs](https://www.gnu.org/software/emacs/download.html). El problema con Doom es similar a Arch: muchos de los paquetes se actualizan regularmente siguiendo al _upstream_, además de que la configuración de Emacs puede ser temperamental _per se_. Esto me ha llevado en muchas ocasiones (significativamente más de las que me han sucedido con Arch), a que Doom deje de funcionar después de una actualización. La siguiente estrategia sólo aplica a instalaciones donde `/home` es (parte de) un sistema de archivos Btrfs.

### La estrategia

Lo primero es mudar Doom Emacs a un subvolumen, para ello hay que:

Cambiar el nombre del directorio donde se guardan los componentes de Doom,

`mv ~/.emacs.d ~/.emacs.d.bak`

crear un subvolumen para la nueva ubicación con el nombre que liberamos

`sudo btrfs subvolume create ~/.emacs.d`

y copiar todos los archivos (con sus propiedades) a la nueva ubicación:

`cp -a ~/.emacs.d.bak/* ~/.emacs.d`

Este último paso es relativamente innecesario, pues Doom es declarativo y su especificación se encuentra en `~/.doom.d`:[^doom] bastaría correr `doom sync -u` para volver a poblar esta ubicación. Sin embargo esto toma mucho tiempo.

[^doom]: En este directorio se ubican tres archivos: uno para la especificación de componentes (`init.el`), otro para las opciones de configuración (`config.el`) y uno para paquetes adicionales (`packages.el`).

Si Doom está funcionando correctamente, podemos borrar el respaldo con `rm -rf ~/.emacs.d.bak`.

**La estrategia consiste en crear una snaphot de solo escritura, o _read-only_,[^snap] antes una actualización**. 

`sudo btrfs subvolume snapshot -r ~/.emacs.d/ ~/.emacs.d.bak.ro`

En caso de que la actualización deje a Doom fuera de combate, los siguientes pasos lo reviertiran a su estado anterior:

1. Borrar la configuración defectuosa: `sudo btrfs subvolume delete ~/emacs.d/`
1. Restaurar la vieja configuración: `sudo btrfs subvolume snapshot ~/.emacs.d.bak.ro ~/.emacs.d`

Doom Emacs es un editor fantástico, y ahora no tengo reservas en actualizarlo más cotidianamente. Por su parte Btrfs me ha demostrado las posbilididades de los sistemas de archivos. La exploración de este tunel valió la pena (espero).

---
