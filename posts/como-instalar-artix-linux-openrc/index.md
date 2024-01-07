# Como Instalar Artix Linux Openrc

## Tutorial en YouTube

{{&lt; youtube 8eAgxwZBPjI &gt;}}
## Bajar el ISO

Lo primero es descargar el ISO del sitio &lt;a href=&#34;https://artixlinux.org/download.php&#34; target=&#34;_blank&#34;&gt;web oficial de Artix Linux&lt;/a&gt;

Artix tiene varias versiones, en esta ociasión vamos a realizar una instalación base, también instalaremos OpenRC en lugar de systemd y por último pondermos Mate como escritorio:

![descarga el ISO que dice &#34;openrc&#34;](/images/posts/artix-mate/download.png)
## Comienza la Instalación

Ya que arrancamos nuestra máquina, ya sea virtual o física comencemos la instalación entonces.

![Artix Pantalla Inicial](/images/posts/artix-mate/login-screen.png)

### Verificiar que tengas conexión a Internet.

```bash
ip a
```

Este comando debe regresar los nombres de los dispositivos de red y una dirección de IP.

### Crear particiones

Antes de crear las particiones y comezar la instalación, debemos cambiarnos al usuario de &#34;root&#34;

```bash
sudo su -
```

Para verificar el nombre del dispositivo, es decir el disco donde instalaremos Artix, teclea:

```bash
lsblk
```

 En mi caso el disco que donde voy a instalar Artix es &#34;vda&#34;

 ![lsblk](/images/posts/artix-mate/lsblk.png)

 ```bash
 cfdisk /dev/vda
 ```

Como es una instalación sobre UEFI, selecciona &#34;gpt&#34;

Crea una partición para &#34;boot&#34; y otra para &#34;/&#34;

La partición de &#34;boot&#34; será de 100 MB y el resto del disco para &#34;raíz&#34;.

Recuerda que la partición de &#34;boot&#34; debe ser tipo EFI system

![Así deben quedar las particiones](/images/posts/artix-mate/cfdisk.png)

### Formatear particiones

Formateamos la primera partición de tipo &#34;FAT32&#34;

```bash
mkfs.fat -F 32 /dev/vda1
```

Luego formateamos la segunda partición de tipo &#34;ext4&#34;

```bash
mkfs.ext4 /dev/vda2
```
### Montar las particiones

Primero montamos la partición 2 **&#34;vda2&#34;** en /mnt

```bash
mount /dev/vda2 /mnt
```

Luego creamos el directorio de **boot** y montamos la primera partición

```bash
mkdir /mnt/boot
mount /dev/vda1 /mnt/boot
```
### Instalar el sistema base

```bash
basestrap /mnt base base-devel linux linux-firmware openrc vim amd-ucode (intel-ucode si es tu caso)
```

### Generar tabla de sistema de archivos (fstab)

```bash
fstabgen -U /mnt &gt;&gt; /mnt/etc/fstab
```

## Configurar el sistema base

Para esto hay que entrar a **nuestra instalación** tecleando:

```bash
artools-chroot /mnt
```

### Crear archivo de Swap

```bash
dd if=/dev/zero of=swapfile bs=1G count=2 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

### Editar /etc/fstab

Agrega lo siguiente al final del archivo

```text
/swapfile   none    swap    defaults    0 0
```

### Configurar el reloj del sistema

```bash
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

#### Sincroniza el reloj del hardware con el del sistema

```bash
hwclock --systohc
```

### Localización 

```bash
vim /etc/locale.gen
```

Genera el &#34;locale&#34;

```bash
locale-gen
```

Necesitamos ahora poner este &#34;locale&#34; en el archivo de configuración

```bash
vim /etc/locale.conf
```

agrega lo siguiente

```text
LANG=&#34;en_US.UTF-8&#34;
```

### GRUB

Para instalar GRUB

```bash
pacman -S grub os-prober efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub (para sistemas con UEFI)
grub-mkconfig -o /boot/grub/grub.cfg
```

### Crea un password para usuario root

```bash
passwd
```

### Crea tu usuario

```bash
useradd -mG wheel jorge
passwd jorge
```

### Habilita el grupo &#34;wheel&#34;

```bash 
EDITOR=vim visudo
```

Descomenta la linea que dice:

```text
%wheel ALL=(ALL) ALL
```

### Configura la red

Edita el archivo 

```bash
vim /etc/hostname
```

Agrega solo el nombre de tu computadora, sin dominio ni nada más

Después edita el archivo 

```bash
vim /etc/hosts
```

Para que quede como sigue:

```text
# Static table lookup for hostnames.
# See hosts(5) for details.

127.0.0.1	localhost
::1	localhost
127.0.0.1	artix.localdomain	artix
```


### Instala connman

```bash
pacman -S connman-openrc connman-gtk (or cmst for Qt-based DEs)
rc-update add connmand
```

### Resetea la computadora

```bash
exit
umount -R /mnt
reboot
```

## Configuración post-instalación e Instalación de Escritorio (MATE)

### Instala xorg

```bash
sudo pacman -Syyy
sudo pacman -S xorg --ignore xorg-server-xdmx
```

### Instala el controlador de tu tarjeta de video

Si tienes tarjeta **intel** instala:

```bash
sudo pacman -S xf86-video-intel
```

Si tienes tarjeta **AMD** instala:

```bash
sudo pacman -S xf86-video-amdgpu
```

Si tienes tarjeta **NVIDIA** instala: 

```bash
sudo pacman -S nvidia-lts
```

### Instala el escritorio

```bash
sudo pacman -S mate mate-extra system-config-printer blueman connman-gtk
```

### Instala el Administrador de Inicio de Sesión (Display Manager)

```bash
sudo pacman -S displaymanager-openrc sddm
```

### Habilita el Administrador de Inicio de Sesión en OPENRC

```bash
sudo rc-update add xdm default
sudo vim /etc/conf.d/xdm 
```

Edita para que quede así:

```text
DISPLAYMANAGER=&#34;sddm&#34;
```

### Reinicia el sistema.

```bash 
sudo reboot
```



---

> Autor:   
> URL: http://localhost:1313/posts/como-instalar-artix-linux-openrc/  

