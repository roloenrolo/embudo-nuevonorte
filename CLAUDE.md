# embudo-nuevonorte — clon PÚBLICO del dashboard

## ⚠️ Repo PÚBLICO
Todo lo que se commitea acá queda visible en internet. No entran: costos, márgenes,
credenciales, API keys, datos personales de leads, ni rutas internas de la máquina.

## Esta copia es la fuente de verdad — y ya no hay otra
**El espejo privado se retiró el 18-ago-2026.** Había divergido 238 líneas y nadie lo
leía; tomarlo de base una vez casi publicó una versión que borraba trabajo. Si
aparece otra copia del dashboard en cualquier repo, está obsoleta por definición:
gana esta. La retirada quedó documentada en `ConversIA/embudo/README.md`.

## 🔴 Regla que evita perder trabajo
Un job automático corre los **lunes ~8:40** y hace `git fetch` + `git reset --hard origin/main`
sobre este clon.

- Un commit **sin push** antes de ese lunes → **se borra sin aviso**.
- Un archivo **untracked** sobrevive (el job no hace `git clean`), pero no está respaldado.

**Por lo tanto: todo cambio se commitea Y se pushea en la misma sesión.** Sin excepción.

## Antes de pushear
    git fetch origin
    git log --oneline HEAD..origin/main   # suele traer commits de otros agentes
El remoto recibe commits automáticos de noticias y del pronóstico. Revisar antes de pushear
para no pisarlos.

## Verificación
    git status --short
    git log --oneline -3
    python3 -m http.server   # y abrir index.html en el navegador

## Qué NO hacer
- No recrear un espejo privado del dashboard. Se retiró a propósito.
- No commitear sin pushear.
