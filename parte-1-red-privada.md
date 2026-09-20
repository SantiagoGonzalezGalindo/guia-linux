---
layout: default
title: Parte 1 · La red privada
nav_order: 2
---

# Parte 1 · La red privada
Los pasos 1 a 6 montan la conexión entre la laptop y tu celular o tableta, funcione con iOS o con Android. Hazlos en orden, porque cada uno depende del anterior.

## Paso 1: instalar Tailscale en Linux

Tailscale es una VPN de malla: cifra el tráfico con WireGuard y conecta tus equipos punto a punto, sin que los archivos pasen por un servidor ajeno. Sus servidores solo sirven para que los dispositivos se encuentren.

El script oficial funciona en Fedora y derivados, Ubuntu, Debian y openSUSE:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

El `/install.sh` del final no es opcional. Si lo omites, `curl` descarga la página web de Tailscale y `sh` intenta ejecutar HTML, lo que produce el error `error de sintaxis cerca del elemento inesperado '<'`.

Si prefieres no ejecutar un script descargado de internet, en Fedora y Nobara puedes agregar el repositorio a mano. Ojo con la sintaxis: DNF5 cambió `--add-repo` por `addrepo --from-repofile=`.

```bash
sudo dnf config-manager addrepo --from-repofile=https://pkgs.tailscale.com/stable/fedora/tailscale.repo
sudo dnf install tailscale -y
```

Con cualquiera de los dos métodos, ahora enciendes el servicio y te autenticas:

```bash
sudo systemctl enable --now tailscaled
sudo tailscale up
```

La terminal te devuelve un enlace que empieza con `https://login.tailscale.com/a/`. Lo abres en el navegador, inicias sesión con Google, GitHub, Apple o Microsoft, y la terminal responde `Success`. Esa cuenta es la que van a usar todos tus dispositivos.

Último detalle de esta parte: por omisión solo root puede darle órdenes al servicio. Dale permiso a tu usuario para no escribir `sudo` cada vez.

```bash
sudo tailscale set --operator=$USER
```

## Paso 2: agregar tu celular o tableta

Descargas Tailscale de la App Store en cada dispositivo e inicias sesión con exactamente la misma cuenta que usaste en la laptop. iOS te va a pedir permiso para añadir una configuración de VPN; se lo das y metes el código del teléfono.

En cuanto estén dentro, cada uno recibe una dirección fija que empieza con `100.` y que no cambia nunca, ni cuando te mueves de red. Esa es la dirección con la que se hablan entre ellos, aunque el celular esté en datos móviles y la laptop en el wifi de la escuela.

Para ver quién está conectado y con qué dirección:

```bash
tailscale status
```

En Android es lo mismo: instalas Tailscale desde Google Play e inicias sesión con la misma cuenta. Cambian dos detalles menores. El permiso que te pide el sistema aparece como una conexión de VPN igual que en iOS, y los archivos que te lleguen por Taildrop caen solos en la carpeta de descargas, sin que tengas que aceptar nada. Del lado de Linux no cambia absolutamente nada.

## Paso 3: activar MagicDNS

MagicDNS te deja usar el nombre de la máquina en lugar de su dirección numérica. En vez de escribir algo como `100.111.33.91` escribes el nombre de tu laptop y ya. Ese nombre es el que Linux le puso al instalarse; lo ves con `hostname` o en la lista de `tailscale status`.

Se activa en la web, no en la terminal: entras al panel de administración de Tailscale, vas a la sección **DNS** y enciendes **MagicDNS**. Es un interruptor y aplica a todos tus dispositivos al instante.

Vale la pena hacerlo antes de configurar SSH, porque así guardas la conexión del celular por nombre y no tienes que tocarla nunca.

## Paso 4: Taildrop automático y sin "acceso denegado"

Taildrop es el AirDrop de Tailscale: mandas una foto o un PDF desde el celular con el menú de compartir, igual en iOS que en Android, eliges Tailscale, y el archivo viaja cifrado y directo a la laptop.

Primero hay que prenderlo en la cuenta, porque sigue en fase alfa y viene apagado. En el panel de administración, en **Settings → General**, activas **Send Files**. Si te saltas esto, el envío falla desde el celular y vas a pensar que el problema es Linux.

En Linux los archivos no caen solos en Descargas. `tailscaled` corre como root y deja lo que llega en una bandeja interna hasta que alguien la vacía:

