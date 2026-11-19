# LSCA2318 · Laboratorio A3 + A4 · Producto real con Gemini

Kit del laboratorio del **18 de noviembre de 2026**. Es la base de la **Actividad 3** y la **Actividad 4**, que se entregan el **lunes 23 de noviembre a las 23:59**.

| Archivo | Qué es |
|---|---|
| `kit_lab_A3_A4_alumno.zip` | Archivos que se copian en la raíz del **repositorio de tu equipo** |
| [`MANUAL_ALUMNO.md`](MANUAL_ALUMNO.md) | Manual paso a paso: ruta A (AWS Learner Lab) y ruta B (tu computadora) |

Todo el material del laboratorio se publica en este repositorio.

## Cómo obtener el kit

### En tu instancia de AWS (ruta A), dentro del repositorio de tu equipo

```
wget -O kit.zip https://github.com/CyberProf-tecmi/lsca2318-lab-a3-a4/raw/main/kit_lab_A3_A4_alumno.zip
unzip -o kit.zip && rm kit.zip
echo "bitacora/usuarios.db" >> .gitignore
```

**Si tu `src/finnorte/herramientas.py` ya tiene `ejecutar_agente` funcionando**, no lo reemplaces. Usa este comando en lugar del `unzip` anterior:

```
unzip -o kit.zip -x src/finnorte/herramientas.py && rm kit.zip
```

### En tu computadora (ruta B)

1. Abre `kit_lab_A3_A4_alumno.zip` en esta página y da clic en **Download raw file**.
2. Descomprímelo en la raíz del repositorio de tu equipo y reemplaza los archivos que te pregunte.

### Después, en ambos casos

1. Haz commit y push para que tu equipo tenga el kit:
   ```
   git add . && git commit -m "lab: kit A3+A4" && git push
   ```
2. Revisa que `git status` **no** muestre `.env` ni `bitacora/usuarios.db`.

## Qué trae el kit

| Archivo | Qué hace | Actividad |
|---|---|---|
| `api/auth.py` | Registro, login y token: **5 funciones `TODO(LAB)`** por completar | A3 |
| `api/main.py` | Servicio con token, errores `problem+json`, límite de solicitudes, `/v1/variante` y `/v1/agente` | A3 · A4 |
| `src/finnorte/agente_servicio.py` | Agente con confirmación humana en dos pasos y validador | A4 |
| `src/finnorte/herramientas.py` | `ejecutar_agente` por completar (si aún no lo tienes) | A4 |
| `app/cliente_api.py` | Interfaz Streamlit que consume el servicio | A4 |
| `tests/test_lab_producto.py` | Pruebas del laboratorio: lo pendiente aparece como SKIP | A3 · A4 |
| `.env.example` | Variables necesarias, **sin llaves** | — |

## Reglas

- Tu llave de Gemini y el `TOKEN_SECRET` viven **solo** en `.env`, nunca en el repositorio. Una llave en el repositorio cuesta **−15** y se revoca.
- En AWS, los puertos 8000 y 8501 se abren **solo a My IP**.
- Todo uso de IA va en `ai_log.md`.
