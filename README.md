# Monitoreo

En este laboratorio exploraremos monitoreo con herramientas disponibles


## Aplicaciones
```bash
docker compose up -d --build
```

## Servicios y URLs
| Servicio       | URL                         | Notas                                  |
|----------------|-----------------------------|----------------------------------------|
| Frontend       | http://localhost:8080       | Hello World + botones de tráfico/carga |
| Backend (API)  | http://localhost:3001       | `/api/hello`, `/metrics`, `/load`      |
| Grafana        | http://localhost:3000       | admin / admin                          |
| Prometheus     | http://localhost:9090       | datasource ya provisionado             |
| Loki           | http://localhost:3100       | datasource ya provisionado             |
| Alloy (UI)     | http://localhost:12345      | estado del recolector de logs          |
| cAdvisor       | http://localhost:8081       | métricas por contenedor                |
| node-exporter  | http://localhost:9100/metrics | métricas del host                    |

## Configuraciones
- **Datasources** Prometheus y Loki (provisionados automáticamente).
- Logs etiquetados por Alloy con `tier=application` o `tier=infrastructure`.

## Actividad
- El **dashboard** (paneles de CPU + logs de app e infra).
- La **alarma** de CPU > 50%.

## Alarma de CPU del backend

Configure una alerta en Grafana llamada **CPU backend > 50%**. En mi entorno cAdvisor no expuso la etiqueta `name="lab-backend"`, por eso use el `id` real del contenedor backend en la consulta:

```promql
sum(rate(container_cpu_usage_seconds_total{id="/docker/bf8413fd5c89dcd830e2ba4dfb77fdf9a0e1adb93632eed96455d8262d26a24c"}[1m])) * 100
```

La condicion quedo como **IS ABOVE 50**, con etiqueta `severity = warning`, grupo de evaluacion **Backend CPU Alerts**, intervalo de `10s`, pending period de `30s` y contact point **Webhook Backend Lab**.

![Consulta y condicion de la alarma](evidencias/alarma-consulta-condicion.png)

![Evaluacion y etiquetas de la alarma](evidencias/alarma-carpeta-etiqueta-evaluacion.png)

![Contact point de la alarma](evidencias/alarma-contacto-webhook.png)

![Regla de alarma guardada](evidencias/alarma-regla-guardada.png)

## Prueba de la alarma

Genere carga de CPU en el backend y verifique que el panel superara el 50%. Luego confirme en Grafana Alerting el cambio de estado de la regla: **Pending**, **Firing** y regreso a **Normal**.

![CPU del backend por encima del 50%](evidencias/prueba-cpu-backend-supera-50.png)

![Alarma en estado Firing](evidencias/alarma-estado-firing.png)

![Alarma de vuelta a Normal](evidencias/alarma-vuelve-normal.png)

## Ciclo alarma a log

El contact point **Webhook Backend Lab** apunta a `http://backend:3001/alerts`. Al dispararse la alerta, Grafana envia el webhook al backend y el backend registra el evento `grafana_alert_received`, visible en Loki como log de aplicacion:

```logql
{tier="application"} | json | msg="grafana_alert_received"
```

![Ciclo alarma a log](evidencias/ciclo-alarma-log-aplicacion.png)

## Reset
```bash
docker compose down -v   # borra también dashboards/alarmas creados
```

> Nota de versiones: el tag `prom/prometheus:latest` apunta aún a la rama 2.x (LTS),
> por eso fijamos `v3.8.1`. Promtail EOL (2026-03-02); el recolector de logs
> es Grafana Alloy.
