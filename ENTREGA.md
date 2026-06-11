# Laboratorio de Observabilidad

## Verificacion inicial del stack

En esta primera parte prepare el proyecto base, verifique los requisitos de Docker, corregi el ajuste necesario para mi entorno Linux y levante el stack de observabilidad con Docker Compose.

### Clonacion del proyecto

Clone el repositorio base del laboratorio dentro de mi carpeta local de trabajo:

```bash
git clone https://github.com/UPAO-Recursos/iac-observabilidad.git .
```

![Clonacion del repositorio](evidencias/0.png)

Luego confirme que Docker y Docker Compose estuvieran disponibles en mi maquina:

```bash
docker --version
docker compose version
```

![Versiones de Docker y Docker Compose](evidencias/1.png)

### Ajuste inicial en Linux

Al ejecutar el stack por primera vez aparecio el siguiente error asociado al montaje de `node-exporter`:

```text
path / is mounted on / but it is not a shared or slave mount
```

![Error inicial de montaje](evidencias/3.png)

Revise el servicio `node-exporter` en `docker-compose.yml`. El volumen venia con la opcion `rslave`:

```yaml
- /:/host:ro,rslave
```

![Configuracion original del montaje](evidencias/4.png)

Para mi entorno Linux lo ajuste a:

```yaml
- /:/host:ro
```

![Configuracion corregida del montaje](evidencias/5.png)

Con ese cambio el stack pudo levantarse correctamente.

### Levantamiento del stack

Ejecute el comando principal del laboratorio:

```bash
docker compose up -d --build
```

La construccion de las imagenes del backend y frontend se completo correctamente.

![Construccion del stack](evidencias/6.png)

Al finalizar, Docker Compose creo y levanto los contenedores del laboratorio.

![Stack levantado](evidencias/7.png)

Despues verifique el estado general con:

```bash
docker compose ps
```

En la salida se observan los servicios principales arriba: `lab-backend`, `lab-frontend`, `lab-grafana`, `lab-prometheus`, `lab-loki`, `lab-alloy`, `lab-cadvisor` y `lab-node-exporter`.

![Estado de contenedores](evidencias/8.png)

Tambien deje registrado el commit local del ajuste de Linux:

```bash
git add docker-compose.yml
git commit -m "fix: corrige montaje de node exporter en linux"
```

![Commit del ajuste Linux](evidencias/9.png)

### Servicios accesibles

Verifique el frontend en el navegador desde:

```text
http://localhost:8080
```

La pagina del laboratorio cargo correctamente y pude ejecutar el boton **Saludar (API)**, lo que confirma la comunicacion con el backend.

![Frontend funcionando](evidencias/front1.png)

Tambien probe el boton de carga de CPU desde el frontend. Esta accion se usara mas adelante para validar la alarma de CPU.

![Frontend generando carga](evidencias/front2.png)

Verifique el endpoint de metricas del backend en:

```text
http://localhost:3001/metrics
```

La respuesta muestra metricas en formato Prometheus, incluyendo metricas de CPU, memoria y estado del proceso del backend.

![Metricas del backend](evidencias/Backend.png)

Verifique que Grafana estuviera accesible en:

```text
http://localhost:3000
```

Primero se muestra la pantalla de login.

![Login de Grafana](evidencias/Grafana1.png)

Luego ingrese con el usuario `admin` y confirme que Grafana cargara correctamente.

![Grafana accesible](evidencias/grafana2.png)

Finalmente, verifique que Prometheus estuviera disponible en:

```text
http://localhost:9090
```

![Prometheus accesible](evidencias/Prometheus.png)

Los servicios esperados para continuar el laboratorio son:

| Servicio | URL | Estado esperado |
|---|---|---|
| Frontend | `http://localhost:8080` | Pagina "Hello World" con botones |
| Backend | `http://localhost:3001/metrics` | Metricas en formato Prometheus |
| Grafana | `http://localhost:3000` | Login con usuario `admin` |
| Prometheus | `http://localhost:9090` | Interfaz web y consulta de targets |
| Loki | `http://localhost:3100` | Servicio de almacenamiento de logs |
| Alloy | `http://localhost:12345` | Estado del recolector de logs |
| cAdvisor | `http://localhost:8081` | Metricas de contenedores |
| node-exporter | `http://localhost:9100/metrics` | Metricas del host |
