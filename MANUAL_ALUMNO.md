# Manual del alumno · Laboratorio A3 + A4 · Producto real con Gemini (v2)

**LSCA2318 · Sirve para hacerlo en clase o en casa · Entrega del laboratorio: martes 24 de noviembre, 23:59 · Entrega de A3 y A4: jueves 26 de noviembre, 23:59**

Material (kit y este manual): https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4

## 0 · Primero entiende qué es cada cosa

Hay **tres** cosas distintas y se hacen **en este orden**:

| Orden | Qué es | Sobre qué trabajas | ¿Se califica? |
|---|---|---|---|
| 1 | **Laboratorio** (partes 1 a 3 de este manual) | El **caso guía**: el Manual de Ingeniería de FinNorte y un agente que crea tickets de despliegue | Solo la autenticación, que es tu A3 empezada en clase |
| 2 | **Actividad 3** (parte 4) | **Tu variante**: tu servicio con autenticación y tu endpoint | Sí, 6 % |
| 3 | **Actividad 4** (parte 5) | **Tu variante**: tu herramienta, tu agente, tu validador y tu interfaz | Sí, 6 % |

**No necesitas haber terminado nada antes.** El laboratorio funciona aunque tu equipo no tenga lista la A1, el Avance 1 ni la A4: el kit trae el caso guía **ya resuelto**, en una carpeta aparte (`src/finnorte/caso_guia/`), sin tocar tus archivos.

**Lo que se te entrega resuelto y lo que haces tú:**

| Pieza | En el laboratorio | En la A3 o la A4 |
|---|---|---|
| `api/auth.py` (registro, login y token) | **Lo completas tú** (5 funciones) | A3 la califica |
| `api/producto.py` (el servicio) | Se entrega | A3: le agregas **tu** endpoint `POST /v1/<tu-variante>` |
| `caso_guia/` (buscador, asistente, herramientas, agente de referencia) | Se entrega, solo se usa | No se califica |
| Ejecutor del agente | Usas el **de referencia** | A4: implementas **el tuyo** en `herramientas.py` |
| Validador | Usas `validar_plan` del caso guía | A4: escribes **el de tu variante** |
| Interfaz `app/cliente_api.py` | Se entrega para el caso guía | A4: la **adaptas** a tu variante |

**Nombres que vas a ver en los mensajes:**

| Mensaje | Qué significa | Archivo |
|---|---|---|
| `Pendiente: Laboratorio · api/auth.py · <función>` | Te falta esa función del laboratorio | `api/auth.py` |
| `Pendiente: A1` | Buscador semántico (Actividad 1) | `src/finnorte/retrieval.py` |
| `Pendiente: A2` | Pipeline de tu variante (Actividad 2) | `src/finnorte/pipeline.py` |
| `Pendiente: AV1` o `Avance 1` | Motor de tu variante (Avance 1) | `asistente.py`, `variante.py`, `controles.py` |
| `Pendiente: A4` | Tu ejecutor del agente (Actividad 4) | `src/finnorte/herramientas.py` |

El laboratorio **solo** necesita la primera fila.

## 0.1 · Conceptos clave (léelos antes de empezar)

Vas a construir lo mismo que hay detrás de cualquier app con «Iniciar sesión». Estas son las piezas, con una comparación para cada una.

### La app y la red

| Concepto | Qué es | Comparación |
|---|---|---|
| **Servicio o API** | Un programa que espera peticiones por la red y responde con datos (JSON). Aquí es `api/producto.py`, hecho con FastAPI | La ventanilla de un banco: no ves lo de adentro, solo pides y te responden |
| **Endpoint** | Una «ventanilla» concreta del servicio: un método más una ruta, como `POST /auth/login` o `POST /v1/preguntas` | Cada ventanilla atiende un solo trámite |
| **Petición y respuesta** | Lo que mandas (método, ruta, cabeceras y cuerpo JSON) y lo que te regresan (código de estado y JSON) | El formato que llenas y el comprobante que te dan |
| **Puerto** | El «número de puerta» por el que escucha un programa: el servicio usa el 8000 y la interfaz el 8501 | Dos negocios en el mismo edificio, cada uno con su número |
| **Interfaz (Streamlit)** | La página web que usa una persona. **No** hace el trabajo: le pide todo al servicio | El cajero automático: botones para ti, pero el dinero lo mueve el banco |
| **`/docs`** | Página que FastAPI genera sola, donde puedes probar cada endpoint sin escribir código | El catálogo de trámites con un botón de «probar» |

