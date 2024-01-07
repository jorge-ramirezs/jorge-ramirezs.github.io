# Como Instalar Archlinux en BIOS - MBR


&lt;!-- ![Como instalar Arch Linux en VirtualBox](/images/posts/archvbox-install/Arch-Linux-VirtualBox.png) --&gt;

## Bajar el ISO

El primer paso obviamente es bajar el ISO del &lt;a href=&#34;https://www.archlinux.org/download/&#34; target=&#34;_blank&#34;&gt;sitio oficial&lt;/a&gt;

## Instala Arch Linux

Una vez que hayas bajado el ISO y creado el USB, arranca tu maquina.

A continuación los pasos de la instalación.

### Verifica que tengas Conexión a Internet.

```bash
ip a
```

El comando anterior debe regresarte los dispositivos de red, se recomienda que tengas conectada tu computadora por cable ethernet.

Si no tienes cable conectado, entonces tendrás que utilizar iwctl:

```bash
iwctl
```

https://wiki.archlinux.org/index.php/Iwd#iwctl

#### Configura el Teclado

En mi caso estoy utilizando un teclado en Inglés así que no requiero en este paso, pero si tu estás utilizando un teclado en Español lo mejor es que lo configures para no batallar y hacer la instalación más sencilla.

```bash
loadkeys es
```

#### Sincroniza el reloj

A continuación vamos a sincronizar el reloj con internet, para eso escribimos

```bash
timedatectl set-ntp true
```

#### Selecciona el servidor más cercano para instalar los paquetes (Mirror)

