# Super Dolores — SuperApp didáctica desarrollada con TDD

**Materia:** Desarrollo Móvil Integral · UTNG
**Profesor:** Anastacio Rodríguez García
**Actividad:** Manual paso a paso con TDD — *Super Dolores: una SuperApp completa* (mensajería, pagos, comercio y transporte en un solo ecosistema)
**Fecha de entrega:** 16 de noviembre de 2026
**Plataforma de evidencia:** Flutter 3.47.6 / Dart 3.13.5 (Linux Mint 22.3)

---

## 1. Descripción

Super Dolores es una **SuperApp de práctica** ambientada en Dolores Hidalgo: una sola identidad y un solo monedero para cuatro mini-apps (Mensajes, Monedero, Tienda y Viajes), más una mini-app de un tercero (Horóscopo) que sirve para demostrar el modelo de permisos.

El proyecto se construyó **siguiendo el ciclo TDD** (rojo → verde → refactor) propuesto en el manual:

- Primero las **pruebas** que describen la regla de negocio.
- Después el **código mínimo** para ponerlas en verde.
- Al final el **refactor** con la red de seguridad de las pruebas.

Todo el dominio es **Dart puro** (sin paquetes externos). El dinero se maneja en **centavos enteros**, la suma del dinero del sistema se conserva en cada operación, las operaciones críticas son **idempotentes** y los servicios se comunican por **eventos**, no por llamadas directas.

> ⚠️ Es una simulación didáctica: no mueve dinero real, no transporta personas y los datos viven en memoria.

---

## 2. Resultado de la actividad (resumen)

| Indicador | Resultado |
|---|---|
| Pruebas automatizadas | **104 pruebas / 4 archivos — 104 en verde, 0 fallos** |
| Cobertura de líneas | **96.1 %** (634/660 líneas) |
| Análisis estático (`flutter analyze`) | **No issues found!** |
| Compilación web (`flutter build web`) | **✓ Built build/web** |
| Recorrido manual de la app | 23 capturas en [`docs/capturas/`](docs/capturas) |
| Incidencias del manual detectadas | 1 (prueba inconsistente con la especificación) — corregida y documentada |
| Advertencias del analizador corregidas | 3 (import sin usar, API obsoleta, `const`) |

---

## 3. Cómo ejecutar el proyecto

### Requisitos

- Flutter 3.4 o superior (probado con **3.47.6**, Dart **3.13.5**).
- No requiere paquetes de terceros (`flutter pub get` solo resuelve el SDK).

### Pruebas

```bash
flutter pub get
flutter test                 # 104 pruebas
flutter test --coverage      # genera coverage/lcov.info
flutter analyze              # análisis estático
```

### Ejecutar la app

```bash
# Escritorio / móvil
flutter run

# Web (como se generaron las capturas)
flutter run -d web-server --web-port 8080
# o bien:
flutter build web --release
python3 -m http.server 8080 -d build/web
```

En la pantalla de inicio puedes **cambiar de persona** (Ana, Luis, Marta) con el botón superior derecho para ver las dos puntas de una conversación o de una transferencia, probar el **kill switch**, el **tablero de retención** y la **mini-app de un tercero**.

---

## 4. Arquitectura

```
super_dolores/
├── lib/
│   ├── dominio/                 # Reglas del negocio (Dart puro, sin Flutter)
│   │   ├── dinero.dart          # centavos, formato, porcentajes, lunesDe, masDias
│   │   ├── excepciones.dart     # errores de negocio y mensajes seguros
│   │   ├── eventos.dart         # bus de eventos + eventos del sistema
│   │   ├── monedero.dart        # libro de movimientos, idempotencia, límites
│   │   ├── mensajeria.dart      # chats y envío de dinero desde el chat
│   │   ├── comercio.dart        # tiendas, pedidos, comisión, custodia
│   │   ├── transporte.dart      # Haversine, tarifa, máquina de estados
│   │   ├── mini_app.dart        # registro, permisos, kill switch
│   │   ├── contexto.dart        # ContextoMiniApp (una puerta por mini-app)
│   │   ├── plataforma.dart      # composición de servicios y datos demo
│   │   ├── lealtad.dart         # puntos, niveles, bonos, racha, canje
│   │   ├── analitica.dart       # DAU/WAU/MAU, stickiness, cohortes
│   │   ├── notificaciones.dart  # topes, horario de silencio, opt-out
│   │   └── sugerencias.dart     # sugerencia por uso de los últimos 30 días
│   ├── presentacion/
│   │   ├── vm/                  # ViewModels (ChangeNotifier)
│   │   └── pantallas/           # Inicio, Chat, Monedero, Tienda, Viajes,
│   │                            # Métricas y Horóscopo (tercero)
│   └── main.dart                # la "concha" (shell) de la SuperApp
├── test/
│   ├── dominio_test.dart        # 30 pruebas
│   ├── servicios_test.dart      # 21 pruebas
│   ├── retencion_test.dart      # 31 pruebas
│   └── vms_test.dart            # 22 pruebas
├── docs/
│   ├── capturas/                # 23 capturas del recorrido manual
│   └── evidencias/              # logs de pruebas, cobertura y entorno
└── web/                         # shell web generada por Flutter
```