### Identidad: quién eres y si puedes pasar

| Concepto | Qué es | En este laboratorio |
|---|---|---|
| **Autenticación** | Comprobar **quién eres** | El login con `email` y `contrasena` |
| **Autorización** | Decidir **qué puedes hacer** una vez que se sabe quién eres | Solo con token puedes preguntar o usar el agente, y solo tú confirmas tus acciones pendientes |
| **Hash de contraseña** | Una «huella» de la contraseña que no se puede revertir. El servicio guarda la huella, **nunca** la contraseña | `hash_contrasena` usa PBKDF2-HMAC-SHA256 con miles de repeticiones |
| **Sal** | Un valor aleatorio distinto para cada usuario que se mezcla con la contraseña antes de sacar la huella. Así dos personas con la misma contraseña tienen huellas distintas y no sirven las tablas precalculadas | `os.urandom(16)`; se guarda junto al hash, como `pbkdf2$<sal>$<hash>` |
| **Comparación en tiempo constante** | Comparar dos valores tardando siempre lo mismo, para que un atacante no adivine letra por letra midiendo el tiempo | `hmac.compare_digest` en lugar de `==` |
| **Token** | Un «pase» que el servicio te da al iniciar sesión. Lo mandas en cada petición para no enviar tu contraseña cada vez | La pulsera de un evento: te la ponen en la entrada y con ella entras a todas las zonas |
| **Bearer** | La forma estándar de mandar el token: cabecera `Authorization: Bearer <token>`. «Bearer» significa «portador»: quien lo trae, pasa | En `/docs`, el candado **Authorize** agrega la palabra «Bearer» por ti |
| **Firma del token** | Una marca que el servicio calcula con un **secreto** que solo él conoce. Si alguien cambia una letra del token, la firma ya no coincide y responde 401 | El sello de agua de un billete |
| **Secreto (`TOKEN_SECRET`)** | Texto aleatorio largo (64 caracteres hexadecimales) con el que el servicio firma y verifica tokens. Si alguien lo conoce, puede fabricar tokens falsos | La llave del sello: por eso vive solo en `.env` |
| **Expiración** | El token deja de servir después de un tiempo (aquí, 15 minutos). Limita el daño si alguien lo roba | La pulsera de un solo día |
| **Sesión** | El periodo en que «sigues dentro» sin volver a escribir tu contraseña. En este proyecto la sesión **es** el token: el servicio no guarda una lista de conectados (no tiene estado); cada petición trae su token y se valida. Cuando expira, vuelves a iniciar sesión | En la interfaz, el token se guarda en `st.session_state`: la memoria de **esa** pestaña del navegador |
| **Bloqueo por intentos** | Después de 5 contraseñas malas seguidas, el servicio deja de aceptar intentos un rato (429). Frena a quien prueba contraseñas al azar | El cajero que retiene la tarjeta |
| **Mismo mensaje de error** | «Correo o contraseña incorrectos», exista o no el usuario. Si los mensajes fueran distintos, un atacante sabría qué correos están registrados | — |

**Llave de API frente a token:** la `GEMINI_API_KEY` identifica **a ti** ante Google y te cobra la cuota; el token identifica **a un usuario** ante **tu** servicio. Ninguno de los dos se sube al repositorio.

### Configuración y secretos

| Concepto | Qué es |
|---|---|
| **Variable de entorno** | Un valor que el sistema le pasa al programa al arrancar (como `LLM_PROVIDER=gemini`). Permite cambiar la configuración sin tocar el código |
| **`.env`** | Un archivo de texto con esas variables. Está en `.gitignore` para que **nunca** llegue a GitHub. `chmod 600` hace que solo tu usuario pueda leerlo |
| **Proveedor `fake`** | Un «modelo» simulado que responde sin internet. Sirve para probar el flujo cuando no hay cuota o no hay red; sus respuestas no son representativas |

### Errores: los códigos que vas a ver

