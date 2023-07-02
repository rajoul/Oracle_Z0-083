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

duplixed buckup:
	You cannot back up from tape to tape or from tape to disk: only from disk to disk or disk to tape.

### Flash recovery area:
serves as the default storage area for all files related to backup and restore operations

FRA can contain:
	- datafiles copies
	- archived redo log
	- control files
	- flashback logs

we can create a FRA:
```
ALTER SYSTEM SET DB_RECOVERY_FILE_DEST_SIZE = 2G SCOPE=BOTH
ALTER SYSTEM SET DB_RECOVERY_FILE_DEST = 'C:\ORACLE\RECOVERY_AREA' SCOPE=BOTH
```

- BACKUP RECOVERY AREA: can recover all files in the FRA that are not backuped


Oracle Flashback Transaction: use UNDO TBS
	- Query to retrieve metadata and historical data for a given transaction or for all transactions in a given time interval. 
	- Oracle Flashback Transaction Query queries the static data dictionary view FLASHBACK_TRANSACTION_QUERY

Flashback data archive: consists of one or more tablespaces
	- FDA need to be enabled for any table
	- you need to u have the FLASHBACK ARCHIVE object privilege
	- The table is not nested, temporary, remote, or external
	- Flashback Data Archive supports only these DDL statements:
		■ ALTER TABLE statement that does any of the following:
			– Adds, drops, renames, or modifies a column
			– Adds, drops, or renames a constraint
			– Drops or truncates a partition or subpartition operation
		■ TRUNCATE TABLE statement
		■ RENAME statement that renames a table

flashback version query: use UNDO
	- retrieve the different versions of specific rows that existed during a given time interval
	- this query use FVQ
```
SELECT versions_xid XID, versions_startscn START_SCN,
 versions_endscn END_SCN, versions_operation OPERATION,
 empname, salary
FROM emp
VERSIONS BETWEEN SCN MINVALUE AND MAXVALUE
WHERE empno = 111;



Results are similar to:
XID START_SCN     END_SCN O EMPNAME SALARY
---------------- ---------- ---------- - ---------------- ----------
09001100B2200000 10093466 I Tom 927
030002002B210000 10093459 D Mike 555
0800120096200000 10093375 10093459 I Mike 555
```

- flashback logs:
	- files that allow your database to be rollbacked.
	- are automatically purged when DB_FLASHBACK_RETENTION_TARGET < the time they have already been retained.
	- DBA can't delete them
	- are deleted when retention < the time they have already been retained. + pressure space


## multitenant

									CDB$ROOT
										|
		-----------------------------------------------------------------------------------------
		|				|				|														|
	PDB$SEED   		   PDB1			APP1: (AS application container) mini CDB 				   PDB4
											|
							--------------------------------------------
							|			|								|
						  SEED 		  PDB2							   PDB3


A user created in APP1 will have the same privilège under all its PDBs (PDB2, PDB3)
A user in CDB$ROOT can have different privilèges under PDBs.

- LOCAL_UNDO_ENABLED = false in a CDB:
	A. After the change, only a common user with the required privilege can create an undo tablespace in CDB&ROOT.
	B. Any new PDB and existing PDBs are automatically configured to use the default undo tablespace in CDB$ROOT.

- create pluggable database pdb1:
	- SYSTEM, SYSAUX and TEMP tablespaces are always included
	- specify which user tablespaces we can include with user_tablespaces

- Unplugging a database will result in the PDB remaining in a MOUNT state. It is still part of the CDB


PDB_FILE_NAME_CONVERT ?


- LOCKDOWN profile: allow to restrict access and  restrict user operations
	- The CREATE LOCKDOWN PROFILE statement must be issued from the CDB or the Application Root.
	- You must have the CREATE LOCKDOWN PROFILE system privilege in the container in which the statement is issued.

- clone or relocate a remote PDB to an existing CDB:
	- remote and local CDB must be in local undo mode
	- remote and local CDB must be in archive log mode
	- create a database link from local CDB to remote CDB

- Automatic Workload Repository (AWR) and Automatic Database Diagnostic Monitor (ADDM)


- local and common USERS / ROLES:
	1. local users/roles:
		- connect to a PDB
		- CREATE USER test_user3 IDENTIFIED BY password1 CONTAINER=CURRENT;
	2. common users:
		- connect to a CDB
		- create a role/user with a prefix 'C##' with CONTAINER=ALL;

- BACKUP CDB$ROOT, CDB$SEED:
	- connect to CDB
	- BACKUP PLUGGABLE DATABASE "CDB$ROOT", "PDB$SEED"
- BACKUP a PDB:
	- connect to PDB
	- BACKUP DATABASE





## Deploy and Patch management