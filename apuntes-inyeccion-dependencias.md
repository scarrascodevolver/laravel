# Apuntes: Inyección de dependencias, type-hints y el service container de Laravel

> Apuntes de estudio — 27/08/2026

## 1. Type-hints (empezar por aquí, es la base de todo)

Un type-hint es **declarar el tipo de una variable al lado de su nombre**. Es la palabra que va ANTES de la variable:

```php
// SIN type-hint: $edad puede ser cualquier cosa
function saludar($nombre, $edad) { ... }
saludar(123, "hola");  // PHP lo acepta aunque esté al revés 🤷

// CON type-hints
function saludar(string $nombre, int $edad) { ... }
saludar(123, "hola");  // 💥 ERROR inmediato
saludar("Ana", 30);    // ✅
```

También funciona con clases propias:

```php
function handle(Notificador $notificador) { ... }
//              ↑↑↑↑↑↑↑↑↑↑↑ esto es el type-hint: "aquí solo entra un Notificador"
```

Y para lo que un método devuelve (tipo de retorno):

```php
public function calcularDeuda(Unit $unidad): Money
//                            ↑ tipo del parámetro    ↑ tipo de retorno
```

**Clave**: el type-hint no es una llamada ni un `new`. Es una etiqueta. Laravel las lee para saber qué inyectar.

## 2. Qué es la inyección de dependencias (la D de SOLID)

Una clase **no crea** las cosas que necesita: las **recibe desde afuera** (por el constructor).

```php
// ❌ SIN inyección — la clase crea su dependencia (acoplada, no testeable)
class RegistrarPago
{
    public function handle(array $datos)
    {
        $notificador = new NotificadorCorreo(); // siempre correo, sí o sí
        // ...registrar el pago...
        $notificador->enviar($pago);
    }
}

// ✅ CON inyección — la recibe por el constructor
class RegistrarPago
{
    public function __construct(
        private Notificador $notificador  // llega desde afuera
    ) {}

    public function handle(array $datos)
    {
        // ...registrar el pago...
        $this->notificador->enviar($pago);
    }
}
```

**Por qué importa:**
1. **Tests**: en el test le pasas un notificador falso y pruebas la lógica sin mandar correos reales. Con `new` adentro, imposible.
2. **Flexibilidad**: mañana notificas por WhatsApp cambiando qué inyectas, sin tocar `RegistrarPago`.

### Ojo con este error común

Recibir ≠ instanciar. En el constructor NO se hace `new`:

```php
// ❌ Esto NO es inyección (el new sigue adentro)
public function __construct()
{
    $this->notificador = new NotificadorCorreo();
}

// ✅ Esto SÍ (solo recibe y guarda)
public function __construct(Notificador $notificador)
{
    $this->notificador = $notificador;
}

// ✅ Atajo PHP 8 (constructor property promotion) — lo verás en todo Laravel moderno
public function __construct(
    private Notificador $notificador  // parámetro + propiedad + asignación en una línea
) {}
```

**Regla mnemotécnica**: si dentro de tu clase aparece `new` para un servicio, no hay inyección. El constructor es la puerta por donde las dependencias ENTRAN, no la fábrica donde se CREAN.

## 3. ¿Entonces quién hace el `new`? → El service container de Laravel

El `new` no desaparece: **se muda** de tus clases al container de Laravel, que es la pieza del framework cuyo trabajo es instanciar objetos.

Cuando llega una petición a `Route::post('/pagos', [PagoController::class, 'store'])`:

```
Petición HTTP
   ↓
Container: "necesito PagoController"
   ↓ lee los type-hints de su constructor/método
Container: "PagoController necesita RegistrarPago"
   ↓ lee los type-hints de SU constructor
Container: "RegistrarPago necesita Notificador"
   ↓ consulta los bindings → NotificadorCorreo
new NotificadorCorreo()
   → new RegistrarPago($notificador)
   → new PagoController(...)
   → ejecuta store()
```

Es **recursivo**: el container inspecciona los constructores (con reflection), lee las etiquetas de tipo, y construye el árbol completo de objetos. Tú declaras QUÉ necesitas, nunca CÓMO construirlo. Sin type-hint, el container no sabe qué construir:

```php
public function __construct($notificador) {}  // ¿un qué? No puede adivinar 💥
```

## 4. Interfaces y bindings

Si el type-hint es una clase concreta (`RegistrarPago`), el container le hace `new` directo. Si es una **interfaz** (`Notificador`), hay que decirle UNA VEZ qué implementación usar, en un Service Provider (`app/Providers/AppServiceProvider.php`):

```php
public function register(): void
{
    // "cuando alguien pida Notificador, entrégale un NotificadorCorreo"
    $this->app->bind(Notificador::class, NotificadorCorreo::class);
}
```

Cambiar toda la app de correo a WhatsApp = cambiar esa única línea.

En tests, se reemplaza el binding por un objeto falso:

```php
$this->mock(Notificador::class);  // prueba RegistrarPago sin enviar nada real
```

## 5. Cómo se ve en un controlador real

```php
public function store(RegistrarPagoRequest $request, RegistrarPago $accion)
{
    $pago = $accion->handle($request->validated());
    return new PagoResource($pago);
}
```

Nadie escribió `new RegistrarPago()`: Laravel vio el type-hint y lo entregó armado con todas sus dependencias.

## Resumen en 3 frases

1. **Type-hint** = escribir el tipo antes de la variable; es como le PIDES algo al container.
2. **Inyección de dependencias** = las clases reciben sus servicios por el constructor en vez de crearlos con `new`.
3. **Service container** = el único que hace `new`; lee los type-hints recursivamente y arma todo el árbol de objetos.

## Regla para el CLAUDE.md del proyecto

```markdown
- Inyección de dependencias vía el service container: los servicios/acciones se reciben
  por constructor o type-hint, nunca se instancian con `new` dentro de otras clases.
  Facilita tests con mocks.
```

## Para seguir estudiando
- Docs Laravel: Service Container → https://laravel.com/docs/container
- Docs Laravel: Service Providers → https://laravel.com/docs/providers
- Docs PHP: constructor property promotion (PHP 8)