| Código | Significa | Ejemplo en el laboratorio |
|---|---|---|
| **200 / 201** | Todo bien / se creó algo | Login correcto / registro nuevo |
| **401** | No sé quién eres o tu pase no sirve | Sin token, token alterado o expirado, contraseña incorrecta |
| **404** | No existe (o no es tuyo) | Confirmar una acción de otro usuario o que ya se usó |
| **409** | Conflicto con lo que ya existe | Registrar un correo repetido |
| **422** | Lo que mandaste no tiene el formato correcto | Pregunta de 1 carácter, correo sin `@` |
| **429** | Demasiadas solicitudes | Bloqueo tras 5 intentos, o más de 30 peticiones por minuto |
| **501** | Esa parte todavía no está implementada | Una función `TODO(LAB)` de `api/auth.py` sin completar |
| **503** | El modelo no está disponible | Se acabó la cuota de Gemini o no hay red |

**`problem+json`** es un formato estándar para errores: siempre trae `type`, `title`, `status`, `detail` y `trace_id`. Así la interfaz y quien depura saben qué pasó sin adivinar.

### El agente

| Concepto | Qué es |
|---|---|
| **Agente** | El modelo resuelve una tarea en pasos: en cada paso **propone** en JSON usar una herramienta o responder |
| **Herramienta** | Una función de Python que el agente puede pedir usar, con un esquema JSON que define sus argumentos válidos |
| **Herramienta sensible** | La que tiene efecto en el mundo (crear un ticket, borrar, pagar). **Nunca** se ejecuta sin que una persona vea los datos exactos y la autorice |
| **Confirmación en dos pasos** | 1) el código guarda la acción **pendiente** y te da un `id`; 2) tú la apruebas o rechazas con ese `id`, y entonces el **código** (no el modelo) la ejecuta |
| **Validador** | Una regla escrita en código normal que revisa el plan antes de ofrecerlo (aquí: no hay ticket si la ventana de despliegue no lo permite) |
| **Caso guía** | El ejemplo resuelto que usa todo el grupo (el Manual de Ingeniería de FinNorte). Tu variante es el problema de tu equipo |

### Así viaja una petición

```
1. Registro   POST /auth/registro {email, contrasena}  → el servicio guarda hash + sal          → 201
2. Login      POST /auth/login    {email, contrasena}  → verifica, firma un token con el secreto → 200 {token}
3. Uso        POST /v1/preguntas  + Authorization: Bearer <token>
              → el servicio revisa la firma y la expiración → si todo bien, responde       → 200
              → si el token es falso o venció                                              → 401
```

---

## 1 · Preparación (30 min)

### 1.1 · Llave de Gemini

1. Entra a https://aistudio.google.com/apikey con tu cuenta de Google. Si tu cuenta institucional no te deja, usa una cuenta personal de Gmail.
2. Acepta los términos y da clic en **Create API key**. Copia la llave: empieza con `AIza`.
3. La llave es personal. **Nunca** la pegues en un chat de IA ni la subas al repositorio. Si se filtra, revócala en esa misma página y crea otra.

### 1.2 · Elige dónde trabajar

| | Ruta A · AWS Learner Lab | Ruta B · Tu computadora |
|---|---|---|
| Cuándo | En clase (recomendada) o en casa | En casa |
| Necesitas | AWS Academy y cuenta de GitHub | Python 3.10 o más, VS Code y Git |

### 1.3 · Ruta A: lanza tu instancia (10 min)

1. AWS Academy › tu curso › **Modules › Learner Lab › Start Lab**. Espera el punto verde y da clic en **AWS**.
2. La región, arriba a la derecha, debe ser **N. Virginia (us-east-1)**.
3. **EC2 › Launch instance:**

| Campo | Valor |
|---|---|
| Name | `finnorte-<tu-usuario>` |
| AMI | **Ubuntu Server 24.04 LTS** (no Amazon Linux: trae Python 3.9) |
| Instance type | **t3.small** |
| Key pair | `vockey` |
| Storage | **16 GiB** |

4. **Network settings › Edit**, crea un security group con estas reglas:

| Tipo | Puerto | Origen |
|---|---|---|
| SSH | 22 | Anywhere |
| Custom TCP | 8000 | **My IP** |
| Custom TCP | 8501 | **My IP** |

   - Si más tarde la página deja de abrir (cambiaste de red o de salón), entra a **EC2 › Security groups › tu grupo › Edit inbound rules** y vuelve a elegir **My IP** en las reglas 8000 y 8501.
   - **Nunca** los abras a «Anywhere»: cualquiera podría registrarse en tu app y gastar tu cuota de Gemini.

