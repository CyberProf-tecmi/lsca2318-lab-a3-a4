# LSCA2318 · Laboratorio A3 + A4 · Producto real con Gemini

Material para el **laboratorio** y las **Actividades 3 y 4**. Entrega de A3 y A4: **lunes 23 de noviembre, 23:59**.

| Archivo | Qué es |
|---|---|
| [`MANUAL_ALUMNO.md`](MANUAL_ALUMNO.md) | **Empieza aquí.** Primero el laboratorio paso a paso y después la A3 y la A4, explicadas desde cero |
| `kit_lab_A3_A4_alumno.zip` | Kit **v2**: se copia encima del repositorio de tu equipo y **no reemplaza** ningún archivo tuyo |
| `Presentacion_Laboratorio_A3_A4.pptx` | Presentación del laboratorio: conceptos (token, secreto, sesión, códigos HTTP), pasos y checkpoints |
| `Presentacion_Actividades_A3_A4.pptx` | Presentación de la A3 y la A4: qué construyes, pasos, rúbricas y lista de revisión |

## En una línea

1. **Laboratorio:** con el caso guía, que ya viene resuelto, levantas un servicio con login y token, un agente que pide permiso y una interfaz. Solo escribes las 5 funciones de `api/auth.py`.
2. **A3:** autenticación más el endpoint de **tu** variante.
3. **A4:** herramienta, agente, validador e interfaz de **tu** variante.

**No necesitas tener terminadas la A1, el Avance 1 ni la A4** para hacer el laboratorio.

## Copiar el kit (dentro del repositorio de tu equipo)

```
wget -O kit.zip https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4/raw/main/kit_lab_A3_A4_alumno.zip
unzip -n kit.zip
echo "bitacora/usuarios.db" >> .gitignore
```

`unzip -n` nunca sobrescribe. En Windows: abre el zip en esta página, da clic en **Download raw file**, descomprímelo y copia su contenido eligiendo **Omitir** en los archivos repetidos.

**¿Copiaste el kit anterior (v1)?** Lee la sección 7 del manual.

## Novedades de la v2 (18 de noviembre)

- El laboratorio ya no depende de A1, Avance 1 ni A4: el caso guía viene resuelto en `src/finnorte/caso_guia/`.
- El servicio ahora es `api/producto.py` (`uvicorn api.producto:app`) y **no reemplaza** tu `api/main.py`.
- Cada checkpoint tiene un resultado exacto: 3 passed, 4 passed y 12 passed.
- Sin red (`LLM_PROVIDER=fake`) también funciona el agente.
- La interfaz lee `API_URL` del `.env`.
- El manual explica los conceptos clave: servicio, endpoint, token, Bearer, secreto, firma, sesión, hash, sal y códigos HTTP.

## Reglas

- Tu llave de Gemini y el `TOKEN_SECRET` viven **solo** en `.env`. Una llave en el repositorio cuesta **−15** y se revoca.
- En AWS, los puertos 8000 y 8501 se abren **solo a My IP**.
- Todo uso de IA va en `ai_log.md`.
