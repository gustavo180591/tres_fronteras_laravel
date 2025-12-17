# Tres Fronteras — Sistema de Fotos + Pedidos (idea mejorada)

> Documento de diseño funcional + técnico (base para implementar con Laravel).

---

## 1) Contexto del torneo (fixture como fuente de verdad)

El torneo “Tres Fronteras” organiza categorías **2014–2017**, con **fase de grupos (Fecha 1–3)** y luego **playoffs Oro/Plata** (cuartos, semis y finales).  
El fixture del ejemplo trae cada partido con **Categoría, Grupo, Cancha y Horario** (ej.: *Viernes 12/12 Fecha 1*, distintos partidos por categoría) y luego define eliminatorias con “1ero Grupo A vs 2do Grupo B”, etc. fileciteturn0file0L1-L5 fileciteturn0file0L81-L88 fileciteturn0file0L105-L114

**Principio clave:** el sistema tiene que modelar el fixture para que el pedido de fotos sea **rápido y sin errores** (evitar olvidar padres/pedidos).

---

## 2) Objetivo del sistema

1) **Pantalla “Padres/Clientes” (kiosco o tablet):** Carla busca el partido (equipo vs equipo, cancha, hora, categoría), navega fotos, marca “me gusta”, arma su carrito y confirma el pedido.

2) **Pantalla “Fotógrafos/Impresión” (operativa):** recibe pedidos en tiempo real, imprime/entrega, controla pago y estado.

3) **Biblioteca de fotos por partido:** fotos cargadas/ordenadas por fecha+partido para que encontrarlas sea inmediato.

---

## 3) Roles (muy simple, pero claro)

- **Cliente (Padre/Madre):** crea pedidos, selecciona fotos, puede editar antes de confirmar.
- **Operador (Foto/Impresión):** valida pedido, cobra, imprime/entrega, marca estados.
- **Admin:** gestiona fixture, precios, dispositivos/kioscos, reportes.

---

## 4) Concepto central para no olvidarse de nadie: “Ticket de atención + pedido”

En cancha, el problema real no es solo “carrito de fotos”: es que hay **cola, urgencia y ruido**.  
La solución robusta es separar:

### A) Ticket (Atención)
- Identifica a la persona en el momento: **Nombre + Apellido + Teléfono**.
- Le asigna un **código corto** (ej. TF-482) o QR.
- “Bloquea” una sesión de selección por un tiempo (ej. 30 min).

### B) Pedido (Order)
- Lo que finalmente se cobra/entrega: items, total, pago, entrega.
- Puede nacer vacío y luego llenarse con las fotos que “likea”.

> Beneficio: aunque Carla se vaya y vuelva, el ticket existe, y no se pierde.

---

## 5) Estados (los tuyos, pero con dos capas)

### 5.1 Estado del pedido (Order Status)
- `PENDIENTE` (creado, aún sin pagar o sin producir)
- `EN_PRODUCCION` (ya confirmado, listo para imprimir/preparar)
- `IMPRESO` (si aplica)
- `LISTO_PARA_ENTREGAR` (digital listo / físico impreso)
- `ENTREGADO`
- `CANCELADO`

### 5.2 Estado de pago (Payment Status)
- `NO_PAGADO`
- `PAGADO_PARCIAL`
- `PAGADO`
- `REEMBOLSADO` (si algún caso raro)

**Regla de negocio:** si `payment_status != PAGADO` => **no permitir “ENTREGADO”** (tu regla de “si no pagó no se entrega”, formalizada).

---

## 6) Tipos de entrega (Delivery Type)

- `FISICO` (foto impresa)
- `DIGITAL` (archivo JPG/PNG enviado por WhatsApp o link)
- `PDF` (pack/hoja en PDF)
- `COMBO` (físico + digital)

Cada item del pedido puede tener su tipo, o podés definirlo a nivel pedido con “extras”.

---

## 7) La parte más crítica: identificar la foto sin confusión

