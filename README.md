# Despliegue de servicios de red: consulta de acciones con Flask, Docker y AWS

Taller de la Maestría en Computación Distribuida y Frameworks para Desarrollo de Aplicaciones Inteligentes (Universidad del Valle), Actividad 7: Despliegue de Servicios.

El proyecto toma un script en Python que consulta el precio de una acción en Yahoo Finance, lo empaqueta en un contenedor Docker, lo expone como servicio web con Flask, lo publica en Docker Hub y lo despliega en una instancia EC2 de AWS.

## Contenido

| Archivo | Descripción |
|---|---|
| `read_value_yf.py` | Script que descarga con `yfinance` los datos de una acción desde 2020-01-01 hasta hoy. Recibe el ticker como argumento (por defecto `GOOGL`). |
| `app.py` | Servicio web Flask que expone el resultado del script como tabla HTML en `/`. |
| `historical_data.py` | Script auxiliar de consulta de datos históricos. |
| `Dockerfile` | Define la imagen del servicio (`python:3.11.3-bullseye`, puerto 5000). |
| `requirements.txt` | Dependencias de Python (`Flask`, `gunicorn`, `yfinance`). |

## Requisitos

- Docker (Docker Desktop en Windows con WSL2)
- Cuenta en Docker Hub
- Cuenta en AWS (para la parte de nube)

## Ejecución local

### Versión 1.0: script en contenedor

El `Dockerfile` de esta versión usa como comando `python3 read_value_yf.py ${STOCK}` y la variable de entorno `STOCK` (por defecto `GOOGL`).

```bash
docker image build -t stock-market:1.0 .
docker container run --rm stock-market:1.0
docker container run --rm -e STOCK=AMZN stock-market:1.0
```

### Versión 2.0: servicio web con Flask

El `Dockerfile` actual construye esta versión.

```bash
docker image build -t stock-market:2.0 .
docker container run --rm --name stock-market -p 5000:5000 stock-market:2.0
```

Abrir `http://localhost:5000` en el navegador para ver la tabla de GOOGL.

## Publicación en Docker Hub

Reemplazar `TU_USUARIO` por el usuario de Docker Hub.

```bash
docker login
docker image tag stock-market:2.0 TU_USUARIO/stock-market:2.0
docker image push TU_USUARIO/stock-market:2.0
```

## Despliegue en AWS EC2

1. Crear una instancia EC2 con Ubuntu (elegible para capa gratuita), un par de llaves `.pem` y habilitar **Allow HTTP traffic from the internet**.
2. Conectarse por SSH:

   ```bash
   ssh -i demo_docker.pem ubuntu@DNS_PUBLICO_DE_LA_INSTANCIA
   ```

3. Instalar Docker en la instancia (guía oficial: https://docs.docker.com/engine/install/ubuntu/) y habilitar al usuario `ubuntu` para usarlo:

   ```bash
   sudo usermod -aG docker ubuntu
   ```

   Cerrar la sesión SSH y volver a entrar para que el cambio aplique.

4. Ejecutar el contenedor mapeando el puerto 80 de la instancia al 5000 del contenedor:

   ```bash
   docker container run -d --name stock-market -p 80:5000 TU_USUARIO/stock-market:2.0
   ```

5. Abrir `http://DNS_PUBLICO_DE_LA_INSTANCIA` en el navegador (con `http`, no `https`).

## Limpieza

Al terminar, **terminar la instancia** en la consola de EC2 (Instance state, Terminate instance) para no consumir créditos de AWS.

## Notas

- El servidor de Flask usado es el de desarrollo; no es adecuado para producción.
- Los avisos `Pandas4Warning` que imprime `yfinance` son de deprecación y no afectan el resultado.
- No subir al repositorio las llaves `.pem` ni credenciales; el `.gitignore` las excluye.

## Referencias

- https://docker-curriculum.com/
- https://docs.docker.com/engine/install/ubuntu/
