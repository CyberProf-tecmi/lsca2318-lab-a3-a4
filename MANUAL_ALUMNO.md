# Manual del alumno · Laboratorio A3 + A4 · Producto real con Gemini

**LSCA2318 · 18 de noviembre de 2026 · Sirve para hacerlo en clase o en casa**

**Entrega de A3 y A4: lunes 23 de noviembre, 23:59.** Ambas son la base del Avance 2.

**Material del laboratorio (kit y este manual):** https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4

## Qué vas a construir

Una aplicación de verdad, con piezas que se hablan por la red:

```
Navegador ─► Streamlit :8501 ─(token)─► FastAPI :8000 ─► Gemini
                                         ├─ /v1/preguntas   caso guía
                                         ├─ /v1/variante    tu motor    ← A3
                                         └─ /v1/agente ─► /confirmar   ← A4
```

| Pieza | Archivo | Actividad |
|---|---|---|
| Interfaz | `app/cliente_api.py` | A4 |
| Servicio con token | `api/main.py` | A3 |
| Autenticación | `api/auth.py` (5 funciones por completar) | A3 |
| Agente con confirmación | `src/finnorte/agente_servicio.py` | A4 |
| Validador de reglas | `validar_plan` | A4 |

**La idea central:** el modelo **propone**; el código **decide**; la persona **autoriza** lo que tiene efecto.

## Elige tu ruta

| | Ruta A · AWS Learner Lab | Ruta B · Tu computadora |
|---|---|---|
| Cuándo | **En clase** (recomendada) o en casa | En casa, si ya tienes el andamio funcionando |
| Ventaja | Todos con el mismo Linux; sin problemas de Windows | No dependes de la sesión del lab |
| Necesitas | Acceso a AWS Academy y cuenta de GitHub | Python 3.10 o más, VS Code y Git |

**Para las dos rutas:**
1. **Llave de Gemini:** entra a https://aistudio.google.com/apikey con tu cuenta de Google, acepta los términos y da clic en **Create API key**. Copia la llave (empieza con `AIza`).
   - Si tu cuenta institucional no te deja entrar a AI Studio, usa una cuenta personal de Gmail.
   - La llave es personal. **Nunca** la pegues en un chat de IA ni la subas al repositorio.
   - Si se filtra, revócala en esa misma página y crea otra.
2. **Kit en el repositorio del equipo.** Basta con que **una persona del equipo** lo haga y los demás actualicen su copia. Descarga `kit_lab_A3_A4_alumno.zip` de https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4 y descomprímelo en la raíz del repositorio:
   - reemplaza `api/main.py`, `src/finnorte/herramientas.py` y `tests/test_s4_api.py`;
   - los demás archivos son nuevos.

   **Si tu `herramientas.py` ya tiene `ejecutar_agente` funcionando, no lo reemplaces.**
3. Agrega a `.gitignore` la línea `bitacora/usuarios.db`: ahí se guardan los hashes de las contraseñas de prueba.
4. Commit y push: `git add . && git commit -m "lab: kit A3+A4" && git push`.
   - Si nadie del equipo tiene el repositorio en su PC: en GitHub, **Add file › Upload files**, arrastra el **contenido** del kit (las carpetas `api`, `app`, `src`, `tests` y los archivos sueltos) y da clic en **Commit changes**.
   - Los archivos que empiezan con punto, como `.env.example`, pueden no subirse al arrastrar. No pasa nada: tu repositorio ya trae uno y en el paso A4 solo le agregas dos líneas.

## Ruta A · AWS Learner Lab

### A1 · Lanzar tu instancia (10 min)

1. AWS Academy › tu curso › **Modules › Learner Lab › Start Lab**. Espera el punto verde y da clic en **AWS**.
2. Arriba a la derecha, la región debe ser **N. Virginia (us-east-1)**.
3. **EC2 › Launch instance**:

| Campo | Valor |
|---|---|
| Name | `finnorte-<tu-usuario>` |
| AMI | **Ubuntu Server 24.04 LTS** (no Amazon Linux: trae Python 3.9 y el proyecto pide 3.10 o más) |
| Instance type | **t3.small** |
| Key pair | `vockey` |
| Storage | **16 GiB** |

4. **Network settings › Edit**, crea un security group con estas reglas:

