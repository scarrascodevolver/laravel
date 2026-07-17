# Guía de aprendizaje — Repaso de Laravel

Cuaderno de apuntes que vamos llenando a medida que avanzamos con Claude.
Cada concepto va con su analogía + la explicación técnica + el código real
que usamos en este proyecto. La idea es poder repasar sin tener que buscar
en el historial del chat.

> Ver `CLAUDE.md` para el plan de fases completo y las convenciones del
> proyecto. Esta guía es el "cuaderno de clase"; CLAUDE.md son las reglas
> del juego para Claude.

---

## Fase 0-1 — Entorno y CRUD básico

### El orden de construir algo en Laravel — analogía de la casa

1. **Diseñar en papel** — pensar qué campos/tablas necesitas antes de tocar
   código (ej: Producto → nombre, descripcion, precio, stock, categoria).
2. **Pedir el kit vacío** — `php artisan make:model Producto -mcr` no
   construye nada real, solo te trae los archivos vacíos: el modelo, la
   migración y el controlador (con sus 7 métodos resource ya armados pero
   sin contenido). Como pedir a la ferretería el kit de construcción sin
   ensamblar.
   - `-m` → migración vacía (los cimientos sin excavar)
   - `-c` → controlador vacío
   - `-r` → que el controlador sea tipo *resource* (index, create, store,
     show, edit, update, destroy — las 7 "habitaciones" estándar de un CRUD)
3. **Excavar los cimientos de verdad** — completar la migración con los
   tipos de columna reales, pensando en que sea robusta a futuro (ej.
   `precio` como `decimal`, no `integer`, para no perder centavos).
4. **`php artisan migrate`** — acá se construye la casa físicamente en la
   base de datos. Antes de esto la tabla no existe.
5. **Modelo → `$fillable`** — ver más abajo.
6. **Rutas → Controlador → Vistas** — las puertas, el mayordomo, y la
   decoración, en ese orden.

### `$fillable` — el guardia de la lista VIP

Un formulario web normal solo tiene los campos que tú definiste. Pero un
atacante no necesita tu formulario — puede mandar la petición HTTP
directamente (con Postman, curl, etc.) agregando campos extra que tu
formulario nunca mostró.

```php
Producto::create($request->all());
```

Esto le dice a Eloquent "guarda TODO lo que llegó en la petición". Sin
protección, un campo colado (ej. `destacado=1` si existiera esa columna)
se guardaría también.

`$fillable` es la lista de invitados autorizados en la puerta: cualquier
campo que no esté en la lista, Eloquent lo ignora en silencio.

```php
protected $fillable = ['nombre', 'descripcion', 'precio', 'stock', 'categoria'];
```

**Pendiente:** todavía no lo agregamos al modelo `Producto` — hay que
definir primero las columnas reales en la migración.

---

## Glosario rápido

| Término | Analogía | Qué es en realidad |
|---|---|---|
| Migración | Planos + cimientos | Define el schema de una tabla en código versionable |
| Modelo Eloquent | El mayordomo de una tabla | Clase PHP que representa y gestiona una tabla |
| Controlador resource | Las 7 habitaciones estándar de una casa CRUD | Clase con index/create/store/show/edit/update/destroy |
| `$fillable` | Guardia con lista VIP | Whitelist de columnas permitidas en asignación masiva |

---

*(Se va actualizando en cada sesión — próximo tema: diseño de columnas
para la tabla `productos`.)*
