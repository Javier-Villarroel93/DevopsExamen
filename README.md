# Flask Auth + Calculadora (Villarroel)

Aplicacion Flask minimal que combina registro/login basico y una calculadora con interpretacion de lenguaje natural. Incluye contenedor Docker, pruebas automatizadas y pipeline CI/CD para publicar en GHCR y desplegar en un stack remoto.

## Uso local
1. Instala dependencias:
   ```bash
   pip install -r requirements.txt
   ```
2. Ejecuta:
   ```bash
   python app.py
   ```
3. Endpoints:
   - `POST /api/register` -> JSON `{"username": "ana", "password": "secreto"}` crea usuario (in-memory).
   - `POST /api/login` -> devuelve `token` ficticio si las credenciales son validas.
   - `POST /api/calc` -> calcula expresiones o frases sencillas.
   - `GET /health` -> estado.

## Pruebas
```bash
pytest
```

## Contenedor
Construye y prueba la imagen localmente:
```bash
docker build -t ghcr.io/<owner>/villarroel:1.0.5 .
docker run -p 5000:5000 ghcr.io/<owner>/villarroel:1.0.5
```

## Pipeline CI/CD (GitHub Actions)
- Rama de trabajo: `villarroel`.
- Flujo:
  1. `test`: instala dependencias y ejecuta `pytest`.
  2. `build_and_push`: construye y publica la imagen en GHCR con tags `1.0.5` y `latest` (`ghcr.io/<owner>/villarroel`).
  3. `deploy`: se conecta al VPS, obtiene la imagen y ejecuta `docker stack deploy` usando `stack.yml`.

### Secrets requeridos
- `GHCR_PAT`: token personal con `packages:write` para que el VPS haga pull.
- `VPS_HOST`, `VPS_USER`, `VPS_PASSWORD`, `VPS_SSH_PORT`: acceso SSH al VPS.
- `STACK_NAME`: nombre del stack a desplegar (ej. `villarroel-stack`).

### Variables en runtime del pipeline
- `IMAGE_VERSION` (default `1.0.5`), `APP_NAME` (`villarroel`), `GH_OWNER` (dueño del repo), `IMAGE_NAME` (`ghcr.io/<owner>/villarroel`).

## Stack de despliegue
`stack.yml` usa las variables `GH_OWNER` e `IMAGE_VERSION` para apuntar a la imagen publicada y expone el servicio por Traefik con el host `villarroel.byronrm.com`. El pipeline copia el archivo y ejecuta:
```bash
IMAGE_VERSION=1.0.5 GH_OWNER=<owner> docker stack deploy -c stack.yml <STACK_NAME>
```

## Subdominio
Asegura que `villarroel.byronrm.com` apunte al VPS donde corre el stack para completar la evidencia del despliegue.