| Tipo | Puerto | Origen |
|---|---|---|
| SSH | 22 | Anywhere |
| Custom TCP | 8000 | **My IP** |
| Custom TCP | 8501 | **My IP** |

   **Nunca abras 8000 ni 8501 a «Anywhere»:** tu servicio quedaría expuesto a todo internet.

5. **Launch instance.** Anota la **Public IPv4 address**. En este manual la llamamos `<IP>`.

### A2 · Entrar e instalar (5 min)

1. Selecciona tu instancia › **Connect › EC2 Instance Connect › Connect**. Se abre una terminal en el navegador.
2. Para pegar en ella usa **Ctrl+Shift+V** o el clic derecho.

```
sudo apt update && sudo apt install -y python3-venv python3-pip unzip gh
```

### A3 · Traer el repositorio de tu equipo (5 min)

```
gh auth login
```

Responde: **GitHub.com › HTTPS › Yes** (autenticar Git) **› Login with a web browser**.
- Te muestra un código de 8 caracteres.
- En **tu PC** abre https://github.com/login/device, escribe el código y autoriza.
- **No** generes ni pegues tokens ni contraseñas en la terminal.

```
gh repo clone <organización-o-usuario>/<repositorio-del-equipo>
cd <repositorio-del-equipo>
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
ls api        # debe aparecer auth.py
```

Si `api/auth.py` no aparece, tu equipo aún no sube el kit. Descárgalo directo en la instancia y súbelo:

```
wget -O kit.zip https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4/raw/main/kit_lab_A3_A4_alumno.zip
unzip -o kit.zip && rm kit.zip
echo "bitacora/usuarios.db" >> .gitignore
git add . && git commit -m "lab: kit A3+A4" && git push
```

Si ya tienes `ejecutar_agente` funcionando, usa `unzip -o kit.zip -x src/finnorte/herramientas.py`.

### A4 · Entorno y llave (5 min)

```
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt && pip install -e .
cp -n .env.example .env
python3 -c "import secrets; print(secrets.token_hex(32))"     # copia el resultado
nano .env
```

`pip install` tarda de 3 a 4 minutos. En `nano` deja así las líneas:

```
LLM_PROVIDER=gemini
LLM_MODEL=gemini-flash-lite-latest
GEMINI_API_KEY=<tu llave>
EMB_PROVIDER=gemini
TOKEN_SECRET=<el resultado que copiaste>
API_URL=http://127.0.0.1:8000
```

- Si alguna línea no existe en tu `.env`, agrégala al final.
- Guarda con **Ctrl+O, Enter** y sal con **Ctrl+X**. Después protege el archivo:

```
chmod 600 .env
python -m finnorte info        # debe decir gemini
```

**`API_URL` se queda en 127.0.0.1** porque la interfaz y el servicio corren en la misma instancia. Solo tu navegador usa `<IP>`.

**Checkpoint 0 (ruta A):** `python -m finnorte info` muestra `gemini`.

### A5 · Dos terminales

Abre una segunda terminal con **Connect** otra vez. En **cada** terminal nueva escribe primero:

```
cd <repositorio-del-equipo> && source .venv/bin/activate
```

| | Ruta A (AWS) |
|---|---|
| Terminal 1 · servicio | `uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload` |
| Terminal 2 · interfaz | `streamlit run app/cliente_api.py --server.address 0.0.0.0 --server.port 8501 --server.headless true` |
| Documentación del servicio | `http://<IP>:8000/docs` |
| Interfaz | `http://<IP>:8501` |

Escribe `http://`, **no** `https://`. Si cierras la pestaña de una terminal, se detiene lo que corría en ella.

Ahora sigue con los **pasos 1 a 6**.

## Ruta B · Tu computadora

En la carpeta del repositorio de tu equipo (con el kit ya copiado), en la terminal de VS Code:

| | Windows (PowerShell) | Mac o Linux |
|---|---|---|
| Entorno | `python -m venv .venv` y `.venv\Scripts\activate` | `python3 -m venv .venv` y `source .venv/bin/activate` |
| Instalar | `pip install -r requirements.txt` y `pip install -e .` | igual |
| `.env` | `copy .env.example .env` (si no existe) | `cp -n .env.example .env` |
| Secreto | `python -c "import secrets; print(secrets.token_hex(32))"` | igual, con `python3` |

- Llena `.env` igual que en la ruta A4 y guárdalo con Ctrl+S.
- **Checkpoint 0 (ruta B):** `python -m finnorte info` muestra `gemini`.