### Las cuatro ideas de diseño que se verifican

1. **Una identidad, un monedero, muchos servicios.** Todos los servicios usan el mismo `Monedero`; el saldo es siempre la suma del libro de movimientos, nunca un número editable.
2. **Servicios que no se conocen entre sí: eventos.** Comercio publica `PedidoCreado`; Lealtad, Analítica y Notificaciones reaccionan. Agregar un consumidor no obliga a tocar a los demás.
3. **Mini-apps con permisos mínimos.** Cada mini-app recibe un `ContextoMiniApp` atado a **un usuario** y valida el permiso en **cada** llamada. El Horóscopo solo tiene `identidad`: no puede leer saldo, chats ni cobrar.
4. **Kill switch.** Apagar Viajes desde el registro lo quita de la barra de navegación y bloquea sus llamadas, sin publicar una versión nueva.

---

## 5. Pruebas realizadas

### 5.1 Pruebas automatizadas (TDD)

Salida completa: [`docs/evidencias/flutter_test.txt`](docs/evidencias/flutter_test.txt)

```
00:05 +104: All tests passed!
```

| Archivo | Pruebas | Grupos que cubre |
|---|---:|---|
| `test/dominio_test.dart` | 30 | dinero y fechas · monedero (libro de movimientos) · bus de eventos · mensajería · mini-apps, permisos y kill switch |
| `test/servicios_test.dart` | 21 | comercio · transporte · integración entre servicios (invariantes de dinero, idempotencia, custodia) |
| `test/retencion_test.dart` | 31 | lealtad (puntos, niveles, bonos semanales, racha, canje) · analítica de retención · notificaciones éticas · sugerencias |
| `test/vms_test.dart` | 22 | InicioViewModel · MonederoViewModel · ChatViewModel · TiendaViewModel · ViajesViewModel · MetricasViewModel |
| **Total** | **104** | |

Ejemplos de reglas verificadas por las pruebas:

- `formatearPesos` con negativos, ceros y millones; `porcentaje` redondeando al más cercano.
- `lunesDe`: los siete días de una semana apuntan al mismo lunes; `masDias` cruza de mes.
- El saldo es la suma de movimientos; no se puede pagar más de lo que hay; montos inválidos y tope por operación.
- **Idempotencia**: el mismo evento o la misma clave de operación no cobra/premia dos veces.
- La compra reparte comisión y **la suma de todo el dinero del sistema no cambia**.
- Un viaje recorre `solicitado → aceptado → enCurso → finalizado`; los saltos ilegales fallan **sin tocar dinero**; desde `enCurso` no se puede cancelar.
- El Horóscopo (tercero) **no** puede leer el saldo, cobrar ni leer chats.
- El kill switch de Viajes bloquea las llamadas y se restaura al encenderlo.
- Lealtad: 1 punto por cada $10, multiplicadores por nivel (100/125/150 %), bonos semanales 50/100/200, racha +70 cada 7 días, canje 100 pts = $10.
- Analítica: DAU ≤ WAU ≤ MAU, tasas entre 0 y 1, D1 ≥ D7, y el generador sintético es reproducible con la misma semilla.
- Notificaciones: tope diario, horario de silencio y respeto al opt-out.

### 5.2 Cobertura

Resumen completo: [`docs/evidencias/coverage_resumen.txt`](docs/evidencias/coverage_resumen.txt) · datos crudos: [`docs/evidencias/lcov.info`](docs/evidencias/lcov.info)

| Capa | Cobertura |
|---|---:|
| `lib/dominio/` (14 archivos) | 96.4 % (484/502) |
| `lib/presentacion/vm/` (6 archivos) | 94.9 % (150/158) |
| **Total** | **96.1 % (634/660)** |

Archivos al 100 %: `analitica`, `comercio`, `dinero`, `lealtad`, `mensajeria`, `notificaciones`, `plataforma`, `sugerencias`, `transporte`, `tienda_vm`.

### 5.3 Análisis estático

Salida: [`docs/evidencias/flutter_analyze.txt`](docs/evidencias/flutter_analyze.txt)

```
Analyzing super_dolores...
No issues found! (ran in 1.4s)
```

Se corrigieron tres hallazgos del código del manual: un `import` sin usar en `pantalla_monedero.dart`, el parámetro obsoleto `value:` de `DropdownButtonFormField` (ahora `initialValue:`) y una declaración `final` que debía ser `const` en `metricas_vm.dart`.

