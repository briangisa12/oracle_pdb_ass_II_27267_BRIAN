# oracle_pdb_ass_II_27267_BRIAN
Assignment II – Oracle Pluggable Databases (PDB)

Student Name:	Brian GANZA GISAGARA 

Student ID:	27267

Course:	PL/SQL 

Date	22 September 2026

1. Overview of Tasks
Task	                          Description	Result

Task 1	Create a new Pluggable Database and a user inside it	✅ BR_PDB_27267 created, user BRIAN_PLSQLAUCA_27267 created inside it

Task 2	Create a temporary PDB, verify it, delete it completely, and confirm deletion	✅ BR_TO_DELETE_PDB_27267 created, verified, dropped (including datafiles)

Task 3	Access Oracle Enterprise Manager / a monitoring dashboard showing the environment, PDB tasks and username	✅ Done using SQL Developer DBA Instance Viewer (see Challenge 5)

Task 4	Document all work in this README	✅ This file

2. Oracle Environment Used
Item	                        Value

Operating System	Windows 11 Pro Education (64-bit)

Database	Oracle AI Database 26ai Free (reports internal version 23.0.0.0)

Container Database (CDB)	FREE (service name FREE)
Default PDB	FREEPDB1

Listener	localhost:1521

Datafile location	C:\APP\ORACLE\ORADATA\FREE\

Client tool	Oracle SQL Developer

Monitoring dashboard	SQL Developer → DBA panel → Instance Viewer

SQL Developer Connections

Connection Name	Username	Role	Service Name	Purpose

SYS_CDB	sys	SYSDBA	FREE	Administration in the CDB root

brian_pdb	brian_plsqlauca_27267	default	br_pdb_27267	Working user inside the PDB

4. Explanation of Each Task
   
**Task 1 – Create a New Pluggable Database**

Naming conventions used

Item	        Value

PDB Name	 br_pdb_27267

Username   inside PDB	brian_plsqlauca_27267

Password	 ************

Step 1 – Connect as SYS and confirm we are in the root container

SHOW CON_NAME;   -- Result: CDB$ROOT

Step 2 – Find the location of the seed datafiles (con_id = 2 is PDB$SEED)

SELECT name FROM v$datafile WHERE con_id = 2;
-- Result: C:\APP\ORACLE\ORADATA\FREE\PDBSEED\SYSTEM01.DBF ...

Step 3 – Create the PDB from the seed

CREATE PLUGGABLE DATABASE br_pdb_27267
  ADMIN USER br_admin IDENTIFIED BY ********
  FILE_NAME_CONVERT = (
    'C:\APP\ORACLE\ORADATA\FREE\PDBSEED\',
    'C:\APP\ORACLE\ORADATA\FREE\BR_PDB_27267\'
  );
ADMIN USER is mandatory when creating a PDB from the seed; it becomes the PDB's local administrator.
FILE_NAME_CONVERT tells Oracle where to copy the seed datafiles for the new PDB.

Step 4 – Open the PDB and keep it open after restarts

ALTER PLUGGABLE DATABASE br_pdb_27267 OPEN;
ALTER PLUGGABLE DATABASE br_pdb_27267 SAVE STATE;
SELECT name, open_mode FROM v$pdbs;
Result:

NAME              OPEN_MODE
PDB$SEED          READ ONLY
FREEPDB1          READ WRITE
BR_PDB_27267      READ WRITE

Step 5 – Switch into the PDB

ALTER SESSION SET CONTAINER = br_pdb_27267;
SHOW CON_NAME;   -- Result: BR_PDB_27267

Step 6 – Create the user inside the PDB and grant privileges

CREATE USER brian_plsqlauca_27267 IDENTIFIED BY ********;

GRANT CONNECT, RESOURCE, CREATE SESSION TO brian_plsqlauca_27267;
GRANT CREATE TABLE, CREATE VIEW, CREATE PROCEDURE, CREATE SEQUENCE TO brian_plsqlauca_27267;
GRANT UNLIMITED TABLESPACE TO brian_plsqlauca_27267;

Step 7 – Verify the user is local to the PDB

SELECT username, common, created
FROM dba_users
WHERE username = 'BRIAN_PLSQLAUCA_27267';
Result:

USERNAME                COMMON  CREATED
BRIAN_PLSQLAUCA_27267   NO      22-SEPT-26
COMMON = NO proves the user is a local user created inside the PDB, not a common user in the root.


Step 8 – Log in as the new user

SELECT USER, SYS_CONTEXT('USERENV','CON_NAME') AS pdb FROM dual;
Result:

USER                    PDB
BRIAN_PLSQLAUCA_27267   BR_PDB_27267
Requirements

done
PDB created successfully
done
User created inside the PDB

**Task 2 – Create and Delete a PDB**

Temporary PDB name: br_to_delete_pdb_27267

Step 1 – Return to the root container (PDBs can only be created/dropped from CDB$ROOT)

ALTER SESSION SET CONTAINER = CDB$ROOT;
SHOW CON_NAME;   -- Result: CDB$ROOT
Step 2 – Create the temporary PDB and verify it exists

