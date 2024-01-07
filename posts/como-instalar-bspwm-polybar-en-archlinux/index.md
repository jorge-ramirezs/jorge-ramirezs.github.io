# Como Instalar Bspwm Polybar en Archlinux


## BSPWM
&lt;a href=&#34;https://github.com/baskerville/bspwm&#34; target=&#34;_blank&#34;&gt;bspwm&lt;/a&gt; es un manejador de ventanas en mosaico, la característica principal es que presenta las ventanas como hojas de un archivo binario.
&lt;a href=&#34;https://github.com/baskerville/bspwm&#34; target=&#34;_blank&#34;&gt;bspwm&lt;/a&gt; soporta el uso de múltiple monitores y es configurado y controlado a través de mensajes.

Si has estado siguiendo mi tutorial de como instalar archlinux en una máquina virtual, esta es la continuación.

### Instalación.

Vamos a instalar los paquetes necesarios para correr bspwm. En esta ocasión no voy a instalar un &#34;Display Manager&#34;, así que para iniciar el modo gráfico vamos a utilizar &#34;startx&#34;.

```bash
sudo pacman -S xorg-server xorg-xinit xf86-video-qxl bspwm sxhkd dmenu nitrogen picom terminator chromium arandr
```

### Configuración Inicial

Antes de iniciar el modo gráfico tenemos que configurar bspwm y sxhkd

Vamos a crear los directorios de configuración y copiar los archivos.

```bash
mkdir ~/.config/bspwm
mkdir ~/.config/sxhkd

cp /usr/share/doc/bspwm/examples/bspwmrc ~/.config/bspwm

cp /usr/share/doc/bspwm/examples/sxhkdrc ~/.config/sxhkd
```

Ahora vamos a cambiar la terminal por defecto en sxhkdrc por &#34;terminator&#34; que fue la teminal que instalamos.


Edita ~/.config/sxhkd/sxhkdrc
Y remplaza **urxvt** por **terminator** para que quede como sigue:

```text
# terminal emulator
super &#43; Return
    terminator
```

Luego tenemos que configurar **Xinit**.

Primero copiamos el archivo de ejempo xinitrc en nuestro directorio

```bash
cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

Luego removemos las ultimas 5 lineas y agregamos las siguientes dos por lo pronto:

```text
picom -f &amp;
exec bspwm
```

Como estamos en una máquina virtual tenemos que deshabilitar la siguiente opción en /etc/xdg/picom.conf

Cambia la opción &#34;vsync = true&#34; a 

```text
vsync = false
```

O si lo deseas puedes comentar la linea, eso hara que deshabilites la opción.

### Iniciar modo gráfico y configurar escritorio

Configurado bspwm y sxhkd, ahora sí, iniciamos el modo gráfico con:

```bash
startx
```

Verás que no aparece nada, pero en realidad es que necesitas al menos abrir una terminal, para verificar que la configuración es correcta y que bspwm esta funcionando presaiona

```
super &#43; Return
```
para iniciar terminator. (Donde &#34;super&#34; es la tecla de &#34;windows&#34; en tu teclado)

### Configurar resolucion de pantalla

En la terminal que acabas de abrir, teclea:

```bash
arandr
```
para arreglar la resolucion de la pantalla.

Para grabar la configuración selecciona el boton de &#34;guardar&#34; o &#34;save&#34; y nombra el arcvhivo &#34;display.sh&#34;

Lo siguiente es hacer este archivo ejecutable con:

```bash
chmod &#43;x ~/.screenlayout/display.sh
```

Luego agregalo a .xinitrc justo antes de &#34;picom -f &amp;&#34; para que quede como sigue:

```text
$HOME/.screenlayout/display.sh
picom -f&amp;
exec bspwm
```

Para salir de bspwm, presiona &#34;super &#43; alt &#43; q&#34;

E inicia el modo gráfico de nuevo con startx.

selecciona una imagen de wallpaper

inicia nitrogen y configura el wallpaper

agrega nitrogen a xinitrc

```text
$HOME/.screenlayout/display.sh
nitrogen --restore &amp;
picom -f&amp;
exec bspwm
```

Para &#34;arreglar&#34; el cursor debes instalar xorg-xsetroot

```bash
sudo pacman -S xorg-xsetroot
```

Agrega xsetroot -cursor_name left_ptr a .xinitrc para que quede como sigue.


```shell
#!/bin/sh

