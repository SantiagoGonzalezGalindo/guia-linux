---
layout: default
title: Parte 2 · La terminal
nav_order: 3
---

# Parte 2 · La terminal
Estos dos pasos son independientes de la red y de todo lo demás. Sirven aunque nunca instales Tailscale, y puedes hacer uno sin el otro.

## Paso 7: la misma terminal en la laptop, el celular y el iPad

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

## Cómo salirte sin matar la sesión

Aquí es donde todo el mundo tropieza. El detach son dos pulsaciones separadas, no una combinación: presionas `Ctrl + b`, sueltas las dos teclas, y luego presionas `d` sola. Con eso solo se desconecta tu dispositivo y los demás siguen conectados.

Si presionas `Ctrl + d` estás mandando fin de entrada, que cierra la shell y mata la sesión completa. Cuando hay dos dispositivos enganchados, los dos salen disparados a la vez y parece que uno arrastró al otro. No es eso: mataste la sesión. Para que un resbalón no te vuelva a costar el trabajo, agrega `export IGNOREEOF=2` a tu `~/.bashrc` y Bash te va a exigir tres pulsaciones antes de cerrar.

Para cerrar una sesión a propósito, `tmux kill-session -t trabajo`.

## Dos ajustes que valen la pena

El prefijo `Ctrl + b` es incomodísimo en el teclado de un celular. Casi todos lo mueven a `Ctrl + a` con una línea en `~/.tmux.conf`:

```bash
set -g prefix C-a
```

Y si conectas la laptop y el iPad al mismo tiempo, vas a notar que la pantalla se encoge al tamaño del dispositivo más chico, porque ambos comparten la misma ventana. Para que cada uno se dimensione solo, el segundo se engancha con una sesión propia que comparte las ventanas:

```bash
tmux new-session -t trabajo -s ipad
```

## Paso 8: un historial que sí sirve

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

## Otras que valen la pena

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

## ble.sh, la sugerencia en gris

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