### 5.4 Incidencia detectada en el manual (rojo → verde)

La prueba `Lealtad: 1 punto por cada $10 en compras y viajes` esperaba `5 + 8 = 13` puntos, pero el motor entregaba **63**: además de los 13 puntos por gasto, la regla 2 del propio manual otorga **+50 por usar 2 servicios distintos en la misma semana** (comercio y viajes).

Es decir, **la prueba contradecía la especificación**, no el código. Se documentó el caso, se ajustó la expectativa a `5 + 8 + 50` con un comentario explicativo y las 104 pruebas quedaron en verde. Este es exactamente el tipo de ambigüedad que TDD saca a la luz: primero se aclara la regla, luego se escribe la prueba.

## 6. Capturas del recorrido (pruebas de aceptación)

Capturas tomadas sobre el build web a 800 × 655 px. Algunos emojis se ven como recuadros porque el navegador usado no tenía la fuente de emojis instalada; no afecta a la funcionalidad ni a las pruebas.

### Prueba 1 · Inicio con identidad única

Inicio de Ana: saldo de $1,000, 0 puntos, nivel Bronce, racha 0 y sugerencia "Para ti".

![Inicio de Ana](docs/capturas/01_inicio_ana.jpg)

### Prueba 2 · Envío de dinero desde el Monedero

Ana envía $100 a Luis: el saldo baja de $1,000 a $900, aparece "Dinero enviado" y el movimiento queda registrado en el libro.

![Envío de dinero desde el Monedero](docs/capturas/02_monedero_envio.jpg)

### Prueba 3 · Compra en la Tienda

Catálogo de negocios de Dolores Hidalgo.

![Negocios de Dolores](docs/capturas/03_tienda_negocios.jpg)

Productos con precio y existencias disponibles.

![Productos de la tortillería](docs/capturas/04_tienda_productos.jpg)

Carrito con 2 kg de tortillas y 1 paquete de tostadas: total $91.00.

![Carrito con total de $91.00](docs/capturas/05_carrito.jpg)

Pedido `ped-5` confirmado: inventario descontado y saldo en $809.00.

![Pedido confirmado por $91.00](docs/capturas/06_pedido_confirmado.jpg)

### Prueba 4 · Viaje completo (máquina de estados)

Estimación del trayecto Jardín Principal → UTNG: 2.7 km por $57.20.

![Estimación del viaje](docs/capturas/07_viajes_inicio.jpg)

Estado `solicitado`, con la simulación del conductor y la opción de cancelar.

![Viaje solicitado](docs/capturas/08_viaje_solicitado.jpg)

Estado `aceptado`.

![Viaje aceptado](docs/capturas/09_viaje_aceptado.jpg)

Estado `enCurso`, listo para llegar y pagar.

![Viaje en curso](docs/capturas/10_viaje_en_curso.jpg)

Estado `finalizado` con cobro de $57.20 y saldo en $751.80.

![Viaje finalizado y cobrado](docs/capturas/11_viaje_finalizado.jpg)

### Prueba 5 · Chat y dinero dentro del chat

Conversación de Ana con Luis, con el envío de $100 ya reflejado como mensaje.

![Chat de Ana con Luis](docs/capturas/12_chat_ana_con_luis.jpg)

Mensaje de texto y envío de $50 sin salir de la conversación.

![Mensaje y envío de $50 en el chat](docs/capturas/13_chat_mensaje_y_envio.jpg)

La otra punta: Luis ve los mismos mensajes y ya no tiene mensajes sin leer.

![Chat visto por Luis](docs/capturas/14_chat_luis_recibido.jpg)

### Prueba 6 · Cambio de persona (dos puntas)

Luis recibe los $150 de Ana y muestra el badge de 3 no leídos.

![Inicio de Luis](docs/capturas/15_inicio_luis.jpg)

Ana, tras sus operaciones: 364 puntos, racha de 1 día y sugerencia actualizada.

![Inicio de Ana tras las operaciones](docs/capturas/16_inicio_ana_tras_operaciones.jpg)

### Prueba 7 · Tablero de retención

DAU 52, WAU 169, MAU 200, stickiness 26.0 %, D1 41.5 %, D7 40.4 % y 65.5 % de usuarios multiservicio, con la advertencia de datos simulados.

![Tablero de retención](docs/capturas/17_tablero_retencion.jpg)

Detalle de la comparación por servicios y la nota de que correlación no implica causalidad.

![Detalle del tablero de retención](docs/capturas/18_tablero_retencion_detalle.jpg)

### Prueba 8 · Permisos de la mini-app de un tercero

El Horóscopo intenta leer el saldo y es bloqueado por falta del permiso `monederoLeer`.

