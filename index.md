---
layout: default
title: Guía de configuración de Linux
---

# Guía de configuración: red privada, terminal y acceso remoto en Linux

*Última modificación: 20 de septiembre de 2026 · [Santiago Gonzalez Galindo](https://github.com/SantiagoGonzalezGalindo)*

> **Aviso: lee la guía completa antes de ejecutar cualquier comando.** Varios pasos dependen de los anteriores y algunos modifican la configuración de arranque o de sesión del sistema. Ejecutarlos en desorden, a medias o copiando bloques sueltos puede dejarte sin entorno gráfico o sin poder iniciar sesión.

La guía va en cuatro partes. La primera monta la red privada entre la laptop y el celular, y sus pasos van en orden. La segunda mejora la terminal y es independiente: sirve aunque no instales nada de lo anterior. La tercera junta los errores y las confusiones con las que me topé, escritos por el mensaje que vas a ver en pantalla. La cuarta es una lista de comandos para tener a la mano.

## Contenido

- [Qué vas a tener al terminar](#qué-vas-a-tener-al-terminar)
- [Qué sentido tiene juntar todo esto](#qué-sentido-tiene-juntar-todo-esto)
- [Cómo leer esta guía](#cómo-leer-esta-guía)
- [Parte 1 · La red privada](#parte-1--la-red-privada)
  - [Paso 1: instalar Tailscale en Linux](#paso-1-instalar-tailscale-en-linux)
  - [Paso 2: agregar tu celular o tableta](#paso-2-agregar-tu-celular-o-tableta)
  - [Paso 3: activar MagicDNS](#paso-3-activar-magicdns)
  - [Paso 4: Taildrop automático y sin "acceso denegado"](#paso-4-taildrop-automático-y-sin-acceso-denegado)
  - [Paso 5: controlar la laptop desde el celular](#paso-5-controlar-la-laptop-desde-el-celular)
  - [Paso 6 (opcional): que suspender no reinicie la laptop](#paso-6-opcional-que-suspender-no-reinicie-la-laptop)
- [Parte 2 · La terminal](#parte-2--la-terminal)
  - [Paso 7: la misma terminal en la laptop, el celular y el iPad](#paso-7-la-misma-terminal-en-la-laptop-el-celular-y-el-ipad)
  - [Paso 8: un historial que sí sirve](#paso-8-un-historial-que-sí-sirve)
- [Parte 3 · Cuando algo sale mal](#parte-3--cuando-algo-sale-mal)
- [Parte 4 · Comandos para consultar](#parte-4--comandos-para-consultar)

## Qué vas a tener al terminar

Una red privada entre tu laptop y tu celular que funciona desde cualquier lugar: mandas archivos del celular a la laptop sin cables ni nube, y controlas la terminal de la laptop desde el celular aunque esté guardada en la mochila.

Esto se probó en Nobara Linux (derivado de Fedora, usa DNF5) con KDE Plasma sobre Wayland, más un iPhone. Si tu compañero trae Fedora, Bazzite o cualquier derivado, todo aplica igual. En Ubuntu o Debian solo cambia el gestor de paquetes. La parte del celular funciona igual en iPhone, iPad y Android.

No necesitas dejar la laptop prendida en casa ni tener IP fija ni abrir puertos en el módem. Tailscale resuelve eso solo.

Una advertencia antes de empezar: varios comandos que circulan por ahí para esto están mal escritos y te van a hacer perder media hora. Los de esta guía están verificados contra la documentación oficial de Tailscale. Al final hay una sección con los errores concretos para que no caigas en ellos.

## Qué sentido tiene juntar todo esto

Aquí hay dos cosas distintas que se tocan en un solo punto. Conviene entenderlo antes de instalar nada, porque así sabes qué te puedes saltar.

**Tailscale resuelve un problema de distancia.** Normalmente, para que tu celular hable con tu laptop tienen que estar en la misma red, o tú tienes que abrir puertos en el módem y dejar tu máquina expuesta a internet. Tailscale quita esa disyuntiva: le da a cada equipo una dirección fija privada y los conecta cifrado, estén donde estén. De ahí salen las tres cosas de la parte 1: mandarte archivos sin cables ni nube de por medio, entrar a la terminal de tu laptop desde el celular, y llamar a tus máquinas por su nombre en lugar de por un número. Más adelante, si montas un servidor de archivos o quieres enseñarle un proyecto a un compañero sin subirlo a ningún lado, la tubería ya está puesta.

**Lo de la parte 2 resuelve un problema de fricción.** tmux, Atuin, zoxide, fzf y ble.sh no te dejan hacer nada que antes fuera imposible; te quitan segundos y esfuerzo mental de cosas que haces cien veces al día. tmux hace que tu terminal deje de morir cuando cierras la ventana, aunque eso no significa que se guarde para siempre: la sesión vive mientras la laptop siga encendida y desaparece al apagarla o reiniciarla. Atuin convierte tu historial en algo que se puede consultar en lugar de una lista ciega. zoxide te quita el recordar rutas y fzf el recordar nombres. ble.sh te muestra lo que probablemente ibas a escribir.

**El punto donde se cruzan los dos grupos es tmux.** Sin él, entrar por SSH desde el celular te da una terminal nueva y vacía cada vez: sirve para apagar la laptop o revisar algo rápido, pero no para trabajar. Con él hay una sesión permanente a la que te asomas desde donde sea. Dejas algo compilando en la biblioteca, guardas la laptop, y desde el celular en el camión ves cómo va. Ese es el único caso donde la suma vale más que las partes.

El resto no se combina, y eso no es un defecto sino la razón por la que puedes instalarlas con confianza. Cada una se justifica sola: Atuin vale la pena aunque nunca uses zoxide, y fzf te sirve aunque jamás instales ble.sh. Como no dependen unas de otras, puedes agregarlas de una en una, quedarte con las que te acomoden y quitar cualquiera sin romper las demás. Lo decimos aquí para que no las instales todas de golpe pensando que solo funcionan juntas, porque entonces no vas a saber cuál te está ayudando y cuál te estorba. De hecho lo único que sí se cruza entre ellas es cuando dos quieren el mismo atajo de teclado, y eso queda resuelto en los pasos correspondientes.

## Cómo leer esta guía

Si es la primera vez que trabajas en la terminal, léete esto antes. Te va a ahorrar la mayoría de los tropiezos.

**Dónde se escriben los comandos.** En KDE la terminal se llama Konsole y la abres desde el menú de aplicaciones o con `Ctrl + Alt + T`. Todo lo que está en los recuadros grises de esta guía va ahí. Se pega con `Ctrl + Shift + V`, no con `Ctrl + V`, que en la terminal significa otra cosa.

**Las palabras que tienes que sustituir.** Cuando veas `cd carpeta`, `rm archivo` o `cp origen destino`, esas palabras son huecos, no comandos literales. Van reemplazadas por el nombre real de tu carpeta o tu archivo. Si escribes `man comando` tal cual, Linux te va a decir que no existe ninguna entrada para "comando", porque tomó la palabra en serio.

**Los símbolos que van a salir seguido.** El `~` es tu carpeta personal, o sea `/home/tu-usuario`; `~/Descargas` significa la carpeta Descargas que está dentro de ella. El `$USER` se reemplaza solo por tu nombre de usuario, así que ese sí lo copias tal cual. Y todo lo que va después de un `#` dentro de un recuadro es un comentario para ti, no forma parte del comando.

**Cuando un comando pide contraseña.** Los que empiezan con `sudo` corren como administrador y te la van a pedir. Al escribirla no se ve nada en pantalla, ni asteriscos ni puntos: es normal, escríbela completa y presiona Enter.

**Cuando algo no funciona.** Si te dice `instrucción no encontrada`, revisa primero que no tengas un error de dedo, porque es lo más común. Si está bien escrito, casi siempre es que falta instalar el programa o que la terminal sigue siendo la de antes; `exec bash` la recarga. La parte 3 tiene los casos concretos con los que nos topamos.

Y si nada de eso lo resuelve, pregúntale a la IA que uses. El truco está en cómo le pasas el error: copia de la terminal el bloque completo, incluyendo la línea que escribiste tú, no nada más el mensaje de error. Esa línea es la que le dice qué intentabas hacer.

```bash
usuario@laptop:~$ tailsca
bash: tailsca: instrucción no encontrada...
```

Con eso cualquier IA ve de inmediato que el comando está mal escrito. Si le mandas solo "instrucción no encontrada" va a tener que adivinar, y ahí es donde empieza a inventar soluciones. Aun así, revisa lo que te conteste antes de ejecutarlo: más de un comando de los que nos dictaron durante esta configuración simplemente no existía.

## Parte 1 · La red privada

Los pasos 1 a 6 montan la conexión entre la laptop y tu celular o tableta, funcione con iOS o con Android. Hazlos en orden, porque cada uno depende del anterior.

### Paso 1: instalar Tailscale en Linux

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

### Paso 2: agregar tu celular o tableta

Descargas Tailscale de la App Store en cada dispositivo e inicias sesión con exactamente la misma cuenta que usaste en la laptop. iOS te va a pedir permiso para añadir una configuración de VPN; se lo das y metes el código del teléfono.

En cuanto estén dentro, cada uno recibe una dirección fija que empieza con `100.` y que no cambia nunca, ni cuando te mueves de red. Esa es la dirección con la que se hablan entre ellos, aunque el celular esté en datos móviles y la laptop en el wifi de la escuela.

Para ver quién está conectado y con qué dirección:

```bash
tailscale status
```

En Android es lo mismo: instalas Tailscale desde Google Play e inicias sesión con la misma cuenta. Cambian dos detalles menores. El permiso que te pide el sistema aparece como una conexión de VPN igual que en iOS, y los archivos que te lleguen por Taildrop caen solos en la carpeta de descargas, sin que tengas que aceptar nada. Del lado de Linux no cambia absolutamente nada.

### Paso 3: activar MagicDNS

MagicDNS te deja usar el nombre de la máquina en lugar de su dirección numérica. En vez de escribir algo como `100.111.33.91` escribes el nombre de tu laptop y ya. Ese nombre es el que Linux le puso al instalarse; lo ves con `hostname` o en la lista de `tailscale status`.

Se activa en la web, no en la terminal: entras al panel de administración de Tailscale, vas a la sección **DNS** y enciendes **MagicDNS**. Es un interruptor y aplica a todos tus dispositivos al instante.

Vale la pena hacerlo antes de configurar SSH, porque así guardas la conexión del celular por nombre y no tienes que tocarla nunca.

### Paso 4: Taildrop automático y sin "acceso denegado"

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

### Paso 5: controlar la laptop desde el celular

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

### Paso 6 (opcional): que suspender no reinicie la laptop

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

## Parte 2 · La terminal

Estos dos pasos son independientes de la red y de todo lo demás. Sirven aunque nunca instales Tailscale, y puedes hacer uno sin el otro.

### Paso 7: la misma terminal en la laptop, el celular y el iPad

Una sesión de SSH normal te abre una terminal nueva e independiente, así que no puedes asomarte a la que dejaste abierta en el escritorio. Lo que sí puedes es hacer que la terminal deje de vivir dentro de la ventana y pase a vivir en un servidor aparte, al que se engancha quien quiera. Eso es `tmux`, y una vez que lo usas ya no vuelves atrás.

```bash
sudo dnf install tmux -y
```

En la laptop abres una sesión con nombre, y desde el celular te enganchas a esa misma:

```bash
tmux new -s trabajo      # en la laptop
tmux attach -t trabajo   # desde Termius
```

Lo que escribes en el celular se mueve en el monitor, en vivo. `tmux ls` te lista las sesiones que hay vivas.

Para que la ventana del escritorio también quede adentro sin que te acuerdes cada vez, entras a la configuración del perfil de Konsole y cambias su comando de `/bin/bash` a `tmux new-session -A -s main`. Desde ahí cada terminal que abras se engancha sola.

La otra ventaja es que la sesión sobrevive a que cierres Termius, te quedes sin señal o cierres la tapa. Puedes dejar una compilación corriendo, guardar la laptop y volver desde el iPad a ver cómo va. Lo que sí la mata es apagar o reiniciar; suspender no, porque la memoria se conserva.

#### Cómo salirte sin matar la sesión

Aquí es donde todo el mundo tropieza. El detach son dos pulsaciones separadas, no una combinación: presionas `Ctrl + b`, sueltas las dos teclas, y luego presionas `d` sola. Con eso solo se desconecta tu dispositivo y los demás siguen conectados.

Si presionas `Ctrl + d` estás mandando fin de entrada, que cierra la shell y mata la sesión completa. Cuando hay dos dispositivos enganchados, los dos salen disparados a la vez y parece que uno arrastró al otro. No es eso: mataste la sesión. Para que un resbalón no te vuelva a costar el trabajo, agrega `export IGNOREEOF=2` a tu `~/.bashrc` y Bash te va a exigir tres pulsaciones antes de cerrar.

Para cerrar una sesión a propósito, `tmux kill-session -t trabajo`.

#### Dos ajustes que valen la pena

El prefijo `Ctrl + b` es incomodísimo en el teclado de un celular. Casi todos lo mueven a `Ctrl + a` con una línea en `~/.tmux.conf`:

```bash
set -g prefix C-a
```

Y si conectas la laptop y el iPad al mismo tiempo, vas a notar que la pantalla se encoge al tamaño del dispositivo más chico, porque ambos comparten la misma ventana. Para que cada uno se dimensione solo, el segundo se engancha con una sesión propia que comparte las ventanas:

```bash
tmux new-session -t trabajo -s ipad
```

### Paso 8: un historial que sí sirve

El `Ctrl + R` que trae Bash de fábrica es torpe: busca de una línea a la vez y no te deja ver opciones. Atuin lo reemplaza por un buscador de pantalla completa sobre una base de datos que guarda, además del comando, en qué carpeta lo corriste, cuánto tardó y si falló. Escribes tres letras y filtra.

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh
```

Antes de correrlo vale la pena abrir esa dirección en el navegador y ver qué hace el script, como con cualquier cosa que descargas y ejecutas de internet.

Durante la instalación te pregunta si quieres crear una cuenta para sincronizar. Elige **Skip sync for now**. La sincronización sirve para compartir historial entre varias máquinas, y entrar por SSH desde el celular no cuenta como otra máquina: los comandos se ejecutan en la laptop, así que el historial ya es el mismo. Activarla solo significaría subir tu historial a un servidor sin ganar nada. El día que tengas un segundo equipo, `atuin register` lo resuelve.

Al final te pide agregar una línea a tu `~/.bashrc`. Después de hacerlo, recarga la shell:

```bash
exec bash
```

Si estás dentro de `tmux`, eso solo arregla el panel donde lo corriste; los demás siguen con la shell vieja hasta que los vuelvas a abrir.

Ahora `Ctrl + R` abre el buscador, y al escribir un comando te aparecen abajo los parecidos que ya usaste. Esa lista no se ejecuta sola: si presionas `Enter` corre lo que tú escribiste. Solo actúa si te mueves con las flechas. `Esc` la cierra.

Si más adelante instalas fzf, ten presente que él también quiere quedarse con `Ctrl + R`. No hay que hacer nada: Atuin se carga después y gana, que es lo que quieres porque su buscador es mejor. A fzf le quedan sus otros atajos intactos.

#### Otras que valen la pena

No las instales todas el mismo día. Cada una cambia cómo se siente la terminal y si las metes juntas no vas a saber cuál te sirve y cuál te estorba.

| Herramienta | Qué hace |
| --- | --- |
| `zoxide` | Aprende tus carpetas frecuentes y saltas a ellas escribiendo un pedazo del nombre |
| `fzf` | Buscador difuso que se engancha a casi todo, incluido `Ctrl + T` para elegir archivos |
| `starship` | Prompt que muestra la rama de git, el lenguaje del proyecto y si el último comando falló |
| `ble.sh` | Sugerencia en gris mientras escribes, estilo VS Code, que aceptas con la flecha derecha |

Las dos primeras están en los repositorios: `sudo dnf install zoxide fzf -y`. Las otras dos se instalan desde sus propios sitios.

Con zoxide no basta con instalarlo, y aquí es donde todos se atoran: `z` no es un programa, es una función que hay que inyectar en la shell. Si te dice `z: instrucción no encontrada`, falta esto:

```bash
echo 'eval "$(zoxide init bash)"' >> ~/.bashrc
exec bash
```

Tampoco adivina tus carpetas. Su base de datos se llena con tus visitas, no escaneando el disco, así que solo conoce aquellas donde ya estuviste desde que lo instalaste. Que una carpeta exista y salga en `ls` no le sirve de nada: tienes que entrar a cada una con `cd` la primera vez, y de ahí en adelante `z` ya te lleva desde donde sea. Después de una semana de trabajo normal ya tiene todo lo tuyo. Con `zoxide query -l` ves lo que ha aprendido.

Y ojo con la diferencia entre los dos comandos. `z practica1` salta directo. `zi` abre una lista interactiva de lo que ya conoce, así que con la base vacía te responde `no match found` aunque la carpeta esté ahí enfrente. El que vas a usar casi siempre es `z` a secas.

#### ble.sh, la sugerencia en gris

Es el que se parece a VS Code: mientras escribes, te aparece en gris el resto del comando que ya usaste antes y lo aceptas con la flecha derecha. Además colorea la línea conforme la escribes, así que un comando mal armado se ve mal desde antes de presionar Enter.

No está en los repositorios de Fedora. Se compila desde su código, y **clónalo estando en tu carpeta personal**, porque `git clone` descarga donde estés parado y es fácil terminar con la carpeta tirada dentro de un proyecto:

```bash
cd ~
git clone --recursive --depth 1 --shallow-submodules https://github.com/akinomyoga/ble.sh.git
make -C ble.sh install PREFIX=~/.local
echo 'source -- ~/.local/share/blesh/ble.sh' >> ~/.bashrc
exec bash
```

La instalación copia lo que necesita a `~/.local/share/blesh`, así que después puedes borrar la carpeta que clonaste con `rm -rf ~/ble.sh` y no pasa nada.

Si ya tienes fzf, hay un paso más que no es opcional: los dos pelean por el control del teclado y hay que usar la integración oficial. Se declara en `~/.blerc`, que es el archivo de configuración de ble.sh:

```bash
cat >> ~/.blerc <<'EOF'
ble-import -d integration/fzf-completion
ble-import -d integration/fzf-key-bindings
EOF
```

Una advertencia honesta antes de meterle mano: de todo lo que está en esta guía, este es el más invasivo. No es un programa que corre aparte, es una reescritura completa de cómo Bash maneja la línea de comandos. Convive bien con Atuin y fzf si haces lo de arriba, pero cuando algo se rompa raro en tu terminal, empieza sospechando de él. Instálalo un día que puedas romperlo sin costo, no la noche antes de entregar algo.

Se desactiva quitando su línea del `~/.bashrc`, y se desinstala del todo borrando `~/.local/share/blesh`.

## Parte 3 · Cuando algo sale mal

Todos estos salieron de intentarlo en vivo. Varios vienen de comandos que una IA inventó con mucha seguridad y que no existen.

**No reinicies `systemd-logind` con la sesión gráfica abierta.** El comando `sudo systemctl restart systemd-logind` deja el escritorio en un bucle de inicio de sesión: metes la contraseña, se apagan los monitores y regresas a la pantalla de login. Si tocas algo en `logind.conf`, reinicia la computadora completa en lugar del servicio. Y si ya caíste en el bucle, se sale con `Ctrl + Alt + F3` para entrar a una terminal de texto, o directo por SSH desde el celular.

**Cuidado con los comandos que una IA te dicta.** Además de la URL del instalador sin `/install.sh`, en nuestra sesión aparecieron `curl ... | /install.sh`, que busca un archivo en la raíz del sistema, y `--add-repo`, que DNF5 ya no acepta. Cuando un comando falle dos veces de la misma forma, busca la documentación oficial en lugar de pedir otra variante.

### Confusiones de la primera noche

Todas estas me pasaron a mí armando esto. Si te topas con alguna, aquí está la explicación.

**Se murió la sesión de tmux y se salieron los dos dispositivos.** No fue que uno arrastrara al otro: presionaste `Ctrl + d`, que cierra la shell y mata la sesión completa. El detach son dos pulsaciones separadas, `Ctrl + b`, soltar, `d`.

**Instalaste algo y el comando no existe.** Casi siempre es que la shell sigue siendo la de antes. Corre `exec bash` y vuelve a probar. Y si estás dentro de tmux, eso solo arregla el panel donde lo corriste; los demás siguen con la shell vieja.

**`z: instrucción no encontrada` aunque instalaste zoxide.** `z` no es un programa, es una función que hay que inyectar en la shell con `eval "$(zoxide init bash)"` en el `~/.bashrc`. Lo mismo aplica a fzf con `eval "$(fzf --bash)"`.

**`zoxide: no match found` con una carpeta que sí existe.** Zoxide solo conoce carpetas donde ya estuviste desde que lo instalaste; no escanea el disco. Entra una vez con `cd` y a partir de ahí ya te lleva.

**Atuin te ofrece crear una cuenta para sincronizar.** Dile que no. Entrar por SSH desde el celular no es otra máquina: los comandos corren en la laptop y el historial ya es el mismo. La sincronización solo sirve el día que tengas un segundo equipo con su propio Linux.

**Clonaste algo y apareció una carpeta rara dentro de tu proyecto.** `git clone` descarga donde estés parado, no pregunta. Muévete a `~` antes de clonar, o borra la carpeta sobrante después; si ya corriste `make install`, la instalación no depende de ella.

**El comando no se encuentra y lo escribiste bien.** Revísalo letra por letra antes de buscar en internet. `autin` por `atuin` me costó un rato.

**Los archivos que llegan por Taildrop no se dejan borrar.** Es el servicio corriendo como root. Recuperas el control con `sudo chown -R $USER:$USER ~/Descargas/Taildrop/` y evitas que se repita con la línea `User=` en el servicio, como está en el paso 4.

## Parte 4 · Comandos para consultar

Todo lo de aquí abajo funciona igual desde la laptop o desde Termius en el celular.

### Lo básico de la terminal

| Comando | Qué hace |
| --- | --- |
| `ls -la` | Lista todo, incluidos los archivos ocultos, con permisos y tamaños |
| `cd carpeta` | Entra a una carpeta (`cd ..` sube una, `cd` sola te lleva a tu casa) |
| `pwd` | Dice en qué carpeta estás parado |
| `mkdir -p ruta/carpeta` | Crea una carpeta, y las intermedias si faltan |
| `cp -r origen destino` | Copia (la `-r` es para carpetas) |
| `cp README.md ~/Descargas/` | Copia un archivo a otra carpeta |
| `cp README.md README.bak` | Copia con otro nombre, o sea un respaldo |
| `cp -r Practica1 Practica1-respaldo` | Copia una carpeta entera |
| `mv origen destino` | Mueve, y también sirve para renombrar |
| `rm -r carpeta` | Borra, sin papelera y sin preguntar |
| `cat archivo` | Escupe el contenido de un archivo |
| `less archivo` | Lo abre para leerlo con calma (se sale con `q`) |
| `nano archivo` | Lo edita (`Ctrl + O` guarda, `Ctrl + X` sale) |
| `grep -r "texto" .` | Busca un texto dentro de todos los archivos de aquí abajo |
| `find . -name "*.pdf"` | Busca archivos por nombre |
| `chmod +x script.sh` | Le da permiso de ejecución a un archivo |
| `df -h` | Cuánto espacio le queda a tus discos |
| `du -sh carpeta` | Cuánto pesa una carpeta |
| `sudo dnf update -y` | Actualiza todo el sistema |

Tres costumbres que ahorran más tiempo que cualquier comando: `Tab` autocompleta nombres de archivos y carpetas a medias, `Ctrl + C` mata lo que esté corriendo, y `man` abre el manual de lo que sea. Por ejemplo, `man ls` te muestra todo lo que puedes hacer con ese comando.

### Borrar sin arrepentirte

En la terminal no hay papelera. Lo que borras con `rm` desaparece del disco en ese instante: no está en la papelera de KDE y no hay Ctrl+Z. Por eso `rm -rf` es el comando más peligroso de esta guía; si le pasas la ruta equivocada se lleva todo lo que haya debajo sin un solo aviso.

```bash
rm archivo.txt              # borra un archivo
rm -r Practica1-respaldo    # borra una carpeta con todo lo de adentro
rm -rf ~/ble.sh             # lo mismo, sin preguntar nada
```

La `-r` es para carpetas, igual que en `cp`. La `-f` fuerza: no pide confirmación ni se queja si algo no existe.

Dos costumbres que ahorran sustos: corre `ls` sobre la ruta antes de borrarla para confirmar que es la que crees, y mientras agarras confianza usa `rm -i`, que pregunta archivo por archivo.

Si prefieres poder arrepentirte, instala `trash-cli`. Manda las cosas a la papelera de KDE, de donde sí se recuperan:

```bash
sudo dnf install trash-cli -y
trash-put carpeta
```

### Borrar una carpeta pero conservar lo de adentro

No hay un comando que haga las dos cosas. Primero sacas el contenido y después borras la carpeta vacía.

```bash
mv Practica1/* .
rmdir Practica1
```

El `*` significa todo lo que hay dentro y el `.` es la carpeta donde estás parado, así que la primera línea sube todo un nivel. `rmdir` es el hermano seguro de `rm`: solo borra carpetas vacías y se niega si queda algo, que es justo lo que quieres aquí.

Ojo con un detalle que muerde: el `*` no toma los archivos ocultos, los que empiezan con punto. Si la carpeta tiene un `.git` o un `.env` se quedan atrás y `rmdir` te va a reclamar. Para esos casos:

```bash
mv Practica1/* Practica1/.* .
```

Se va a quejar de `.` y `..`, que no puede mover. Es normal. Confirma con `ls -la` que ya no quedó nada antes de borrar.

### Red

| Comando | Qué hace |
| --- | --- |
| `tailscale status` | Lista tus dispositivos, sus direcciones y cuáles están conectados |
| `tailscale ip -4` | Muestra la dirección de esta máquina |
| `sudo tailscale up` | Reconecta si te desconectaste |
| `sudo tailscale down` | Desconecta esta máquina de la red |
| `tailscale file cp archivo.pdf celular:` | Manda un archivo de la laptop al celular |
| `sudo tailscale file get ~/Descargas/` | Vacía la bandeja de Taildrop a mano |

### Control remoto

| Comando | Qué hace |
| --- | --- |
| `sudo systemctl suspend` | Duerme la laptop antes de guardarla |
| `sudo poweroff` | La apaga |
| `sudo reboot` | La reinicia |
| `top` | Qué está consumiendo CPU y RAM (se sale con `q`) |
| `w` | Quién está conectado y en qué terminal |
| `systemctl status taildrop-auto.service` | Revisa que el servicio de Taildrop siga vivo |

### tmux

| Comando | Qué hace |
| --- | --- |
| `tmux new -s trabajo` | Crea una sesión con nombre |
| `tmux attach -t trabajo` | Se engancha a una sesión existente |
| `tmux ls` | Lista las sesiones vivas |
| `tmux new-session -t trabajo -s ipad` | Se engancha desde un segundo dispositivo con su propio tamaño |
| `tmux kill-session -t trabajo` | Cierra una sesión a propósito |
| `Ctrl + b`, soltar, `d` | Sale sin matar nada |

### Lo que instalamos

| Comando | Qué hace |
| --- | --- |
| `Ctrl + R` | Abre el buscador de historial de Atuin |
| `atuin search "texto"` | Busca en el historial sin salir de la línea |
| `atuin stats` | Te dice qué comandos usas más |
| `z proyecto` | Salta a una carpeta frecuente escribiendo un pedazo del nombre (zoxide) |
| `zi` | Lo mismo pero eligiendo de una lista |
| `htop` | Monitor de sistema con colores (`sudo dnf install htop -y`) |

> **Detalle que confunde:** fzf también quiere quedarse con `Ctrl + R`, pero Atuin llega después y gana. Es lo que quieres, porque el de Atuin es mejor.

### Cuando algo no arranca

| Comando | Qué hace |
| --- | --- |
| `sudo systemctl daemon-reload` | Recarga los servicios después de editar uno |
| `sudo systemctl restart <servicio>` | Reinicia un servicio |
| `journalctl -u <servicio> -n 50` | Últimas 50 líneas del registro de un servicio |
| `exec bash` | Recarga la shell sin cerrar la ventana |
| `echo $XDG_SESSION_TYPE` | Dice si estás en Wayland o X11 |

---

Esta guía se va a seguir actualizando conforme aparezcan cosas nuevas o cambien los comandos, así que vale la pena volver al enlace en lugar de guardarte una copia. Aun así, no la tomes como la última palabra: los proyectos cambian de sintaxis entre versiones y lo que aquí funcionó puede quedarse viejo. Ante cualquier duda, revisa la documentación oficial de cada herramienta y compara con lo que diga la de tu distribución.

Fuentes: [Tailscale](https://tailscale.com/docs), [tmux](https://github.com/tmux/tmux/wiki), [Atuin](https://docs.atuin.sh), [zoxide](https://github.com/ajeetdsouza/zoxide), [fzf](https://github.com/junegunn/fzf), [ble.sh](https://github.com/akinomyoga/ble.sh).