| | Ruta B (tu PC) |
|---|---|
| Terminal 1 · servicio | `uvicorn api.main:app --reload` |
| Terminal 2 · interfaz | `streamlit run app/cliente_api.py` |
| Documentación del servicio | `http://127.0.0.1:8000/docs` |
| Interfaz | `http://localhost:8501` |

**Si Windows se traba:**
- **«La ejecución de scripts está deshabilitada»:** corre `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, responde `S` y vuelve a activar el entorno.
- **`python` abre la Microsoft Store:** usa `py` en lugar de `python`, o reinstala Python desde python.org marcando «Add to PATH».
- **El firewall pregunta al arrancar `uvicorn` o `streamlit`:** da clic en **Permitir**.
- **Trabajas dentro de OneDrive:** mueve el proyecto a `C:\proyectos`.

---

**Sobre la cuota gratuita:** cada paso del agente es una solicitud a Gemini.
- Usa *flash-lite* y prueba con pocas entradas.
- Si ves 429 o 503 por cuota, sigue trabajando con `LLM_PROVIDER=fake` y `EMB_PROVIDER=hash` en `.env`, y reinicia el servicio.

## Paso 1 · El servicio responde (10 min)

1. Arranca el servicio en la **terminal 1** y abre la documentación del servicio (`/docs`) en tu navegador. FastAPI la genera sola.
2. `GET /salud` › **Try it out › Execute**: muestra el modelo configurado.
3. `POST /v1/preguntas` sin iniciar sesión: responde **401** en formato `problem+json`.

**Checkpoint 1:** captura del 401 con sus campos `type`, `title`, `status`, `detail` y `trace_id`.

## Paso 2 · La autenticación, con el asistente de IA bajo disciplina (25 min)

`api/auth.py` trae 5 funciones marcadas `TODO(LAB)`. Cada una tiene una pista. Las completas **con un asistente de IA**, pero revisando lo que propone.

**Prompt de diseño** (guárdalo para tu PDF de la A3):

> Actúa como ingeniero de seguridad de una fintech. Diseña en Python, sin librerías externas, las funciones `hash_contrasena`, `verificar_contrasena`, `emitir_token` (expira en 15 minutos) y `validar_token`, y un `login` con bloqueo tras 5 intentos fallidos. Requisitos: PBKDF2-HMAC-SHA256 con sal; comparación en tiempo constante; el mismo mensaje para usuario inexistente y contraseña incorrecta; el secreto del token se lee de una variable de entorno; la bitácora de intentos nunca guarda la contraseña. Explica cada decisión y señala qué riesgos quedan.

**Evalúa la propuesta antes de copiarla.** Encuentra al menos 3 fallas y relaciona cada una con su regla del Manual (SEC, ing-02, ing-07). Las fallas típicas son:
- MD5 o SHA-256 sin sal;
- comparar con `==` en lugar de `hmac.compare_digest`;
- mensajes distintos para «usuario no existe» y «contraseña incorrecta»;
- el secreto del token escrito en el código;
- tokens sin expiración.

**Para editar en la ruta A:** `nano api/auth.py`. Si prefieres editar en VS Code, hazlo en tu PC, haz push y en la instancia corre `git pull`.

Anota en `ai_log.md` qué **aceptaste**, qué **modificaste** y qué **rechazaste**.

**Checkpoint 2:** corre esto en una terminal libre. Las 4 pruebas deben quedar en verde.

```
pytest tests/test_lab_producto.py -k "mensaje or alterado or bloqueo or texto_plano"
```

## Paso 3 · El token en acción (10 min)

En `/docs`:
1. `POST /auth/registro` con tu correo y una contraseña de 8 o más caracteres.
2. `POST /auth/login` y copia el valor de `token`, sin comillas.
3. Da clic en el candado **Authorize** (arriba a la derecha) y pega **solo el token**, sin escribir «Bearer».
4. Repite `POST /v1/preguntas`: ahora responde Gemini, con fuentes.

| Prueba | Resultado esperado |
|---|---|
| Login con contraseña mala 6 veces | La sexta da **429** (bloqueo) |
| Token alterado (cambia la última letra en Authorize) | **401** |
| Pregunta de 1 carácter | **422** en `problem+json` |
| Usuario inexistente vs. contraseña mala | El mismo 401 con el mismo mensaje |

**Checkpoint 3:** captura de cada caso. Van en la sección de pruebas de tu A3.

## Paso 4 · El agente detrás de la API (20 min)

En una app web, el agente no puede «esperar» a que alguien dé clic. Por eso la confirmación va en dos pasos:

| Paso | Endpoint | Qué pasa |
|---|---|---|
| 1 | `POST /v1/agente {"tarea": ...}` | El agente usa las herramientas de consulta. Si propone una **sensible**, no la ejecuta: el código la guarda como **pendiente** con los datos exactos y devuelve un `id` |
| 2 | `POST /v1/agente/confirmar/{id} {"aprobar": true}` | El **código** ejecuta exactamente lo guardado; el modelo ya no interviene |
| 2 | `POST /v1/agente/confirmar/{id} {"aprobar": false}` | No se ejecuta nada; la bitácora registra `rechazada` |

- **Validador:** antes de ofrecer la acción corre `validar_plan`. En el caso guía, un ticket solo se ofrece si `revisar_ventana_despliegue` dijo `permitido: true` para la misma fecha y el mismo riesgo (regla ing-03).
- **Si tu `ejecutar_agente` aún no está implementado,** impleméntalo ahora; guíate por su docstring y por `tests/test_a4_agente.py`. Debe:
  - validar los argumentos contra el esquema;
  - no ejecutar lo sensible si `confirmar` devuelve `False` o `None`;
  - detectar repeticiones (misma herramienta con los mismos argumentos);
  - tener tope de pasos.

**Checkpoint 4:** todo debe quedar en verde o SKIP, sin errores.

```
pytest tests/test_a4_agente.py tests/test_lab_producto.py
```

Puedes ver la bitácora con `tail -3 bitacora/trazas.jsonl`.

## Paso 5 · La interfaz consume el servicio (15 min)

1. Arranca la interfaz en la **terminal 2**. El servicio sigue corriendo en la terminal 1.
2. Abre la interfaz en tu navegador. Regístrate y entra desde la barra lateral.
3. **Preguntar:** observa el estado de espera, las fuentes y la latencia.
4. **Agente:** haz estas tres pruebas.

| Tarea | Lo que debe pasar |
|---|---|
| «Quiero desplegar un cambio de riesgo alto en pagos-api el **2026-11-24**. Si se puede, crea el ticket.» | Aparece la acción pendiente con sus datos exactos. **Autoriza:** se crea el ticket |
| La misma tarea | Esta vez **rechaza**: no se crea nada y la bitácora registra `rechazada` |
| La misma tarea con fecha **2026-11-27** | Es viernes: la herramienta dice que no y no aparece nada para autorizar |

5. Detén el servicio (Ctrl+C en la terminal 1) y presiona **Preguntar**. La interfaz debe mostrar un error amable, no tronar.

**Checkpoint 5:** captura de la acción pendiente, del rechazo y del error con el servicio apagado. Son tres de las evidencias de tu A4.

## Paso 6 · Tu variante (el resto de la sesión y la tarea)

### Actividad 3 · Servicio con autenticación

1. **Endpoint `POST /v1/variante`** (puedes renombrarlo `/v1/<tu-variante>`), que llama a `variante.sistema()`.
   - Ya está protegido por el token, con `problem+json` y límite de solicitudes.
   - Pruébalo con 2 entradas reales de tu variante.
2. **Análisis del `login()` oficial:**
   - tabla con problema, explicación técnica, impacto real y tipo de mejora;
   - di si la respuesta del asistente fue suficiente, ambigua o incompleta.
3. **Las 3 versiones:** `a3/login_seguridad.py`, `a3/login_eficiencia.py` y `a3/login_mantenible.py`. Para cada una explica qué cambió y por qué, e incluye la matriz problema → versión → evidencia.
4. **Pruebas `tests/test_a3_*.py`:**
   - las 3 oficiales: login exitoso, contraseña incorrecta y usuario inexistente;
   - al menos 3 propias: token expirado, token alterado, bloqueo o entrada 422.
5. **Comparación y reflexión:**
   - tabla de 5 criterios para las 4 versiones;
   - recomendaciones de la IA que aceptaste, modificaste y descartaste.
6. Etiqueta `a3-<usuario>` y PDF de 5 a 7 páginas. Las instrucciones completas y la rúbrica están en el Word de la A3.

### Actividad 4 · Interfaz y herramienta con confirmación

1. **La herramienta de tu variante,** con esquema estricto, tipo, control y límite.
   - En `api/main.py`, cambia `_herramientas()` para que devuelva tus herramientas.
   - Puedes conservar también las del caso guía.

| Variante | Herramienta mínima | Control |
|---|---|---|
| v1-qa | `ejecutar_pytest(archivo)` | Solo el archivo generado; tiempo máximo de 30 s |
| v2-revisor | `correr_linter(ruta)` con ruff o bandit | Sin ejecutar el código revisado |
| v3-soporte | `consultar_openapi(ruta, metodo)` | Solo lectura de `openapi.json` |
| v4-documentador | `validar_openapi(especificacion)` | Validador local, sin red |
| v5-triage | `buscar_logs(desde, hasta, patron)` | Rango máximo de 2 h; PII enmascarada |

2. **Tu validador de lógica tradicional:** reemplaza `validar_plan` con la regla de tu variante (está en la tabla de la A4). Debe correr en cada respuesta y verse en la interfaz.
3. **La interfaz sobre TU servicio,** en los modos «Mi variante» y «Agente», con estados de carga y de error, y con fuentes.
4. **Primera ejecución con 5 entradas:**
   - matriz de trazabilidad;
   - errores clasificados;
   - 2 mejoras;
   - las mismas 5 entradas otra vez.
5. **Prueba de seguridad:** una falla de herramienta (por ejemplo, un archivo que no existe) y una **confirmación rechazada**.
6. Etiqueta `a4-<usuario>` y PDF de 5 a 7 páginas. Las instrucciones completas y la rúbrica están en el Word de la A4.

## Al terminar cada sesión

1. **Guarda tu trabajo en GitHub.** Antes del commit revisa que `git status` no muestre `.env` ni `bitacora/usuarios.db`.
   ```
   git status
   git add . && git commit -m "lab A3+A4: <qué hiciste>" && git push
   ```
2. **Ruta A:**
   - EC2 › tu instancia › **Instance state › Stop instance**, y luego **End Lab**.
   - **No la termines (Terminate)** hasta después del 23 de noviembre.
3. **Para volver (ruta A):**
   1. Start Lab › EC2 › **Start instance**.
   2. La `<IP>` **cambia**: copia la nueva.
   3. Si estás en otra red, actualiza las reglas 8000 y 8501 a **My IP**.
   4. Entra con Connect, haz `cd` a tu repositorio, `source .venv/bin/activate` y `git pull`.

## Errores comunes

| Síntoma | Causa | Solución |
|---|---|---|
| `501 Pieza pendiente` | Falta completar una función (`TODO(LAB)`, A1, Avance 1 o A4) | El `detail` dice cuál |
| `RuntimeError: Define TOKEN_SECRET` | Falta el secreto en `.env` | Genera uno con `secrets.token_hex(32)` |
| `503 Proveedor del modelo no disponible` | Cuota agotada, llave mala o sin red | Revisa la llave, espera un minuto o usa `fake` |
| 401 aunque pusiste el token | Escribiste «Bearer» en Authorize, o pasaron 15 minutos | Pega solo el token; vuelve a iniciar sesión |
| La interfaz dice «sin conexión» | El servicio no está corriendo | Arráncalo en la terminal 1 |
| El navegador no abre `<IP>:8000` o `<IP>:8501` (A) | Usaste `https`, falta la regla «My IP» o cambió la IP | `http://`, revisa el security group y la IP |
| `requires-python >=3.10` (A) | Lanzaste Amazon Linux | Lanza Ubuntu 24.04 |
| `ensurepip is not available` (A) | Falta `python3-venv` | `sudo apt install -y python3-venv` |
| `pip` se queda sin memoria (A) | Instancia micro | Usa t3.small |
| Streamlit pide un correo (A) | Falta `--server.headless true` | Agrégalo al comando |
| El agente da vueltas sin terminar | El modelo no devolvió el JSON esperado | Revisa la traza; el tope de pasos lo detiene |

## Qué no se negocia

- La llave de Gemini y el `TOKEN_SECRET` viven solo en `.env`. Una llave en el repositorio cuesta **−15** y se revoca.
- Los puertos 8000 y 8501 se abren solo a **My IP**.
- Nada sensible se ejecuta sin la autorización de una persona que vio los datos exactos.
- Todo uso de IA va en `ai_log.md`.