5. **Launch instance** y anota la **Public IPv4 address**. En este manual la llamamos `<IP>`.
6. Selecciona la instancia › **Connect › EC2 Instance Connect › Connect**. Para pegar en la terminal usa **Ctrl+Shift+V**.

```
sudo apt update && sudo apt install -y python3-venv python3-pip unzip git gh
gh auth login
```

En `gh auth login` responde **GitHub.com › HTTPS › Yes › Login with a web browser**, abre https://github.com/login/device en **tu PC**, escribe el código y autoriza. No pegues tokens ni contraseñas en la terminal.

```
gh repo clone <organización-o-usuario>/<repositorio-del-equipo>
cd <repositorio-del-equipo>
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
```

### 1.4 · Copia el kit (rutas A y B)

**Dentro de la carpeta del repositorio de tu equipo** (donde están `pyproject.toml` y `src/`):

```
wget -O kit.zip https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4/raw/main/kit_lab_A3_A4_alumno.zip
unzip -n kit.zip
echo "bitacora/usuarios.db" >> .gitignore
```

- `unzip -n` **nunca sobrescribe**: el kit solo agrega archivos nuevos. Deja `kit.zip` por ahora; lo borras al terminar la sección 1.
- **En Windows (ruta B):** en la página del repositorio abre `kit_lab_A3_A4_alumno.zip`, da clic en **Download raw file**, descomprímelo y copia su **contenido** a la raíz del repositorio de tu equipo. Si Windows pregunta por un archivo repetido, elige **Omitir**.
- **Si ya habías copiado el kit anterior (v1)**, lee la parte 7 antes de seguir.

### 1.5 · Instala y configura

| | Ruta A (Ubuntu) | Ruta B (Windows PowerShell) |
|---|---|---|
| Entorno | `python3 -m venv .venv && source .venv/bin/activate` | `python -m venv .venv` y `.venv\Scripts\activate` |
| Instalar | `pip install -r requirements.txt && pip install -e .` | igual |
| `.env` | `cp -n .env.example .env && cat .env.lab.example >> .env` | `copy .env.example .env` y pega al final el contenido de `.env.lab.example` |
| Secreto | `python3 -c "import secrets; print(secrets.token_hex(32))"` | `python -c "import secrets; print(secrets.token_hex(32))"` |

Abre `.env` (`nano .env` en la ruta A; VS Code en la ruta B) y deja estas líneas:

```
LLM_PROVIDER=gemini
GEMINI_API_KEY=<tu llave>
EMB_PROVIDER=gemini
LLM_MODEL=gemini-flash-lite-latest
TOKEN_SECRET=<el secreto que generaste>
API_URL=http://127.0.0.1:8000
```

En `nano` guarda con **Ctrl+O, Enter** y sal con **Ctrl+X**. En la ruta A protege el archivo con `chmod 600 .env`.

**Checkpoint 0:** corre `python -m finnorte info`. Debe mostrar `gemini`. Después borra el zip con `rm kit.zip`.

| Si sale… | Causa | Solución |
|---|---|---|
| `No module named finnorte` | No estás en la raíz del repositorio o faltó `pip install -e .` | `cd` a la carpeta que tiene `pyproject.toml` y repite la instalación |
| `fake` en lugar de `gemini` | El `.env` no tiene `LLM_PROVIDER=gemini` o la llave está vacía | Revisa el `.env` |
| `requires-python >=3.10` | Instancia con Amazon Linux | Lanza Ubuntu 24.04 |

**Sin cuota o sin red:** con `LLM_PROVIDER=fake` y `EMB_PROVIDER=hash`, todo el laboratorio funciona con respuestas simuladas, incluido el agente.

---

## 2 · Laboratorio: el producto del caso guía (90 min)

Necesitas **dos terminales**. En la ruta A abre la segunda con **Connect** otra vez. En cada terminal nueva escribe primero:

```
cd <repositorio-del-equipo> && source .venv/bin/activate
```

