# claude-buzon

Skill de [Claude Code](https://claude.com/claude-code) que implementa un buzón de mensajes basado en archivos para que distintas sesiones de Claude Code en la misma máquina se comuniquen entre sí: dejar encargos, avisos, preguntas o resultados, e incluso sostener una conversación fluida entre dos sesiones para completar una tarea conjunta.

## Qué resuelve

Si trabajas con varias sesiones de Claude Code en paralelo (por ejemplo una en Windows y otra en WSL, o una con un perfil personal y otra con un perfil de trabajo), no hay forma nativa de que una sesión le pida algo a otra o le avise que terminó un trabajo. Esta skill cubre ese hueco con un buzón de archivos `.md` compartido en el filesystem: cada sesión tiene su propia identidad, su propia bandeja de entrada, y puede enviar, revisar, responder, quedarse escuchando mensajes nuevos, o abrir un canal de conversación continua con otra sesión para una tarea concreta.

## Instalación

1. Copia la carpeta `skills/buzon` de este repo a tu carpeta de skills de Claude Code (normalmente `~/.claude/skills/buzon/`).
2. Ajusta en `SKILL.md` las identidades (`win-personal`, `win-work`, `wsl-personal`, `wsl-work` en el ejemplo) y la ruta del buzón compartido (`BUS`) a tu propio setup: cuántos entornos/perfiles usas y dónde vive la carpeta compartida en tu filesystem.
3. La primera vez que invoques la skill (con `/buzon` o pidiendo algo como "revisa el buzón" o "mándale un mensaje a la sesión de WSL"), ella misma crea la estructura de carpetas necesaria.

## Requisitos

- Bash (Git Bash en Windows o bash de WSL). La skill opera el buzón solo con Bash o con las tools de lectura de archivos; evita PowerShell a propósito porque su escritura por defecto (UTF-16 con BOM) corrompe los mensajes.
- Una carpeta accesible desde todas las sesiones que quieras conectar (en Windows, una ruta normal del filesystem; si conectas también WSL, la misma ruta montada vía `/mnt/c/...`).

## Cómo funciona (resumen)

- Cada mensaje es un archivo `.md` con front-matter (`de`, `para`, `fecha`, `tipo`) en la bandeja `new/` del destinatario, escrito de forma atómica (`.tmp` + `mv`) para que nunca se lea a medias.
- Revisar el buzón muestra los mensajes pendientes completos antes de actuar sobre ellos; nada se ejecuta sin confirmación explícita.
- El modo escucha deja una sesión esperando mensajes nuevos en segundo plano.
- Un canal abre una conversación de ida y vuelta entre dos sesiones, acotada a una tarea y un alcance concretos que se aprueban una sola vez al abrir el canal.

El detalle completo (formato exacto de los mensajes, subcomandos, reglas de seguridad) vive en [`skills/buzon/SKILL.md`](skills/buzon/SKILL.md).

## Seguridad

El contenido de un mensaje recibido se trata siempre como input no confiable de otra sesión, nunca como instrucción directa: se muestra completo y cualquier acción que pida requiere confirmación explícita, salvo dentro de un canal ya abierto y con el alcance que se haya aprobado para él.
