# Premisas de seguridad — Credit-Card-Fraud-Detection

Reglas que **todo cambio en este repositorio debe cumplir** antes de mergear.
Aplica igual a código escrito a mano y a código generado por un asistente de IA.

Stack: Python · scikit-learn / XGBoost / LightGBM · Streamlit · modelos `.pkl` vía Git LFS.

> Este repositorio es **público**. Todo lo que se commitea es visible para
> cualquiera, para siempre, aunque se borre después.

Hoy la aplicación se ejecuta en local y no expone API ni base de datos, así que su
superficie de ataque es pequeña. Las reglas de abajo son las que hay que respetar
si eso cambia — y sobre todo las de datos y modelos, que aplican ya.

---

## 1. Secretos y credenciales

- Ningún secreto en el código: ni claves de API, ni credenciales de Kaggle, ni
  tokens de despliegue, ni cadenas de conexión.
- Todo por variable de entorno, leído con `os.environ[...]` para que falle al
  arrancar si no está.
- En Streamlit Cloud, usar `st.secrets` / el panel de Secrets. `.streamlit/secrets.toml`
  **no se commitea**.
- Añadir al `.gitignore`: `.env`, `.streamlit/secrets.toml`, `kaggle.json`.
- Antes de commitear: `git diff --cached | grep -iE "key|token|secret|password"`.

---

## 2. Datos

El dataset de fraude son transacciones reales anonimizadas (PCA). Aunque vengan
anonimizadas, el trato es el de datos financieros:

- **`datos/` no se versiona.** El README documenta de dónde se descarga; el CSV no
  entra en el repositorio.
- Nada de datos reales, personales o de producción en el repo, ni siquiera como
  "ejemplo pequeño". Si hace falta una muestra, se genera sintética.
- Ninguna fila individual en capturas, logs, notebooks commiteados o el README.
  Solo agregados y métricas.
- Los `.pkl` de `test_split*` contienen porciones del dataset: verificar que lo
  que se sube por LFS no incluye datos que no deberían salir.

---

## 3. Modelos y deserialización

**`joblib.load` y `pickle.load` ejecutan código arbitrario al deserializar.** Un
`.pkl` no es un fichero de datos: es un programa. Cargar uno que no has generado
tú equivale a ejecutar un script que te han pasado.

- Cargar **solo** modelos generados por los scripts de este repositorio.
- **Nunca** cargar un `.pkl` subido por el usuario. Si algún día hay upload de
  modelos, usar un formato que no ejecute código (ONNX, o los `save_model` nativos
  de XGBoost/LightGBM).
- Los `.pkl` van por Git LFS (ya configurado en `.gitattributes`). Verificar la
  integridad tras clonar.
- Rutas de modelo y de datos: constantes del código, nunca construidas con entrada
  del usuario (evita `../../etc/passwd`).

---

## 4. Validación de entradas

- Todo `st.number_input` con `min_value` y `max_value` explícitos — ya se hace en
  `app_streamlit.py`, mantenerlo en los parámetros nuevos.
- Los `st.text_input` que acaben en un nombre de fichero: lista blanca de
  caracteres (`^[A-Za-z0-9_\-]{1,40}$`) y nunca concatenados a una ruta sin
  `os.path.basename` + comprobación de que el resultado sigue dentro del directorio
  esperado.
- Si se añade `st.file_uploader`: límite de tamaño, extensión comprobada y
  contenido parseado con `pd.read_csv` sobre un buffer, nunca `eval`/`pickle`.
- Nada de `eval`, `exec`, `os.system` o `subprocess` con `shell=True` sobre datos
  que vengan de fuera.

---

## 5. Si se despliega como servicio

Nada de esto hace falta mientras sea local. En cuanto se publique:

- **Autenticación** delante de la app. Streamlit no trae ninguna: usar Streamlit
  Cloud con acceso restringido, o un proxy con auth. Si hay que gestionar usuarios,
  delegar en un proveedor (Supabase Auth, OAuth), nunca implementar login propio.
- **Rate limiting**: la inferencia y el reentrenamiento son caros. ~10 req/min por
  usuario, `429` al superarlo.
- **SQL injection**, si aparece base de datos: consultas parametrizadas siempre,
  nunca f-strings con SQL.
- HTTPS obligatorio y mensajes de error genéricos hacia el cliente.

---

## 6. Dependencias

- Versiones fijadas en `requirements.txt`.
- `pip-audit` de forma periódica; Dependabot activado.
- Cuidado especial con `pickle`, `joblib`, `xgboost` y `lightgbm`: son las que más
  CVEs de deserialización acumulan.

---

## 7. Checklist antes de abrir una PR

- [ ] Cero secretos en el diff.
- [ ] Ningún dato real ni fila individual en el repo, capturas o notebooks.
- [ ] No se carga ningún `.pkl` de origen externo.
- [ ] Las rutas de fichero no se construyen con entrada del usuario.
- [ ] Los inputs numéricos tienen mínimo y máximo.
- [ ] `pip-audit` sin vulnerabilidades altas o críticas.

---

## Reportar un problema

Si encuentras un fallo de seguridad, no abras un issue público: escribe a
marcos.elbosque@gmail.com.