| | Ruta A (AWS) | Ruta B (tu PC) |
|---|---|---|
| Terminal 1 · servicio | `uvicorn api.producto:app --host 0.0.0.0 --port 8000 --reload` | `uvicorn api.producto:app --reload` |
| Terminal 2 · interfaz | `streamlit run app/cliente_api.py --server.address 0.0.0.0 --server.port 8501 --server.headless true` | `streamlit run app/cliente_api.py` |
| Documentación (`/docs`) | `http://<IP>:8000/docs` | `http://127.0.0.1:8000/docs` |
| Interfaz | `http://<IP>:8501` | `http://localhost:8501` |

Escribe `http://`, no `https://`. Para correr las pruebas usa una **tercera** terminal, o detén el servicio con Ctrl+C.

### Paso 1 · El servicio responde (10 min)

1. Arranca el servicio en la terminal 1 y abre `/docs`.
2. `GET /salud` › **Try it out › Execute**: muestra el modelo configurado.
3. `POST /v1/preguntas` sin iniciar sesión: responde **401** en formato `problem+json`.

**Checkpoint 1:**

```
pytest tests/test_lab_producto.py -k "salud or sin_token or invalida"
```

Debe decir **3 passed**. Toma una captura del 401 con sus campos `type`, `title`, `status`, `detail` y `trace_id`.

### Paso 2 · La autenticación, escrita con IA y revisada por ti (30 min)

Antes de empezar repasa en la sección 0.1 qué son el hash, la sal, el token, el secreto y la expiración.

`api/auth.py` trae 5 funciones marcadas `TODO(LAB)`, cada una con una pista: `hash_contrasena`, `verificar_contrasena`, `emitir_token`, `validar_token` y `login`. **Esta parte es tu A3 empezada en clase**, así que guarda todo para tu PDF.

1. Pega este **prompt de diseño** en un asistente de IA:

> Actúa como ingeniero de seguridad de una fintech. Diseña en Python, sin librerías externas, las funciones `hash_contrasena`, `verificar_contrasena`, `emitir_token` (expira en 15 minutos) y `validar_token`, y un `login` con bloqueo tras 5 intentos fallidos. Requisitos: PBKDF2-HMAC-SHA256 con sal; comparación en tiempo constante; el mismo mensaje para usuario inexistente y contraseña incorrecta; el secreto del token se lee de una variable de entorno; la bitácora de intentos nunca guarda la contraseña. Explica cada decisión y señala qué riesgos quedan.

2. **Antes de copiar nada**, encuentra al menos 3 fallas en la propuesta y relaciona cada una con su regla del Manual (SEC, ing-02, ing-07). Las fallas típicas son:
   - MD5 o SHA-256 sin sal;
   - comparar con `==` en lugar de `hmac.compare_digest`;
   - mensajes distintos para «usuario no existe» y «contraseña incorrecta»;
   - el secreto del token escrito en el código;
   - tokens sin expiración.
3. Completa las 5 funciones respetando las firmas que ya trae el archivo. En la ruta A edita con `nano api/auth.py`, o edita en tu PC, haz push y en la instancia corre `git pull`.
4. Anota en `ai_log.md` qué **aceptaste**, qué **modificaste** y qué **rechazaste**.

**Checkpoint 2:**

```
pytest tests/test_lab_producto.py -k "mensaje or alterado or bloqueo or texto_plano"
```

Debe decir **4 passed**. Si dice «4 skipped», alguna función sigue pendiente: corre la prueba con `-rs` y el motivo dice cuál.

### Paso 3 · El token en acción (15 min)

Reinicia el servicio (Ctrl+C y arráncalo otra vez) y en `/docs`:

1. `POST /auth/registro` con `{"email": "tu-correo@ejemplo.com", "contrasena": "una-de-8-o-mas"}`. Responde **201**.
2. `POST /auth/login` con los mismos campos. Copia el valor de `token`, sin comillas.
3. Da clic en el candado **Authorize** (arriba a la derecha) y pega **solo el token**, sin escribir «Bearer».
4. `POST /v1/preguntas` con `{"pregunta": "¿Cada cuánto se rotan los secretos?"}`. Ahora responde el modelo, con `fuentes`.

| Prueba | Resultado esperado |
|---|---|
| Login con contraseña mala 6 veces | La sexta da **429** (bloqueo) |
| Token alterado (cambia la última letra en Authorize) | **401** |
| Pregunta de 1 carácter | **422** en `problem+json` |
| Usuario inexistente y contraseña mala | El mismo 401 con el mismo mensaje |

