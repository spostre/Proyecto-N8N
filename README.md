# AgendaBot (Telegram + n8n + Google Sheets)

AgendaBot es un bot conversacional por Telegram, construido con **n8n** y **Google Sheets** como base de datos, para **agendar citas, gestionar tareas, hábitos y listas** mediante **menús numéricos** y flujos guiados (wizard).
 El proyecto cumple el enfoque “sin plataformas de pago / sin APIs que exijan tarjeta” y mantiene la interacción clara: **el usuario siempre elige con números, el bot explica, sugiere y siempre ofrece salida (volver/cancelar)**.

------

## Artefactos del proyecto

- **Workflow n8n (producción)**: 
- **Especificación funcional / reglas del bot**: converted_text

------

## Funcionalidades

### Navegación por menús (100% numérica)

- Menú principal con secciones: **Agenda, Tareas, Recordatorios, Hábitos, Listas, Reportes, Configuración, Administrador**. 
- Validación global de opción inválida y retorno seguro a menús. 

### Registro y sesión

- Si el usuario es nuevo: solicita nombre y lo registra en **USUARIOS**.
- Manejo de “pantalla actual” y progreso de paso con **SESSIONS** para que el flujo sea continuable.

### Agenda (Citas) — Wizard de 6 pasos

Flujo guiado para agendar una cita:

1. Fecha (valida formato y que no sea pasada)
2. Hora
3. Nombre del cliente
4. Motivo
5. Canal (presencial/virtual/llamada)
6. Confirmación y guardado / cancelar

Incluye opción **9** para cancelar/volver cuando aplica.

------

### Tareas

Creación guiada y guardado en **TAREAS** (título, prioridad, fecha objetivo).



### Hábitos

Creación guiada y guardado en **HABITOS** (nombre, frecuencia, hora recordatorio).



### Listas

Creación guiada y guardado en **LISTAS** (nombre, tipo).



### Reportes

Creacion de reportes guiada y guardado en **Reportes** (nombre).



### Configuración

Creacion de configuracion.



### Administracion

Creacion de Administracion junto a Permisos por Rol (Google Sheets).

------



## Stack y restricciones

**Stack permitido**

- Telegram (interfaz conversacional)
- n8n Community Edition (automatización y lógica)
- Google Sheets (almacenamiento) 

**No permitido**

- n8n Cloud de pago
- APIs que requieran tarjeta de crédito
- Entrenamiento de modelos / embeddings / RAG 

------



## Modelo de datos (Google Sheets)

El documento (por ejemplo “AgendaBot_DB”) debe contener estas pestañas y columnas (encabezados en fila 1):

- **CITAS**: `id_cita, fecha, hora, nombre, motivo, canal, estado, creado_por, timestamp_creacion`
- **TAREAS**: `id_tarea, titulo, prioridad, estado, fecha_objetivo, creado_por`
- **HABITOS**: `id_habito, nombre, frecuencia, hora_recordatorio, estado`
- **LISTAS**: `id_lista, nombre_lista, tipo, creado_por`
- **ITEMS_LISTA**: `id_item, id_lista, item, estado`
- **USUARIOS**: `telegram_user, nombre, rol, permitido`
- **LOGS**: `timestamp, telegram_user, pantalla, opcion_elegida, resultado`
- **SESSIONS**: `telegram_user, pantalla_actual, paso_actual, datos_parciales, timestamp_ultima_interaccion`

> Nota: el workflow ya escribe y actualiza principalmente **USUARIOS**, **SESSIONS**, **CITAS**, **TAREAS**, **HABITOS**, **LISTAS** mediante operaciones `lookup`, `append` y `upsert`. 

------

## Arquitectura del workflow (n8n)

A alto nivel, el flujo hace:

1. **Telegram Trigger** recibe el mensaje.
2. **Check Session / Check User Profile** consultan Sheets para saber si el usuario existe y en qué “pantalla” va.
3. **Init Variables** normaliza variables (`msg`, `telegram_user`, `pantalla_actual`, `datos_previos_str`).
4. **Routers (Switch)** enrutan según pantalla y opción numérica (menú/steps).
5. **Set nodes** construyen `response_text`, `next_pantalla`, `datos_parciales`, `action_flag`.
6. **Format Fields → Update Session** persiste la sesión.
7. **If(action_flag) → Map → Save** guarda entidades (cita/tarea/hábito/lista/usuario).
8. **Telegram Reply** responde al usuario.

------

## Instalación y puesta en marcha

### Requisitos

- n8n Community Edition (recomendado: **v2.3.5**)
- Bot de Telegram (token)
- Google Sheets con el esquema anterior
- Credenciales OAuth2 de Google configuradas en n8n para Sheets



### Paso a paso

1. **Crear el bot en Telegram**

- En BotFather crea el bot y obtén el **token**.
- (Opcional) define comandos como `/start`.

1. **Preparar Google Sheets**

- Crea el documento (p.ej. `AgendaBot_DB`) con las pestañas y columnas indicadas arriba. 

1. **Configurar credenciales en n8n**

- **Telegram API**: añade el token del bot.
- **Google Sheets OAuth2**: conecta tu cuenta de Google con permisos al documento.

1. **Importar el workflow**

- n8n → *Workflows* → *Import from File* → selecciona el JSON del workflow: AgendaBot

1. **Ajustar el Sheet ID (si aplica)**

- En los nodos de Google Sheets, confirma/actualiza el `sheetId` para que apunte a tu documento.

1. **Activar el workflow**

- Activa el workflow en n8n.
- Escribe `/start` al bot en Telegram y navega con números.

------



## Uso rápido (Telegram)

- **/start** → bienvenida o registro (si eres nuevo).
- **Menú principal:** escribe el número (ej. `1`).
- **Agenda → 1**: inicia cita (fecha → hora → nombre → motivo → canal → confirmación).
- En pasos/menús, **9** vuelve/cancela cuando esté disponible.

------

## Validaciones y reglas de UX

- Opción válida según el menú mostrado.
- Fecha con formato permitido y **no en el pasado**.
- Confirmación antes de guardar.
- Mensajería humanizada: saluda, explica, muestra opciones y cómo salir.

------

## Licencia

Uso interno / proyecto de referencia (ajústala si vas a publicarlo: MIT/Apache-2.0/etc.).