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
📸: Siempre tomar nuevo screenshot con fecha/hora visible, para los otros 

## 1. Revisión de MQ

* **Login > `wrkmqm`** 📸
* **Opciones:**

	|Opción| Función  |
	| --- | --- |
	| 8   | Estado   |
	| 18  | Colas    |
	| 20  | Canales  |
	| 22  | Trabajos 📸 |
	| 27  | Listeners |
	| 23  | Logs 📸 |
	
> **Ver Logs**
> *Opción **5** para visualizar `AMQERR01.LOG`*
> *Presionar tecla **b** para ir al final del archivo*

> [!tip] Atajos de Navegación
> * **Mostrar más teclas:** `Shift + Fn + F12` y `Shift + Fn + F11`
> * **Regresar:** `Fn + F12`
> * **Salir:** `Fn + F3`

---
## Comandos del Sistema Operativo

* Versión MQ 📸: `QMQM/DSPMQMVER`
* Estado del sistema 📸: `WRKSYSSTS`
* *[Aplica si hay un mensaje por "receptores diarios que pueden eliminarse"]*
	* Analizar Biblioteca del QM 📸:
		```
		DSPLIB QMBDA.TELL
		WRKJRNA JRN(QMBDA.TELL/AMQAJRN) + Shift Fn* F3
		```
* *[CONT only]*
	* Respaldo de Configuraciones y réplica activa📸: 
		```
		RUNQRY QRYFILE((MQBKBDA/QMQSC MQWED2))
		```
	* Listar sincronización de respaldos 📸:
		```
		qsh
		ls -ltr /QSYS.LIB/MQBKBDA.LIB/QMQSC.FILE
		```
* ~~Escuchas a nivel de SO: `wrkmqmlsr`~~
* ~~Estado de discos 📸: `wrkdsksts` (o en QSH: `df -g`)~~

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