# Oracle_Z0-083



## Backup and recovry

2 modes de backup:
	- user managed backup
	- rman

- mode archivelog: permet de faire une copie des fichiers redo log
- point-in-time recovery: permet la restauration de la BD à un time précis.
	steps:
		1. restore all database datafiles
		2. recover database until time '14 june 2023'

- Using RMAN there 2 types of backups:
	1. FULL backup
	2. INCREMENTAL backup:
		- cumulative backup
		- differential backup

duplixed buckup

## multitenant




## Deploy and Patch management