Para esto vamos a necesitar un paquete que se llama [**Reflector**](https://wiki.archlinux.org/index.php/Reflector), así que vamos a instalarlo primero escribiendo

```bash
pacman -Syy reflector
```

![Instala Reflector](/images/posts/archvbox-install/instala_reflector-min.jpg)

Ahora selecciona el servidor más cercano a donde te encuentras, para ver una lista de países escribe

```bash
reflector --list-countries | more
```

![Reflector lista de países](/images/posts/archvbox-install/reflector_paises-min.jpg)

Ya que tengas identificado el país que te queda más cerca escribe lo siguiente:

(En mi caso como estoy en Estados Unidos, escogeré United States por supuesto, pero tú puedes cambialo por el país de la lista anterior que te quede más cerca)

```bash
reflector -c &#34;United States&#34; -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

![Reflector actualiza mirrorlist](/images/posts/archvbox-install/reflector_actualiza_mirrorlist-min.jpg)

#### Actualiza los servidores de paquetes

Siguente teclea

```bash
pacman -Syyy
```

![pacman -Syyy](/images/posts/archvbox-install/actualiza_pacman-min.jpg)

### Crea Particiones en el Disco

Primero vamos a verificar cual es el dispositivo donde se encuentra el disco de nuestra máquina virtual

```bash
lsblk
```

![lsblk](/images/posts/archvbox-install/lsblk01-min.jpg)

Mi disco está en /dev/**sda**

Tenemos que crear 2 particiones, una para los archivos de inicio o &#34;boot&#34; y otra donde estará instalado Arch Linux.
Para crear las particiones voy a utilizar **cfdisk**

```bash
cfdisk /dev/sda
```

**Selecciona &#34;dos&#34; porque esta es una instalación en MBR**

Vamos a crear 2 particiones, la primera partición será para SWAP y la segunda para la instalación del sistema.

1. Selecciona [ New ]

2. El tamaño de la primera partición será de **4GB** en mi caso

3. Selecciona [ primary ] y presiona ENTER

4. Cambia el tipo de partición a SWAP seleccionando [ Type ] y luego &#34;Linux swap&#34;

5. Selecciona el espacio disponible (Free Space), luego selecciona [ New ] , presiona ENTER y luego ENTER de nuevo para seleccionar todo el espacio disponible, en mi caso los 9.8G restantes

6. Selecciona [ Type ] y luego selecciona **&#34;Linux filesystem&#34;** de la lista

7. Selecciona [ Write ] para guardar los cambios y crear las particiones. cfdisk te preguntara si estás seguro de que deseas crear las nuevas particiones. Teclea &#34;yes&#34; y presiona ENTER

#### Verifica las particiones

Para verificar que todo haya ido bien teclea lsblk y asegurate de que las particiones estén como sigue

![lsblk](/images/posts/archvbox-install/lsblk02-min.jpg)

#### Formatea las particiones

1. La primera partición es nuestra partición de SWAP, esta NO es necesario que se formatee.

2. Formatea la segunda partición, pero en esta ocasión el tipo será EXT4

```bash
mkfs.ext4 /dev/sda2
```

![Formatea la segunda partición](/images/posts/archvbox-install/mkfs_ext4-min.jpg)

3. La partición de SWAP no es necesaria formatearla pero si debemos crearla y activarla como sigue:

```bash
mkswap /dev/sda1
swapon /dev/sda1
```

### Monta las particiones

Una vez creadas y formateadas nuestras particiones ahora debemos montarlas en nuestro sistema, para eso necesitmos hacer lo siguiente:

1. Montar la segunda partición en **/mnt** que es nuestro directorio para la instalación

```bash
mount /dev/sda2 /mnt
```

2. Verifica que las particiones estén montadas correctamente utilizando lsblk

```bash
lsblk
```

#### Instala los paquetes &#34;base&#34;

Una vez montadas las particiones comenzamos la instalación de los paquetes base, los paquetes que vamos a instalar son &#34;base, linux, linux-firmware y vim (puedes instalar nano si lo deseas)

```bash
pacstrap /mnt base linux linux-firmware vim
```

![Instalación base](/images/posts/archvbox-install/instalacion_base-min.jpg)

![Instalación base](/images/posts/archvbox-install/instalacion_base02-min.jpg)

#### Crear fstab

Listo, la instalación base ya está lista, ahora lo que necesitamos hacer es crear nuestro archivo **fstab**

```bash
genfstab -U /mnt &gt;&gt; /mnt/etc/fstab
```

### Entra a la instalación

Listo, ya que veas en tu archivo de fstab que todo está correcto, que tienes ambas particiones configuradas correctamente, vamos a crear nuestro archivo de swap.

1. Primero entra a la instalación base

```bash
arch-chroot /mnt
```

### Configura tu zona horaria

Ahora podemos continuar con la configuración de nuestra instalación, así que toca el turno de la zona horaria.

1. Primero vamos a ver un listado de las zonas horarias disponibles. La zona horaria donde me encuentro es la zona de **Chicago**, sustituye Chicago por la ciudad que te corresponda, por ejemplo Mexico

```bash
timedatectl list-timezones | grep Chicago
```

2. Una vez identificada la zona horaria, creamos un enlace simbolico en **etc/localtime**

```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

3. Ahora ya podemos sincronizar el reloj del sistema con el reloj de hardware

```bash
hwclock --systohc
```

### Configura tu &#34;Localización&#34; (locale)

Esto es para decirle a Arch Linux donde te encuentras y cual es el grupo de caracteres que corresponden a tu idioma por ejemplo.

En mi caso voy a utlizar Inglés de Estados Unidos, el código es **&#34;en_US.UTF-8&#34;** pero por ejemplo, si te encuentras en México y estás utilizando teclado en Español y deberás utilizar el código de **&#34;es_MX.UTF-8&#34;**.

**Nota:** Puedes habilitar más de un código de localización.

1. Para habilitar el código de localización deseado, edita el siguente archivo y descomenta la línea donde se encuentre el código que andas buscando. (De nuevo, yo utilizo vim, tu puedes hacerlo con nano o con el que hayas instalado al inicio)

```bash
vim /etc/locale.gen
```

![vim locale.gen](/images/posts/archvbox-install/vim_locale_gen-min.jpg)

2. Genera la localización en el sistema

```bash
locale-gen
```

![locale-gen](/images/posts/archvbox-install/locale_gen-min.jpg)

3. Ahora necesitamos crear el archivo de configuración en nuestra instalación para eso:

```bash
echo LANG=en_US.UTF-8 &gt;&gt; /etc/locale.conf
```

**Nota:** Sustituye &#34;en_US.UTF-8&#34; por el código que hayas habilitado en el paso anterior

4. (Opcional) Si al comienzo de la instalación seleccionaste el tipo de teclado, deberás agregarlo a tu instalación con el siguente comando:
   (Suponiendo que seleccionaste &#34;es&#34;)

```bash
echo KEYMAP=es &gt;&gt; /etc/vconsole.conf
```

### Configura los archivos de Red

1. Crea el archivo &#34;hostname&#34; para darle un nombre a tu máquina virutal. En mi caso voy a ser creativo y le daré el nombre de &#34;archlinux&#34;

```bash
vim /etc/hostname
```

![hostname](/images/posts/archvbox-install/hostname-min.jpg)

2. Agrega lo siguiente a tu archivo /etc/hosts

```bash
vim /etc/hosts
```

```txt
127.0.0.1	localhost
::1		localhost
127.0.1.1	archlinx.localdomain	archlinux
```

![hosts](/images/posts/archvbox-install/hosts-min.jpg)

### Crea un password para root

El siguiente paso de la instalación es darle un password a nuestro usuario de root.

Como ya estamos en nuestra instalación, para cambiar el password solo debemos teclear:

```bash
passwd
```

En el prompt escribe el nuevo password (Uno que no se te olvide) y en el siguente prompt vuelve a escribir el mismo password para confirmarlo.

![Password root](/images/posts/archvbox-install/passwd_root-min.jpg)

### Instala los paquetes finales

Es momento de instalar nuestro &#34;boot loader&#34; y paquetes finales antes de reiniciar la máquina. GRUB será nuestro boot loader.

1. Instala los siguientes paquetes. Puedes agregar paquetes que creas necesarios.

```bash
pacman -S grub networkmanager network-manager-applet dialog wireless_tools wpa_supplicant os-prober mtools dosfstools base-devel linux-headers cups reflector openssh git virtualbox-guest-utils
```

Acepta todos los prompts por defecto. Y presiona ENTER para continuar con la instalación

![Instala Grub y Otros Paquetes](/images/posts/archvbox-install/paquetes_finales-min.jpg)

![Instala Grub y Otros Paquetes](/images/posts/archvbox-install/paquetes_finales2-min.jpg)

Si te aparece la siguiente pantalla, presiona ENTER para aceptar las llaves PGP.

![Instala Grub y Otros Paquetes](/images/posts/archvbox-install/paquetes_finales3-min.jpg)

### Instala GRUB

Es momento de instalar grub en el disco, como es una instalación en MBR, el comando es como sigue:

```bash
grub-install --target=i386-pc /dev/sda
```

3. Ahora necesitamos crear el archivo de configuración

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Habilita los servicios

Llegó el momento de habilitar los servicios que correrán cada que reinicies la máquina.

1. Habilita el manejador de la red
2. Continua con el servicio de SSH
3. Luego el servicio para imprimir

```bash
systemctl enable NetworkManager
systemctl enable sshd
systemctl enable org.cups.cupsd
```

![Habilita Servicios](/images/posts/archvbox-install/habilita_servicios-min.jpg)

### Crea tu usuario

Todo lo hemos hecho con el usuario de root, pero siempre es conveniente tener tu propio usuario para no utilizar root para las tareas diarias. Voy a crear mi usuario que se llamará &#34;jorge&#34;.

1. Crea el usuario y asignalo al grupo &#34;wheel&#34;
2. Crea un password para tu usuario

```bash
useradd -mG wheel jorge
passwd jorge
```

3. Ahora para darle privilegios de &#34;sudo&#34; y puedas ejecutar comandos como superusuario &#34;root&#34;, debemos descomentar la linea que dice **&#34;%wheel ALL=(ALL) ALL&#34;** con el siguiente comando

Recuerda que puedes sustituir &#34;vim&#34; por &#34;nano&#34; si lo prefieres.

```bash
EDITOR=vim visudo
```

![](/images/posts/archvbox-install/visudo-min.jpg)

### Sal de la instalación y reinicia la máquina virtual

Con esto terminamos la instalación, así que solo queda:

1. Salir de la instalación
2. Desmontar las particiones
3. Reiniciar la máquina virtual

```bash
exit
umount -a
reboot
```

![](/images/posts/archvbox-install/reboot-min.jpg)

## Instalación Exitosa

Después de reiniciar la máquina virtual verás que GRUB está instalado y debe estar funcionando correctamente.

![](/images/posts/archvbox-install/grub_inicio-min.jpg)

Entra a tu nueva instalación de Arch Linux con el usuario y password que creaste para verificar que todo esté funcionando correctamente.

![](/images/posts/archvbox-install/instalacion_completa-min.jpg)

Si encontraste este tutorial útil no olvides que puedes compartirlo con tus compañeros, familiares y amigos utilizando los botones al final del artículo.

No dejes de visitarme porque estaré subiendo más tutoriales, ahora lo que sigue es instalar un escritorio en esta máquina virtual. ¿Cuál te gustaría instalar? ¿Gnome? ¿KDE? ¿O talvez un manejador de ventanas como i3 o awesome?


---

> Autor:   
> URL: http://localhost:1313/posts/como-instalar-archlinux-en-mbr/  