![Bloqueo al leer el saldo](docs/capturas/19_horoscopo_saldo.jpg)

Intenta cobrar y es bloqueado por falta del permiso `monederoPagar`.

![Bloqueo al intentar cobrar](docs/capturas/20_horoscopo_cobro.jpg)

Intenta leer los chats y es bloqueado por falta del permiso `mensajesLeer`.

![Bloqueo al leer los chats](docs/capturas/21_horoscopo_mensajes.jpg)

### Prueba 9 · Kill switch de Viajes

Viajes desaparece de la barra y de los chips de servicios; queda la opción de volver a encenderlo.

![Kill switch con Viajes apagado](docs/capturas/22_kill_switch_apagado.jpg)

### Prueba 10 · Canje de puntos de lealtad

Canje de 300 puntos: 364 → 64 puntos y saldo de $701.80 a $731.80, con el aviso "Canjeaste $30.00".

![Canje de puntos](docs/capturas/23_canje_puntos.jpg)

**Invariantes observados en el recorrido:** cada operación dejó rastro en el libro de movimientos; el saldo siempre fue la suma de esos movimientos; el dinero enviado apareció en la cuenta del destinatario; la compra descontó inventario; el viaje cobró al finalizar y no antes.

---

## 7. Lista de verificación del manual

- [x] `flutter test` pasa en verde (cuatro archivos de pruebas, 104 casos).
- [x] Mandar dinero en el chat cambia ambos saldos, deja un mensaje y avisa al destinatario.
- [x] Una compra reparte comisión y la suma total del dinero no cambia.
- [x] Un viaje recorre sus estados y los saltos ilegales fallan sin tocar dinero.
- [x] El horóscopo (tercero) no puede leer el saldo ni los chats.
- [x] Apagar Viajes lo quita de la barra y bloquea sus llamadas.
- [x] El tablero muestra DAU, WAU, MAU, stickiness y retención con la advertencia de datos sintéticos.

---

## 8. Hallazgos técnicos y decisiones

- **Dinero en enteros.** `$1,250.00` se guarda como `125000` centavos; los porcentajes se redondean con `(centavos × pct + 50) ~/ 100`.
- **Semana lunes–domingo** calculada con `DateTime(año, mes, día - desplazamiento)` para que el cambio de horario no mueva eventos de día.
- **Idempotencia en dos niveles:** por `id` de evento en Lealtad y por `clave` de operación en Monedero/Comercio. `TiendaViewModel` conserva la misma clave mientras el carrito no cambie; si el carrito cambia, se genera otra clave (operación distinta).
- **Dinero bajo custodia:** la plataforma retiene la comisión y paga al negocio/ conductor mediante eventos (`PagoRealizado`, `PedidoCreado`, `DineroRecibido`).
- **Métricas honestas:** `MetricasViewModel` genera 200 usuarios **sintéticos** con semilla fija (42). Es reproducible y sirve para aprender a leer un tablero, nunca para conclusiones reales.
- **Privacidad y permisos:** el `ContextoMiniApp` ata cada llamada a un usuario y a un conjunto de permisos; el aislamiento fuerte (sandbox) queda como reto avanzado.

---

## 9. Limitaciones (explícitas en el manual)

- No hay servidor real ni base de datos: todo vive en memoria y se pierde al cerrar.
- No hay inicio de sesión real (se cambia de persona con un botón de demostración).
- No hay cifrado de mensajes, moderación ni notificaciones push reales.
- No hay pagos reales, conductores reales, mapas ni rutas reales.
- No hay sandbox real para mini-apps de terceros.

Operar pagos, transporte y mensajería con usuarios reales exige licencias, seguros y cumplimiento legal (Ley Fintech, LFPDPPP, regulación local de transporte), tal como se explica en las rondas 14 y 15 del manual.

---

## 10. Evidencias incluidas

```
docs/
├── capturas/                  # 23 capturas del recorrido (JPG, 800×600)
└── evidencias/
    ├── flutter_test.txt       # salida completa de las 104 pruebas
    ├── flutter_analyze.txt    # análisis estático sin hallazgos
    ├── flutter_build_web.txt  # compilación web de producción
    ├── coverage_resumen.txt   # cobertura por archivo
    ├── lcov.info              # datos crudos de cobertura
    └── entorno.txt            # flutter --version y flutter doctor -v
```

---

## 11. Créditos

Actividad basada en el *Manual paso a paso con TDD — Super Dolores: una SuperApp completa*, de Anastacio Rodríguez García (Desarrollo Móvil Integral, UTNG). El código de dominio, pruebas y pantallas sigue el manual; se documentan aquí las correcciones y verificaciones realizadas durante la actividad.

> Una SuperApp no se mide por cuántos servicios tiene, sino por cuánto ayuda a las personas en su día a día y por la confianza que construye. 🏅