**Checkpoint 3:**

```
pytest tests/test_lab_producto.py
```

Debe decir **12 passed**. Toma una captura de cada fila de la tabla: van en la sección de pruebas de tu A3.

### Paso 4 · El agente detrás de la API (15 min)

En una app web el agente no puede «esperar» a que alguien dé clic. Por eso la confirmación va en **dos pasos**:

| Paso | Endpoint | Qué pasa |
|---|---|---|
| 1 | `POST /v1/agente {"tarea": ...}` | El agente usa las herramientas de consulta. Si propone una **sensible** (`crear_ticket_cambio`), no la ejecuta: el código la guarda como **pendiente** con los datos exactos y devuelve un `id` |
| 2 | `POST /v1/agente/confirmar/{id} {"aprobar": true}` | El **código** ejecuta exactamente lo guardado; el modelo ya no interviene |
| 2 | `POST /v1/agente/confirmar/{id} {"aprobar": false}` | No se ejecuta nada; la bitácora registra `rechazada` |

Antes de ofrecer la acción corre el **validador** `validar_plan`: un ticket solo se ofrece si `revisar_ventana_despliegue` dijo `permitido: true` para la misma fecha y riesgo (regla ing-03).

En `/docs`, con tu token:
1. `POST /v1/agente` con `{"tarea": "Quiero desplegar un cambio de riesgo alto en pagos-api el 2026-11-24. Si se puede, crea el ticket."}`. Revisa `pasos`, `validador` y `pendientes`.
2. Copia el `id` del pendiente y llama a `POST /v1/agente/confirmar/{id}` con `{"aprobar": true}`. Responde `"estado": "ejecutada"`.
3. Mira la bitácora con `tail -3 bitacora/trazas.jsonl`.

Aquí usas el **ejecutor de referencia** (`src/finnorte/caso_guia/agente_ref.py`). Léelo para entender el flujo. En la A4 escribes **el tuyo**.

### Paso 5 · La interfaz consume el servicio (20 min)

1. Arranca la interfaz en la terminal 2; el servicio sigue en la terminal 1.
2. Regístrate y entra desde la barra lateral.
3. **Preguntar:** observa el estado de espera, las fuentes y la latencia.
4. **Agente:**

| Tarea | Lo que debe pasar |
|---|---|
| «Quiero desplegar un cambio de riesgo alto en pagos-api el **2026-11-24**. Si se puede, crea el ticket.» | Aparece la acción pendiente con sus datos exactos. **Autoriza:** se crea el ticket |
| La misma tarea | Esta vez **rechaza**: no se crea nada y la bitácora registra `rechazada` |
| La misma tarea con fecha **2026-11-27** | Es viernes: la herramienta dice que no y no aparece nada que autorizar |

5. Detén el servicio (Ctrl+C en la terminal 1) y pulsa **Preguntar**. La interfaz debe mostrar un error amable, no tronar.

**Checkpoint 5:** captura de la acción pendiente, del rechazo y del error con el servicio apagado.

**Con esto terminaste el laboratorio.** Guarda tu trabajo:

```
git status            # que NO aparezcan .env ni bitacora/usuarios.db
git add . && git commit -m "lab A3+A4: auth y caso guía" && git push
```

---

## 3 · Lo que hiciste en el laboratorio y lo que sigue

### Entrega del laboratorio · martes 24 de noviembre, 23:59 (individual)

1. **Etiqueta de Git** en el repositorio de tu equipo, con tu `api/auth.py` completo:
   ```
   pytest tests/test_lab_producto.py      # debe decir 12 passed
   git add . && git commit -m "lab: auth completo"
   git tag lab-<usuario> && git push && git push --tags
   ```
2. **Un PDF corto** con estas capturas:

| Checkpoint | Captura |
|---|---|
| 1 | El 401 en `problem+json` con `type`, `title`, `status`, `detail` y `trace_id` |
| 2 | `pytest … -k "mensaje or alterado or bloqueo or texto_plano"` con **4 passed** |
| 3 | `pytest tests/test_lab_producto.py` con **12 passed** y las 4 filas de la tabla del paso 3 (429, token alterado, 422, mismo mensaje) |
| Paso 4 | `/v1/agente` con el pendiente y la confirmación con `"estado": "ejecutada"` |
| 5 | En la interfaz: la acción pendiente, el rechazo y el error con el servicio apagado |