```bash
sudo tailscale file get ~/Descargas/
```

Hacer eso a mano cada vez es insoportable, así que se automatiza con un servicio que escucha en segundo plano. La bandera `--loop` es la que lo deja esperando archivos nuevos en lugar de vaciar una vez y salir.

Aquí está el detalle que casi nadie menciona: si el servicio corre como root, los archivos llegan con dueño root y tu usuario no los puede borrar ni mover desde el explorador. La línea `User=` es lo que evita eso.

```bash
mkdir -p ~/Descargas/Taildrop

sudo tee /etc/systemd/system/taildrop-auto.service <<EOF
[Unit]
Description=Recibir archivos de Taildrop automaticamente
After=tailscaled.service
Wants=tailscaled.service

[Service]
Type=simple
User=$USER
ExecStart=/usr/bin/tailscale file get --loop /home/$USER/Descargas/Taildrop/
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now taildrop-auto.service
```

Para que ese servicio funcione corriendo como tu usuario, `tailscale set --operator=$USER` del paso 1 tiene que estar hecho. Sin eso el servicio arranca y muere en bucle porque no tiene permiso de hablar con el demonio.

Verificas que quedó vivo con `systemctl status taildrop-auto.service`: debe decir `active (running)` en verde. De esa pantalla se sale con la tecla `q`.

Queda encendido para siempre. `enable` lo programa para arrancar solo en cada encendido, así que puedes reiniciar sin volver a tocar nada.

## Paso 5: controlar la laptop desde el celular

Tailscale SSH maneja las llaves criptográficas y los permisos por ti, así que no tienes que generar nada ni abrir el puerto 22 a internet. Solo tus propios dispositivos pueden entrar.

```bash
sudo tailscale set --ssh
```

Usa `set` y no `up`. `tailscale up` reemplaza toda la configuración con las banderas que le pases en ese momento, así que si escribes `sudo tailscale up --ssh` pierdes el `--operator` que configuraste antes y se rompe Taildrop. `set` solo cambia lo que nombras.

Del lado del celular instalas Termius, que existe tanto en iOS como en Android (en iPhone también sirve Blink Shell), y creas un host nuevo con el nombre de tu laptop (el que devuelve `hostname`, gracias a MagicDNS), tu usuario de Linux y tu contraseña normal. La primera vez te pregunta si confías en la llave del servidor; aceptas.

Desde ahí ya tienes la terminal completa en la mano. Lo que más vas a usar:

| Comando | Qué hace |
| --- | --- |
| `sudo systemctl suspend` | Duerme la laptop antes de guardarla |
| `sudo poweroff` | La apaga por completo |
| `sudo reboot` | La reinicia |
| `top` | Ve qué está consumiendo CPU y RAM (se sale con `q`) |

Esto no es un truco de fiesta. Si algún día dejas la laptop colgada en la pantalla de inicio de sesión, entras por SSH desde el celular, deshaces lo que la rompió y la reinicias sin levantarte.

## Paso 6 (opcional): que suspender no reinicie la laptop

Esto solo aplica si al mandar `sudo systemctl suspend` la laptop se apaga de golpe y arranca de nuevo en lugar de dormirse. Es un problema de compatibilidad entre el kernel y el firmware de ciertas laptops modernas, no de Tailscale.

Primero revisas qué modos de reposo soporta tu equipo:

```bash
cat /sys/power/state
```

Si la respuesta incluye `freeze mem disk`, el hardware soporta todo. El modo `mem` es el sueño profundo clásico (S3) y es el que suele fallar. La alternativa es `s2idle`, el reposo moderno que usan las laptops nuevas:

```bash
sudo grubby --update-kernel=ALL --args="mem_sleep_default=s2idle"
sudo reboot
```

`grubby` es propio de Fedora y derivados. En Ubuntu o Debian el parámetro se agrega editando `GRUB_CMDLINE_LINUX_DEFAULT` en `/etc/default/grub` y corriendo `sudo update-grub`.

Sobre la hibernación, que es la otra pregunta obvia: en Fedora viene desactivada de fábrica y activarla requiere una partición swap igual o mayor a tu RAM, además de desactivar Secure Boot. Para llevar la laptop en la mochila no vale la pena. Suspendida gasta muy poca batería y despierta en un segundo con todo abierto; hibernada tarda casi lo mismo que un arranque completo y Tailscale tarda más en reconectarse.