Tu idea de “nombre de archivo” funciona, pero en cancha se equivoca fácil. Mejor: **ID visible + metadata + búsqueda**.

### 7.1 Identidad de foto (Photo)
- `photo_id` interno (UUID o numérico)
- `code` visible tipo: `C10-2014-12-12-F1-000123`
  - (Cancha 10, Cat 2014, fecha, fecha_nro/instancia, correlativo)
- `match_id` (partido)
- `taken_at`
- `photographer_id` (opcional)
- `file_path` / `storage_key`
- `thumb_path`
- `tags` (ej.: “nro10”, “cantera”, “mitre”, “gol”, etc. opcional)

### 7.2 Cómo “Carla sabe cuál quiere”
En la pantalla de cliente:
- Por defecto ve **galería del partido** (ya filtrado).
- Cada foto muestra:
  - miniatura
  - el `code` visible
  - botón ❤️ (me gusta / agregar)
- Búsqueda adicional (si ella te dicta):
  - input para pegar el **code** o parte del code
  - o filtros rápidos: “N° camiseta”, “equipo”, “jugador” (si usás tags)

**Resultado:** Carla no necesita “nombre de archivo” técnico; usa un código simple.


### Filtro por equipo (nuevo)
Además de entrar por “partido”, el cliente puede elegir primero un **equipo** y ver **todas las fotos asociadas a ese equipo** (en uno o varios partidos). Luego puede refinar por:
- **Categoría** (2014–2017)
- **Partido jugado** (equipo vs equipo + cancha + hora)
- **Etapa/Fecha** (Fecha 1/2/3, Cuartos, etc.)


---

## 8) Flujo completo (simple y rápido)

### 8.1 Crear ticket
1. Operador crea Ticket: nombre, apellido, teléfono.
2. Elige Partido (desde el fixture) o lo deja “por definir”.
3. Se genera código/QR del ticket.

### 8.2 Selección de fotos (kiosco / tablet)
1. Carla selecciona (orden flexible):
   - **Categoría** (2014–2017)
   - **Equipo** (opcional, para filtrar toda su galería)
   - **Etapa/Fecha** (Fecha 1/2/3, Cuartos, etc.)
   - **Partido** (Equipo vs Equipo, Cancha, Hora) — viene del fixture.

   > Si elige equipo primero: el sistema muestra partidos donde ese equipo participó, y al entrar a un partido muestra solo su galería. fileciteturn0file0L1-L24
2. Se carga la galería del partido.
3. Carla marca ❤️ en las que quiere.
4. En un panel lateral se arma el “contenedor” (carrito temporal):
   - listado
   - cantidad
   - subtotal
   - botón “Quitar” (si se arrepiente)

### 8.3 Confirmación del pedido
1. Carla elige tipo de entrega: físico/digital/pdf.
2. Selecciona cantidad por foto (ej. 3 copias de la misma).
3. El sistema calcula total automáticamente (ej. $3000 c/u).
4. Confirma => se crea Order con items.

### 8.4 Operación
1. Operador ve el pedido en “PENDIENTE/EN_PRODUCCION”.
2. Cobra (marca pago).
3. Imprime/arma digital.
4. Marca `IMPRESO` / `LISTO_PARA_ENTREGAR`.
5. Entrega => `ENTREGADO`.

---

## 9) Modelo de datos (mínimo viable, bien pensado)

### Torneo
- `tournaments`
- `categories` (2014–2017)
- `teams`
- `venues` (Estadio/Auxiliar/otras canchas)
- `matches`
  - category_id
  - stage (grupos / cuartos / semi / final)
  - group_name (A/B/C/D o null)
  - match_datetime
  - venue_id + field_number (cancha)
  - team_a_id / team_b_id (o placeholders “1ero Grupo A”, etc. para playoffs) fileciteturn0file0L81-L98

### Fotos
- `photos`
  - match_id
  - code
  - storage_key
  - taken_at
- `photo_tags` (opcional)