Las mismas capturas las reutilizas en la A3 y la A4 (jueves 26), así que no las repites.

| Hiciste en el laboratorio | Lo usas en |
|---|---|
| Las 5 funciones de `api/auth.py` | A3 · Componente de autenticación |
| Capturas de 401, 429, 422 y token alterado | A3 · Pruebas |
| El prompt de diseño y las fallas que encontraste | A3 · Evaluación crítica de la propuesta |
| Entendiste el agente en dos pasos y el validador | A4 · tu ejecutor y tu validador |
| Viste la interfaz del caso guía | A4 · adaptarla a tu variante |

---

## 4 · Actividad 3 · Servicio de tu variante con autenticación

**Qué entregas:** la etiqueta `a3-<usuario>` en el repositorio y un PDF de 5 a 7 páginas. Instrucciones y rúbrica completas: el Word de la A3.

1. **Componente de autenticación:** tu `api/auth.py` del laboratorio, con los 6 requisitos mínimos (hash con sal, expiración de 15 minutos o menos, mismo mensaje, comparación en tiempo constante, bloqueo y bitácora sin contraseña).
2. **Tu endpoint** `POST /v1/<tu-variante>` en `api/producto.py`:
   - con **su propio modelo de entrada** (por ejemplo, `{"requisito": "..."}` en v1-qa o `{"logs": "..."}` en v5-triage) y validación;
   - protegido con `Depends(usuario_actual)`, con errores `problem+json` y el límite de solicitudes;
   - que llame al `variante.sistema()` de tu equipo (Avance 1). El `/v1/variante` del kit con `"motor": "demo"` **no cuenta** como tu endpoint;
   - probado con 2 entradas reales de tu variante.
3. **Análisis del `login()` oficial:** tabla con problema, explicación técnica, impacto real y tipo de mejora; di si la respuesta del asistente fue suficiente, ambigua o incompleta.
4. **Las 3 versiones:** `a3/login_seguridad.py`, `a3/login_eficiencia.py` y `a3/login_mantenible.py`, con qué cambió, por qué y la matriz problema → versión → evidencia.
5. **Pruebas** en `tests/test_a3_*.py`: las 3 oficiales (login exitoso, contraseña incorrecta, usuario inexistente) y al menos 3 propias (token expirado, token alterado, bloqueo o entrada 422). Critica las pruebas que te dé el asistente.
6. **Comparación y reflexión:** tabla de 5 criterios para las 4 versiones y recomendaciones de la IA aceptadas, modificadas y descartadas.
7. `ai_log.md` al día, `git tag a3-<usuario> && git push --tags`.

---

## 5 · Actividad 4 · Interfaz y herramienta de tu variante, con confirmación humana

**Qué entregas:** la etiqueta `a4-<usuario>`, un PDF de 5 a 7 páginas y el enlace a la demo de 2 minutos. Instrucciones y rúbrica completas: el Word de la A4.

1. **Tu herramienta** con esquema JSON estricto, tipo, control y límite:

| Variante | Herramienta mínima | Control |
|---|---|---|
| v1-qa | `ejecutar_pytest(archivo)` | Solo el archivo generado; tiempo máximo de 30 s |
| v2-revisor | `correr_linter(ruta)` con ruff o bandit | Sin ejecutar el código revisado |
| v3-soporte | `consultar_openapi(ruta, metodo)` | Solo lectura de `openapi.json` |
| v4-documentador | `validar_openapi(especificacion)` | Validador local, sin red |
| v5-triage | `buscar_logs(desde, hasta, patron)` | Rango máximo de 2 h; PII enmascarada |

2. **Tu ejecutor:** implementa `ejecutar_agente` en `src/finnorte/herramientas.py`. Guíate por su docstring y por `tests/test_a4_agente.py`. Debe:
   - validar los argumentos contra el esquema;
   - no ejecutar lo sensible si `confirmar` devuelve `False` **o `None`** (`None` = pendiente: la decisión llega en otra solicitud);
   - detectar repeticiones y tener tope de pasos.

   El de referencia del laboratorio te sirve para entender el flujo; copiarlo no cuenta como tu implementación.