CREATE PLUGGABLE DATABASE br_to_delete_pdb_27267
  ADMIN USER br_temp_admin IDENTIFIED BY ********
  FILE_NAME_CONVERT = (
    'C:\APP\ORACLE\ORADATA\FREE\PDBSEED\',
    'C:\APP\ORACLE\ORADATA\FREE\BR_TO_DELETE_PDB_27267\'
  );

ALTER PLUGGABLE DATABASE br_to_delete_pdb_27267 OPEN;

SELECT name, open_mode FROM v$pdbs;
Result: BR_TO_DELETE_PDB_27267 appears in the list with READ WRITE.

Step 3 – Delete the PDB completely and confirm

ALTER PLUGGABLE DATABASE br_to_delete_pdb_27267 CLOSE IMMEDIATE;

DROP PLUGGABLE DATABASE br_to_delete_pdb_27267 INCLUDING DATAFILES;

SELECT name, open_mode FROM v$pdbs;
Result:

Pluggable database BR_TO_DELETE_PDB_27267 altered.
Pluggable database BR_TO_DELETE_PDB_27267 dropped.
NAME              OPEN_MODE
PDB$SEED          READ ONLY
FREEPDB1          READ WRITE
BR_PDB_27267      READ WRITE
A PDB must be closed before it can be dropped.
INCLUDING DATAFILES removes the PDB's files from disk, so the deletion is complete.
BR_PDB_27267 from Task 1 was not affected.
Requirements

done
Create the PDB successfully
done
Verify that it exists
done
Delete the PDB completely
done
Confirm that it no longer exists

**Task 3 – Oracle Enterprise Manager (Dashboard)**

!!! Oracle removed Enterprise Manager Database Express (EM Express) starting with Oracle Database 23ai, so it is not available in the 26ai Free installation used here. The equivalent built-in monitoring dashboard in Oracle SQL Developer was used instead.

Steps

In SQL Developer: View → DBA.
In the DBA panel, click + and add the SYS_CDB connection.
Database Status → Instance Viewer – the live dashboard showing the FREE instance, version, uptime, host OS, sessions, processes, waits, CPU and top SQL.
Container Database – lists BR_PDB_27267 and FREEPDB1; BR_TO_DELETE_PDB_27267 no longer appears, which reflects Tasks 1 and 2. Opening BR_PDB_27267 shows its details (CON_ID 4, status NORMAL, created 22-SEPT-26).
To make the username visible on the dashboard screen, this query was run in the SYS_CDB worksheet beside the Instance Viewer:
SELECT p.name AS pdb_name, p.open_mode, u.username, u.common, u.created
FROM cdb_users u
JOIN v$pdbs p ON u.con_id = p.con_id
WHERE u.username = 'BRIAN_PLSQLAUCA_27267';
Result:

PDB_NAME       OPEN_MODE   USERNAME                COMMON  CREATED
BR_PDB_27267   READ WRITE  BRIAN_PLSQLAUCA_27267   NO      22-SEPT-26
Requirements

done
Dashboard is accessible
done
Dashboard reflects the Oracle environment
done
Dashboard reflects completed PDB tasks
done
Username visible on dashboard

**4. Challenges Faced and How They Were Solved**

#	           **Challenge**	                                                                                          **Cause**	                                                                                                 **Solution**

1	ORA-12541: Cannot connect. No listener at host localhost port 1521	 No Oracle Database server was installed on the computer (SQL Developer is only a client).	    Installed Oracle AI Database 26ai Free using setup.exe (run as administrator), to a path without spaces (C:\app\oracle\).

2	SYS connection would not work with default settings	SYS must connect with the SYSDBA role, and the CDB service name is FREE, not orcl.	Set Role = SYSDBA and Service name = FREE in the connection.

3	ORA-65005: missing or invalid file name pattern for file when creating the PDB	FILE_NAME_CONVERT used the path ...\ORCL\PDBSEED\, but the real seed path is ...\FREE\PDBSEED\.	Checked the real path with SELECT name FROM v$datafile WHERE con_id = 2; and changed ORCL to FREE.

4	A new PDB has no USERS tablespace, so the new user could not store data	PDBs created from the seed only include SYSTEM, SYSAUX, UNDO and TEMP.	Granted UNLIMITED TABLESPACE to the user.

5	The SYS_CDB session was still inside BR_PDB_27267 after Task 1	ALTER SESSION SET CONTAINER lasts for the whole session.	Ran ALTER SESSION SET CONTAINER = CDB$ROOT; before Task 2.

6	OEM Database Express was not available (nothing on port 5500)	EM Express was removed by Oracle starting with 23ai.	Used SQL Developer's DBA → Instance Viewer dashboard plus a cdb_users query to show the environment, PDBs and username.

**5. Integrity Statement**

I confirm that all the tasks in this assignment were carried out by me on my own computer, and that every command, result and screenshot in this repository comes from my own Oracle environment. I used an AI assistant for guidance and for help troubleshooting errors. I ran every command myself, checked each result, and understand what each step does. Passwords have been left out of this document on purpose.

**6. Submission Details**

Repository Link:     [https://github.com/briangisa12/oracle_pdb_ass_II_27267_BRIAN]

PDB Name Created:    br_pdb_27267

Issues Encountered:  Yes