### Atención / Pedidos
- `customers` (nombre, apellido, teléfono)
- `tickets` (customer_id, code, status, notes)
- `orders` (ticket_id, status, payment_status, delivery_type, totals)
- `order_items` (order_id, photo_id, qty, unit_price, line_total)

---

## 10) Pantallas (UX)

### 10.1 Cliente (kiosco)
- Paso 1: Elegir categoría
- Paso 2: Elegir equipo (opcional)
- Paso 3: Elegir fecha/etapa (Fecha 1/2/3, Cuartos, etc.)
- Paso 4: Elegir partido (cards con: “Cantera vs Mitre — 10:00 — Cancha 10”)
- Paso 5: Galería
  - Grid grande, scroll fluido
  - ❤️ para agregar
  - Panel lateral “Tu selección” con totales
- Paso 5: Confirmar
  - delivery
  - cantidades
  - total
  - Confirmar / Volver

### 10.2 Operador (admin operativo)
- Tab “Pedidos en vivo”
  - filtros: `PENDIENTE`, `EN_PRODUCCION`, `LISTO_PARA_ENTREGAR`
  - buscador por: teléfono / ticket / apellido
- Vista pedido
  - items + miniaturas
  - botón: “Marcar pagado”
  - botón: “Impreso”
  - botón: “Entregado”
  - acciones rápidas para evitar olvidos

---

## 11) Reglas de negocio (claras)

- No se permite `ENTREGADO` si `payment_status != PAGADO`.
- Un ticket puede tener **varios pedidos** (Carla vuelve y pide otra tanda).
- La selección “❤️” **no es el pedido**: es un “draft” (temporal) hasta confirmar.
- Cada foto puede imprimirse N veces (qty).
- Si el fixture cambia, el match_id sigue siendo la referencia (no dependas del texto).

---

## 12) Implementación en Laravel (alineado a tu markdown)


### Cómo se logra el filtro por equipo (backend)
Como cada **foto pertenece a un partido** (`photos.match_id`), y cada partido tiene `team_a_id` y `team_b_id`, el filtro por equipo es directo:

- “Fotos del equipo X” = fotos de partidos donde `team_a_id = X` **o** `team_b_id = X`.
- Si además se filtra por categoría/fecha: se agrega esa condición sobre `matches`.

**Índices recomendados**:
- `photos(match_id)`
- `matches(category_id, match_datetime)`
- `matches(team_a_id)`, `matches(team_b_id)` (o un índice compuesto según tu motor)


Tu `laravel.md` ya marca principios útiles:
- Mantener lógica fuera de rutas (MVC), usar controladores. fileciteturn0file1L17-L24
- Validar siempre en servidor con `request()->validate()`. fileciteturn0file1L33-L35
- Para tareas lentas (ej. generar PDF, empaquetar fotos digitales) usar colas con `Job::dispatch()` y worker. fileciteturn0file1L36-L38

### Módulos sugeridos (Controllers)
- `MatchController` (listar/filtrar partidos, por categoría/equipo/fecha)
- `PhotoController` (galería por partido, búsqueda por code)
- `TicketController` (crear ticket, buscar por teléfono)
- `OrderController` (confirmar pedido, ver estado)
- `OpsController` (pantalla operativa, cambiar estados)

---

## 13) Lo que necesitamos definir (siguiente paso)

Para cerrar el diseño (sin vueltas), definamos estas decisiones:

1) **Dónde se guardan las fotos**: disco local del server, S3/MinIO, o carpeta organizada (por torneo/fecha/match).  
2) **Cómo se entrega lo digital**: WhatsApp manual, link temporal, QR con descarga.  
3) **Precio**: fijo $3000 o por tipo (físico vs digital).  
4) **Dispositivo kiosco**: 1–2 tablets o una PC con pantalla táctil.

---

## 14) Próximo entregable (cuando me digas “dale”)

- Un **diagrama de flujo** (BPMN simple) + **ERD** (tablas y relaciones).
- Migraciones + modelos Eloquent + rutas `Route::resource`.
- UI base (Blade + Tailwind o lo que decidas).