userresources=$HOME/.Xresources
usermodmap=$HOME/.Xmodmap
sysresources=/etc/X11/xinit/.Xresources
sysmodmap=/etc/X11/xinit/.Xmodmap

# merge in defaults and keymaps

if [ -f $sysresources ]; then
    xrdb -merge $sysresources
fi

if [ -f $sysmodmap ]; then
    xmodmap $sysmodmap
fi

if [ -f &#34;$userresources&#34; ]; then
    xrdb -merge &#34;$userresources&#34;
fi

if [ -f &#34;$usermodmap&#34; ]; then
    xmodmap &#34;$usermodmap&#34;
fi

# start some nice programs
if [ -d /etc/X11/xinit/xinitrc.d ] ; then
 for f in /etc/X11/xinit/xinitrc.d/?*.sh ; do
  [ -x &#34;$f&#34; ] &amp;&amp; . &#34;$f&#34;
 done
 unset f
fi

# Estas son las lineas que agregamos al archivo

$HOME/.screenlayout/display.sh
xsetroot -cursor_name left_ptr
nitrogen --restore &amp;
picom -f&amp;
exec bspwm
```

## Instalar Polybar

Polybar esta en los repositorios AUR, para eso debemos tener instalado &#34;yay&#34;.

Cuando instalamos archlinux, tambien instalamos &#34;git&#34; si no estás siguiendo esta serie de tutoriales, solo instala o verifica que tengas &#34;git&#34; instalado.

### Instalar &#34;yay&#34;

Para instalar &lt;a href=&#34;https://github.com/Jguer/yay&#34; target=&#34;_blank&#34;&gt;yay&lt;/a&gt; teclea los siguientes comandos:

```bash
cd ~
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
### Instalar Polybar

Para instalar Polybar corremos:

```bash
yay polybar
```

Acepta los defaults y selecciona &#34;N&#34; para que no muestre las diferencias en la pantalla.

### Configurar Polybar

Comienza por instalar el archivo de configuración de ejemplo.

```bash
install -Dm /usr/share/doc/polybar/config $HOME/.config/polybar/config
```

Instala los tipos de letra utilizados en la barra de ejemplo

```bash
sudo pacman -S xorg-fonts-misc
yay -S ttf-unifont siji-git
```

Ahora necesitamos crear un script para iniciar Polybar en

```
~/.config/polybar/launch.sh
```

Con el siguiente contenido:

```launch.sh
#!/usr/bin/env bash

# Terminate already running bar instances
killall -q polybar
# If all your bars have ipc enabled, you can also use 
# polybar-msg cmd quit

# Launch bar1 and bar2
echo &#34;---&#34; | tee -a /tmp/polybar1.log /tmp/polybar2.log
polybar example  &gt;&gt;/tmp/polybar1.log 2&gt;&amp;1 &amp;

echo &#34;Bars launched...&#34;
```

Después tenemos que hacer el script ejecutable con:

```bash
chmod &#43;x $HOME/.config/polybar/launch.sh
```

En este punto agregamos este script en nuestro archivo bspwmrc para que quede de la siguiente manera:

```shell
#! /bin/sh

sxhkd &amp;
$HOME/.config/polybar/launch.sh

bspc monitor -d I II III IV V VI VII VIII IX X

bspc config border_width         2
bspc config window_gap          12

bspc config split_ratio          0.52
bspc config borderless_monocle   true
bspc config gapless_monocle      true

bspc rule -a Gimp desktop=&#39;^8&#39; state=floating follow=on
bspc rule -a Chromium desktop=&#39;^2&#39;
bspc rule -a mplayer2 state=floating
bspc rule -a Kupfer.py focus=on
bspc rule -a Screenkey manage=off
```

Resultado Final:

![bspwm y polybar](/images/posts/bspwm-polybar/archVM-Polybar.png)


---

> Autor:   
> URL: http://localhost:1313/posts/como-instalar-bspwm-polybar-en-archlinux/  