3. **Conéctalo** en `api/producto.py`:
   - en `_herramientas()` agrega tu herramienta;
   - en `agente()` usa `agente_servicio.iniciar(..., ejecutor=herramientas.ejecutar_agente, validador=tu_validador)`.
4. **Tu validador** de lógica tradicional, con la regla de tu variante (tabla del Word de la A4). Debe correr en cada respuesta y verse en la interfaz.
5. **Adapta la interfaz** `app/cliente_api.py` a tu variante: el modo «Mi variante» llama a **tu** endpoint de la A3, el modo «Agente» usa tu herramienta, con estados de carga y error, fuentes y el resultado del validador.
6. **Primera ejecución con 5 entradas**, matriz de trazabilidad, errores clasificados, 2 mejoras y las mismas 5 entradas otra vez.
7. **Prueba de seguridad:** una falla de herramienta (por ejemplo, un archivo que no existe) y una **confirmación rechazada**.
8. `ai_log.md` al día, `git tag a4-<usuario> && git push --tags`.

---

## 6 · Al terminar cada sesión (ruta A)

1. `git status`, y después `git add . && git commit -m "..." && git push`.
2. EC2 › tu instancia › **Instance state › Stop instance**, y luego **End Lab**. **No uses Terminate** antes del 26 de noviembre.
3. **Para volver:**
   1. Start Lab › EC2 › **Start instance**.
   2. La `<IP>` **cambia**: copia la nueva.
   3. Si estás en otra red, actualiza las reglas 8000 y 8501 a **My IP**.
   4. Entra con Connect, `cd` a tu repositorio, `source .venv/bin/activate` y `git pull`.

## 7 · Si ya habías copiado el kit v1

El kit v1 **reemplazaba** `api/main.py`, `src/finnorte/herramientas.py` y `tests/test_s4_api.py`, y usaba `uvicorn api.main:app`.

1. Copia el kit v2 como en la sección 1.4. **No pierdes** tu `api/auth.py`: `cp -n` no lo toca.
2. Reemplaza estos 3 archivos, que sí cambiaron:

```
unzip -o kit.zip src/finnorte/agente_servicio.py app/cliente_api.py tests/test_lab_producto.py
```

En Windows copia esos 3 archivos del zip eligiendo **Reemplazar**.

3. Si el v1 borró tu `herramientas.py` o tu `main.py`, recupéralos del commit anterior: `git log --oneline -- src/finnorte/herramientas.py` y luego `git checkout <commit> -- src/finnorte/herramientas.py`.
4. Desde ahora el servicio se arranca con `uvicorn api.producto:app`.

## 8 · Errores comunes

| Síntoma | Causa | Solución |
|---|---|---|
| `501 Pieza pendiente` con `api/auth.py` | Falta esa función del laboratorio | El `detail` dice cuál |
| `RuntimeError: Define TOKEN_SECRET` | Falta el secreto en `.env` | Genera uno con `secrets.token_hex(32)` |
| `503 Proveedor del modelo no disponible` | Cuota agotada, llave mala o sin red | Revisa la llave, espera un minuto o usa `fake` |
| 401 aunque pusiste el token | Escribiste «Bearer» en Authorize, o pasaron 15 minutos | Pega solo el token; vuelve a iniciar sesión |
| La interfaz dice «sin conexión» | El servicio no está corriendo o `API_URL` apunta a otro puerto | Arráncalo en la terminal 1; revisa `API_URL` en `.env` |
| El navegador no abre `<IP>:8000` o `<IP>:8501` | Usaste `https`, cambió tu IP o cambió la de la instancia | `http://`; actualiza **My IP** en el security group |
| `ensurepip is not available` | Falta `python3-venv` | `sudo apt install -y python3-venv` |
| `pip` se queda sin memoria | Instancia micro | Usa t3.small |
| Streamlit pide un correo | Falta `--server.headless true` | Agrégalo al comando |
| `No module named finnorte` | No estás en la raíz del repositorio o faltó `pip install -e .` | Ver Checkpoint 0 |

## Qué no se negocia

- La llave de Gemini y el `TOKEN_SECRET` viven solo en `.env`. Una llave en el repositorio cuesta **−15** y se revoca.
- Los puertos 8000 y 8501 se abren solo a **My IP**.
- Nada sensible se ejecuta sin la autorización de una persona que vio los datos exactos.
- Todo uso de IA va en `ai_log.md`.
