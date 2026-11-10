---
tags:
  - middleware
  - mq
  - instana
  - ibm-i
  - cheatsheet
  - as400
  - bda
---

# Procedimiento

## 1. Revisión de MQ

* **Login > `# wrkmqm`** 📸
  * **8** — Estado
  * **18** — Colas
  * **20** — Canales
  * **22** — Trabajos 📸
  * **27** — Escuchas
  * **23** — Logs:
    * Opción **5** para visualizar `AMQERR01.LOG`
    * Presionar tecla **b** para ir al final del archivo

> [!tip] Atajos de Navegación
> * **Mostrar más teclas:** `Shift + Fn + F12` y `Shift + Fn + F11`
> * **Regresar:** `Fn + F12`
> * **Salir:** `Fn + F3`

---
## Comandos del Sistema Operativo

* Versión MQ 📸: `QMQM/DSPMQMVER`
* Estado del sistema 📸: `WRKSYSSTS`
* Respaldo de Configuraciones y réplica activa (CONT) 📸: 
```
	RUNQRY QRYFILE((MQBKBDA/QMQSC MQWED2))
```
* Listar sincronización de respaldos (CONT) 📸:
```
	qsh
	ls -ltr /QSYS.LIB/MQBKBDA.LIB/QMQSC.FILE
```
* ~~`wrkmqmlsr` — Escuchas a nivel de SO~~
* ~~`wrkdsksts` (o en QSH: `df -g`) — Estado de discos 📸~~

---
## 3. Instana — Dashboards de consumo de Recursos

> [!info] Configuración
> Usar **TimeRange = 24h** según el ambiente a revisar.  
> Dashboards: [Custom dashboards – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/customDashboards?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false) y [Explore – Infrastructure – Analytics – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/explore;group=\(\)~;type=ibmIOs;dataSource=infrastructure;groupBy=!~;tagFilterExpression=!~;order=\(by~label~direction~DESC\)~;query?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false)

* **Producción:**
	* [Custom dashboard: Dash_JP_MQ_PRODBDA – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/customDashboards/view;dashboardId=xoWQsJtnTnm9qVUJemsKtA?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false)
* **Desarrollo:**
	* [Custom dashboard: Dash_JP_MQ_DESABDA – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/customDashboards/view;dashboardId=tXZSpNEVQEWkwQI2SmCc-g?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false)
	* [IBM i: IBM i @BDADESA.BDA – Infrastructure – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/physical/dashboard?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false&snapshotId=KLEj1VHtCUMCftPxwpGS_Lxq7uM)
* **Contingencia:**
	* [Custom dashboard: Dash_FH_MQ_CONTNBDA – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/customDashboards/view;dashboardId=ZBW-5rKuSqmCLXoIGSqEhw?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false)
	* [IBM i: IBM i @BDACONTN.BDA – Infrastructure – Instana (gbmobs-bdapa)](https://gbmobs-bdapa.instana.io/#/physical/dashboard?timeline.ws=86400000&timeline.to&timeline.fm&timeline.ar=false&snapshotId=ABYrIFWLuyULlr0DZ-o8ZqdF8ZI)