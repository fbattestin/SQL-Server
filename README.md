http://lti.cs.vt.edu/NewKA/OpenDSA/Books/CS2114/html/IntroDSA.html

# Partitioned Tables

https://www.red-gate.com/simple-talk/sql/database-administration/partitioned-tables-indexes-and-execution-plans-a-cautionary-tale/?article=1587


# SQL-Server

SET STATISTICS IO ON
https://www.simple-talk.com/sql/performance/simple-query-tuning-with-statistics-io-and-execution-plans/

INDEX INTERNALS
http://www.sqlskills.com/blogs/kimberly/indexes-in-sql-server-20052008-part-2-internals/

White Paper Data Compression - SQL Server 2008r2
https://technet.microsoft.com/en-us/library/dd894051(v=sql.100).aspx

Hotfix List
http://sqlserverbuilds.blogspot.com.br/

# PAGELATCH_XX
https://sqlperformance.com/2015/10/sql-performance/knee-jerk-wait-statistics-pagelatch

https://support.microsoft.com/en-us/help/4013999/fix-significantly-increased-pagelatch-ex-contentions-in-sys-sysobjvalu

https://technet.microsoft.com/en-us/library/ms190969(v=sql.105).aspx

--PDF
https://www.google.com.br/url?sa=t&rct=j&q=&esrc=s&source=web&cd=5&ved=0ahUKEwjKgpnPg4rUAhUDx5AKHekVCYsQFghGMAQ&url=http%3A%2F%2Fdownload.microsoft.com%2Fdownload%2Fb%2F9%2Fe%2Fb9edf2cd-1dbf-4954-b81e-82522880a2dc%2Fsqlserverlatchcontention.pdf&usg=AFQjCNFsTLQUKg-Nqi1JQ2aitXs5kXsuwA



# Filegroups

Moving an existing Index to another filegroup
https://docs.microsoft.com/en-us/sql/relational-databases/indexes/move-an-existing-index-to-a-different-filegroup

As you develop your index design strategy, you should consider the placement of the indexes on the filegroups associated with the database. Careful selection of the filegroup or partition scheme can improve query performance.

By creating the nonclustered index on a different filegroup, you can achieve performance gains if the filegroups are using different physical drives with their own controllers. Data and index information can then be read in parallel by the multiple disk heads. For example, if Table_A on filegroup f1 and Index_A on filegroup f2 are both being used by the same query, performance gains can be achieved because both filegroups are being fully used without contention. However, if Table_A is scanned by the query but Index_A is not referenced, only filegroup f1 is used. This creates no performance gain.
Because you cannot predict what type of access will occur and when it will occur, it could be a better decision to spread your tables and indexes across all filegroups. This would guarantee that all disks are being accessed because all data and indexes are spread evenly across all disks, regardless of which way the data is accessed. This is also a simpler approach for system administrators.

https://technet.microsoft.com/en-us/library/ms190433(v=sql.105).aspx

http://blogs.lessthandot.com/index.php/datamgmt/dbadmin/sql-server-filegroups-the-what/

Just to be clear, based on comments, this is because of parallelism at the I/O subsystem level (not one thread per data file, as that’s a myth, but being able to write to multiple points on the array during checkpoints)

As your databases get larger, it becomes more likely that you’re going to need multiple files and filegroups
Multiple filegroups give you enhanced possibilities for targeted disaster recovery, easier manageability, and I/O subsystem placement
Each filegroup should have 2-4 files at least, with tempdb being a special case

https://www.sqlskills.com/blogs/paul/files-and-filegroups-survey-results/



# Fragmentation

sys.dm_db_index_physical_stats
Logical Fragmentation:avg_fragmentation_in_percent
Page Density:page_space_used_in_percent
Page Count: Page_count
IN_ROW_DATA
SAMPLED


# REPLICATION
https://books.google.com.br/books?id=zBIngL29O8cC&pg=PA529&lpg=PA529&dq=Replication+Publishing+Model+Overview+sql+server+2005&source=bl&ots=dUwl5ZvxD8&sig=JRX4xGXt9fvoSYNvtJ-7NOLAVcc&hl=pt-BR&sa=X&ved=0ahUKEwijsq7oo_LPAhVIx5AKHbL6Cp8Q6AEIRjAF#v=onepage&q=Replication%20Publishing%20Model%20Overview%20sql%20server%202005&f=false

--Transacional Publication:
	- Somente tabelas que usam primary key;
	- Não aceita truncate (tabela origem, Publication)
	- Não gera LOCK (leitura de tlog via agent de replicação);
	- Ao realizar update na tabela replicada (Subscription) a replicação "quebra";
	- Baixo overhead;
	
--Transacional Publication with updatable subscriptions;
	- Somente tabelas que usam primary key;
	- Não aceita truncate (tabela origem, Publication)
	- Não gera LOCK (leitura de tlog via agent de replicação);
	- Aceita realizar update na tabela replicada (Subscription);
	- Alto overhead (devido a necessidade da criação de uma coluna do tipo uniqueidentifier);
	- Necessita de criação de linked server com impersonate entre as conta repllinkproxy e outra escolhida pelo usuário( conta dedicada).
	- cria coluna uniqueidentifier nas tabelas "articles" afim de manter a rastreabilidade dos updates no subscriber:
	
	All articles in a publication allowing updatable subscriptions contain a uniqueidentifier column named 'MSrepl_tran_version' used for tracking changes to the replicated data. SQL Server adds such a column to published tables that do not have one. 

	Adding a new column will:
	     » Cause INSERT statements without column lists to fail.
	     » Increase the size of the table. 

	SQL Server will add a uniqueidentifier column to each of the following tables when the publication is created
	
--Snapshot
	- Tabelas sem restrição de primary key;
	- Gera lock;
	- Replica DDL: Index Cluster - OK, Index Noncluster - NOK, Add Column - OK, Drop Column - OK, Alter Column - OK.
	- Overhead de recursos;
	- O job de replicação é iniciado a partir do server publication. 
	- Aceita update nos subscription sem oferecer risco para a integridade da replicação.
	- não aceita articles com uniqueidentifier (conforme solicitado para replicações Transacionais With Updatables Subscription (ERRO ABAIXO):
	Creating Publication
	- Creating Publication 'snap' (Success)
	SQL Server created publication 'snap'. 

	- Adding article 2 of 2 (Error)
	Article 'RET_TEST_SIMPLE' was added.  
	Messages
	SQL Server Management Studio could not create article 'REP_TEST'. (New Publication Wizard)
	------------------------------
	ADDITIONAL INFORMATION:
	An exception occurred while executing a Transact-SQL statement or batch. (Microsoft.SqlServer.ConnectionInfo)
	------------------------------
	Automatic identity range support is useful only for publications that allow updating subscribers.
	Changed database context to 'ReplicationTest_B'. (Microsoft SQL Server, Error: 21231)
	For help, click: http://go.microsoft.com/fwlinkProdName=Microsoft+SQL+Server&ProdVer=10.00.1600&EvtSrc=MSSQLServer&EvtID=21231&LinkId=20476
	- Starting the Snapshot Agent (Stopped)

	
--Merge Replication;
	- Alto Overhead (para criação do snapshot e para manutenção)

- All merge articles must contain a uniqueidentifier column with a unique index and the ROWGUIDCOL property. SQL Server adds a uniqueidentifier column to published tables that do not have one when the first snapshot is generated. 

	Adding a new column will:
	     » Cause INSERT statements without column lists to fail
	     » Increase the size of the table
	     » Increase the time required to generate the first snapshot 

	SQL Server will add a uniqueidentifier column with a unique index and the ROWGUIDCOL property to each of the following tables. 
	
	
	
https://msdn.microsoft.com/en-us/library/ms152567(v=sql.105).aspx

--artigo:
https://www.mssqltips.com/sqlservertip/3598/troubleshooting-transactional-replication-latency-issues-in-sql-server/

# tempdb

https://www.sqlskills.com/blogs/paul/tempdb-configuration-survey-results-and-advice/

http://social.technet.microsoft.com/wiki/contents/articles/31353.sql-server-demystifying-tempdb-and-recommendations.aspx

https://blogs.msdn.microsoft.com/sqlserverstorageengine/2009/01/11/tempdb-monitoring-and-troubleshooting-allocation-bottleneck/

https://support.microsoft.com/pt-br/kb/2154845

https://www.youtube.com/watch?v=SvseGMobe2w&feature=share&list=PLF80A8A233EE9F22F

--> CONTENTION
	Ref.:	https://technet.microsoft.com/pt-br/library/ms175195(v=sql.105).aspx
		http://www.sqlskills.com/blogs/paul/inside-the-storage-engine-gam-sgam-pfs-and-other-allocation-maps/
		http://www.sqlskills.com/blogs/paul/the-accidental-dba-day-27-of-30-troubleshooting-tempdb-contention/
		https://www.youtube.com/watch?v=SvseGMobe2w&feature=share&list=PLF80A8A233EE9F22F
		
	Global Allocation Map (GAM)
	As páginas GAM registram quais extensões foram alocadas. Cada GAM cobre 64.000 extensões ou quase 4 GB de dados. O GAM tem um bit para cada extensão no intervalo que cobre. Se o bit for 1, a extensão será livre; se o bit for 0, a extensão será alocada.
	Shared Global Allocation Map (SGAM)
	As páginas SGAM registram quais extensões estão sendo usadas atualmente como extensões mistas e também têm pelo menos uma página não usada. Cada SGAM cobre 64.000 extensões ou quase 4 GB de dados. O SGAM tem um bit para cada extensão no intervalo que abrange. Se o bit for 1, a extensão será usada como uma extensão mista e terá uma página livre. Se o bit for 0, a extensão não será usada como uma extensão mista, ou será uma extensão mista e todas as suas páginas estarão sendo usadas.
	
	Extent = 8 páginas de 8kb
	"Usualmente" SQL Server utiliza 1 extent para cada um Objeto, marcando o SGAM como SHARED ou MIXED extent
	
	Caso o SQL Server necessite criar um objeto no tempdb o algoritimo (RANDON ACCESS) irá procurar um por espaço livre em que possa alocar esses dados, o acesso a essas páginas - paginas que contém informações sobre os status das demais páginas de dados - (1,2,3, por exemplo 2:1:3) é realizado para endereçar a páginas de dados com espaço livre para alocação.
	A contenção pode acontecer quando há muitos acessos nessas páginas (automaticamente, muitas requisições de uso do tempdb) a procura de espaço para alocação. Sendo assim o número de waits do tipo PAGELATCH_ podem ser altos (importante realizar a o signal - total_wait).
	Para os LATCHS relacionados a SGAM/GAM o troubleshooting comum é a criação de novos datafiles.
	Pata os LATCHS relacionados a PFS habilitar o trace flag T1118
	
	PROPORCIONAL FILL - UNEVEN FILES
	O tempdb irá basear o custo ao inserir mais dados no maior arquivo de dados que contem maior número de espaço livre. 
	
	RECOMENDAÇÃO PARA ARQUIVOS DE DADOS:
	The best guidance I’ve seen is from a great friend of mine, Bob Ward, who’s the top Escalation Engineer in Microsoft SQL Product Support. Figure out the number of logical processor cores you have (e.g. two CPUS, with 4 physical cores each, plus hyperthreading enabled = 2 (cpus) x 4 (cores) x 2 (hyperthreading) = 16 logical cores. Then if you have less than 8 logical cores, create the same number of data files as logical cores. If you have more than 8 logical cores, create 8 data files and then add more in chunks of 4 if you still see PFS contention. Make sure all the tempdb data files are the same size too. (This advice is now official Microsoft guidance in KB article 2154845.)
	
	TLOG DIFFERENCE:
	
	
	
# RECOVERY MODEL (PERFORMANCE)
Baseline Script
	
		create table x (col1 int,col2 char(5000) not null)
		go
		begin tran

		declare @x int
		set @x = 0
		while (@x < 100000)
		begin
		insert into x values (1,'1')
		set @x = @x + 1
		end
		go
		--rollback
		drop table x

LAB:
	DB - Um Arquivo de Log 100MB, e Um arquivo de dados 100MB
	Tempo Recovery Model SIMPLE: 00:00:35
	Contador Perfmon: Log Bytes Flused/sec
	Comportamento: 100% durante a operação
	Durante ROLLBACK: ~35% a 40% (sem picos) - 00:01:10

	Database Name                  Log Size (MB) Log Space Used (%) Status
	------------------------------ ------------- ------------------ -----------
	master                         2.242188      46.34146           0
	tempdb                         0.7421875     53.68421           0
	model                          1.242188      35.22013           0
	msdb                           5.054688      19.24266           0
	AdventureWorks2012             0.9921875     37.79528           0
	DB_SERVER                      724.7422      99.14679           0

	CHECKPOINT 00:00:06
		
	Database Name               Log Size (MB) Log Space Used (%) Status
	--------------------------- ------------- ------------------ -----------
	master                      2.242188      46.34146           0
	tempdb                      0.7421875     53.68421           0
	model                       1.242188      35.22013           0
	msdb                        5.054688      19.24266           0
	AdventureWorks2012          0.9921875     37.79528           0
	DB_SERVER                   724.7422      0.4861643          0
	sqlnexus                    1.039063      52.25564           0

	DB - Um Arquivo de Log 100MB, e Um arquivo de dados. 100MB
	Tempo Recovery Model FULL: 00:00:32
	Contador Perfmon: Log Bytes Flused/sec
	Comportamento: 100% durante a operação
	Durante ROLLBACK: ~35% a 40% (com picos) - 00:01:11


	DB - Um Arquivo de Log 100MB, e Um arquivo de dados. 100MB
	Tempo Recovery Model BULK LOGGED: 00:00:30
	Contador Perfmon: Log Bytes Flused/sec
	Comportamento: 100% durante a operação
	Durante ROLLBACK: ~35% a 40% (com picos) - 00:01:12

	Database Name                 Log Size (MB) Log Space Used (%) Status
	----------------------------- ------------- ------------------ -----------
	master                        2.242188      59.58188           0
	tempdb                        0.7421875     58.42105           0
	model                         1.242188      33.33333           0
	msdb                          5.054688      17.85162           0
	AdventureWorks2012            0.9921875     58.66142           0
	DB_SERVER                     724.7422      99.78441           0


	CHECKPOINT:00:00:14
	Database Name                  Log Size (MB) Log Space Used (%) Status
	------------------------------ ------------- ------------------ -----------
	master                         2.242188      59.58188           0
	tempdb                         0.7421875     58.42105           0
	model                          1.242188      33.33333           0
	msdb                           5.054688      17.85162           0
	AdventureWorks2012             0.9921875     58.66142           0
	DB_SERVER                      724.7422      0.4780795          0

# WAITSTATS
WRITELOG Wait

O que significa?
- Aguardando o transaction log block buffer realizar o flush (desacarregar) os dados para disco.

Evitando respostas precipitadas:
- Problemas no sub sistema de disco (I/O) - Embora esse seja o problema mais frequente.

Analise:
- Correlacionar WRITELOG wait time com a latencia do sub sistema de disco.
	sys.dm_io_virtual_file_stats
	Olhar para LOGBUFFER waits, analisar a contenção interna para os log buffers.
- Olhar para a média de escrita em disco e a fila gerada (perfmon)
- Olhar o tamanho médio das transações.
- Investigar a frequencia em que são realizados page splits.


WRITELOG Wait Solutions

- Movimentar o log para um disco rápido e/ou rever arquitetura de replicação na camada de disco.
- Incrementar o tamanho das transações para pervinir grandes quantidades de blocos mínimos para flushes no disco
- Remover indices não clusterizados sem utilização para reduzir a sobrecarga no log na manutenção desse índices sem necessidade
durante operações de DML.
- Alterar as chaves dos indices ou alterar o FILLFACTOR para reduzir o page split.
- Se possível analisar a arquitetura de disco para uma melhor distribuição da carga de dados.

https://sqlperformance.com/2012/12/io-subsystem/trimming-t-log-fat

# SQL Server 2016 Technical Documentation
https://technet.microsoft.com/en-us/library/ms130214.aspx

SQL Server Utilities - Windows PS
https://github.com/DevNambi/SqlServerUtilities/tree/master/Save-SqlHistory

# Microsoft SQL Server BPA ( Best Pretices Analyzer ) 
https://www.microsoft.com/en-us/download/details.aspx?id=15289&751be11f-ede8-5a0c-058c-2ee190a24fa6=True
https://www.microsoft.com/en-us/download/details.aspx?id=29302
É necessário o Microsoft Baseline Configuration Analyzer 2.0
https://www.microsoft.com/en-us/download/details.aspx?id=16475


# Buffer Pool
http://www.sqlskills.com/blogs/paul/performance-issues-from-wasted-buffer-pool-memory/

# I/O

https://sqlperformance.com/2012/12/io-subsystem/trimming-t-log-fat

https://support.microsoft.com/pt-br/help/230785/description-of-logging-and-data-storage-algorithms-that-extend-data-reliability-in-sql-server


https://blog.docbert.org/queue-depth-iops-and-latency/
The end result of this is that the more paths a LUN has, the less important the HBA queue depth is. Most HBA's have a default queue depth of around 32, which if our storage can maintain a response time of 1ms (measured at the application) then we know that this is enough to generate up to around (32x1/0.001=) 32,000 IOPS. If we have only 1 path to the LUN then that's our maximum possibly number of IOPS, however the same LUN with 4 paths would be able to do over 100,000 IOPS to this LUN.

https://www.emc.com/collateral/white-papers/h15093-emc-unity-best-practices-guide.pdf
TRANSACTIONAL APPLICATIONS
Unity systems require high concurrency to deliver the maximum performance (IOPS). This is naturally achieved when connecting many
hosts with many LUNs. For systems that will be configured with only a few hosts and/or LUNs, host HBA settings may need to be
adjusted to increase concurrency. Consult the documentation for your OS or HBA on how to adjust LUN queue depth settings.

https://www.brentozar.com/archive/2013/09/iops-are-a-scam/
Combining IOPS, throughput, and latency numbers is a step in the right direction. It lets us combine activity (IOPS), throughput (MB/s), and performance (latency) to examine system performance. 

# Troubleshooting SQL Server I/O requests taking longer than 15 seconds – I/O stalls & Disk latency

http://www.sqlskills.com/blogs/paul/how-to-examine-io-subsystem-latencies-from-within-sql-server/
If you find that I/O latency has increased, look to a change in SQL Server behavior before blaming the I/O subsystem. For instance:

Query plan changes from out-of-date statistics, code changes, implicit conversions, poor indexing that cause table scans rather than index seeks
Additional indexes being created that cause increased index maintenance workload and logging
Access pattern/key changes that cause page splits and hence extra page reads/writes and logging
Adding change data capture or change tracking that causes extra writes and logging
Enabling snapshot isolation that causes tempdb I/Os, plus potentially page splits
Decreased server memory leading to a smaller buffer pool and increased lazy writer and read activity


https://blogs.msdn.microsoft.com/sqlsakthi/2011/02/09/troubleshooting-sql-server-io-requests-taking-longer-than-15-seconds-io-stalls-disk-latency/


# Connecting Troubleshooting

JBOSS - Network Packet Recieved - Chimney TCP/IP
http://www.sqlsolutionsgroup.com/tcp-chimney-offloading/
https://support.microsoft.com/pt-br/kb/951037

--Camada de aplicação / VM Host
https://social.technet.microsoft.com/Forums/windowsserver/en-US/d90e5265-de70-4ea0-9396-401a0eacdde7/disable-ipv4-checksum-offload-ipv4-large-send-offload-and-tcp-connection-offload-ipv4-on-server?forum=winserverhyperv
"How you do it depends on the UI that you have.  (netsh when you don't have a GUI)
You want to disable "TCP Task Offload" options in the driver settings of the NIC - these are driver specific.  What options you have is driver specific.
You do not want to disable "TCP Offload" - which is generally done using a single registry key at the Windows system level.
When do you disable Task Offloading options?  When you have an application in a VM that can benefit from it - Exchange, SQL, Terminal Services, other client server app, etc.  It is all about how the packets flow and how good the vendor implementation of TCP Task Offloading is.
The netsh command you outline generally has no impact and you have to go into the settings of the NIC.
There is an old Virtual Server document that outlines this pretty well (the sceanrio is a bit different).  Check out Method 3: http://support.microsoft.com/kb/888750/en-us
 Also:
http://itproctology.blogspot.com/2009/07/more-about-chimney-and-tcpoffload-in.html
http://itproctology.blogspot.com/2008/05/hyper-v-tcpoffloading-poor-network.html
Brian Ehlert (hopefully you have found this useful) http://ITProctology.blogspot.com"

JDBC: packetsize - 
https://msdn.microsoft.com/en-us/library/ms378988(v=sql.110).aspx

# Missing Files ( Troubleshooting)

--> Artigo: https://support.microsoft.com/en-us/kb/969052

SINTOMAS:
-------------------------------------------------------------------------------------------------------------------
you may encounter the following error messages, and these *may indicate Windows Installer Cache problems*.
TALVEZ possa indicar problemas(falta) dos arquivos *.msi e/ou *.msp no cache: C:\Windows\Installer\

C:\Windows\Installer\ stores important files for applications installed using the Windows Installer technology and should not be deleted. If the installer cache has been compromised, * you may not immediately see problems until you perform an action such as uninstall, repair, or update SQL Server. * 
IMPORTANTE: Nem todos arquivos informados pelo script fornecido são necessários para realizar a atividade de update de hotfix. Por exemplo, se existirem mais de uma instancia com versoes diferentes (2008R2 e 2012) podem ser gerados arquivos referente a um update da versão em que não esta sendo atualizada.

When you install SQL Server, the Windows Installer stores critical files in the Windows Installer Cache (default is C:\Windows\Installer). These files are required for uninstalling and updating applications. Missing files cannot be copied between computers, because they are unique.


RECOMENDACAO MICROSOFT:  
-------------------------------------------------------------------------------------------------------------------
You should run the repair from the original installation media, using the following command line:

setup.exe /ACTION=REPAIR /INDICATEPROGRESS=TRUE

Repair the common shared components and features first, and then repeat the command to repair the instances installed. During the repair process, the setup dialog box disappears. As long as the progress window does not show an error, the repair process is proceeding as expected.

1#  Repair shared components and features.
2#  Repair Instances.

Realizar Repair da instancia, utilizando o arquivo original de instalacao. O problema é que para que o REPAIR seja realizado, esses arquivos do CACHE tambem sao necessarios. O que foi observado é que ao executar o REPAIR de uma instancia onde nenhum arquivo indicado como MISSING FILES foram localizados (foi executado o VBS FindSQLInstalls), ao realizar o REPAIR uma referencia de arquivo *.msi/*.msp foi solicitada, no dialogBox o endereco original do arquivo apontava para um diretorio com nome em formato aparente de HASH, como os diretorios criados pelos Hotfix ao serem executados (realizam a extracao do pacote de atualizacao no diretorio em que sao executados, e geram nomes randomicos com apararencia de HASH). 

CAUSA:
-------------------------------------------------------------------------------------------------------------------
These problems may occur when the Windows Installer database file (.msi) or the Windows Installer patch file (.msp) is missing from the Windows Installer cache. The Windows Installer cache is located in the following folder:
%windir%\installer
When a product is installed by using Windows Installer, a stripped version of the original .msi file is stored in the Windows Installer cache. 

!!!!!!!! -> Every update to the product such as a hotfix, a cumulative update, or a service pack setup, also stores the relevant .msp or .msi file in the Windows Installer cache. <- !!!!!!!

Any future update to the product such as a hotfix, a cumulative update, or a service pack setup, !!!! relies !!!! on the information in the files that are stored in the Windows Installer cache. Without this information, the new update cannot perform the required transformations. --> Nesse caso, a referencia para esses arquivos estão escritos no registro do Windows.

WORKAROUND:
-------------------------------------------------------------------------------------------------------------------
Existem dois cenarios de MISSING FILES que sao causado pela ausencia de arquivos de extensões diferentes.

Cenário1: Missing installer files
Cenário2: Missing patches

Definicao:
Missing installer files - arquivos *.msi, a origem desse MISSING FILE será um diretório de INSTALACAO e o diretório de origem será C:\Windows\Installer\ e o nome desse arquivo será um código atribuido a esse arquivo original, por exemplo, Origem: ..\sql_engine_core_inst.msi Destino:C:\WINDOWS\Installer\19b4d2.msi 

A chave de registro que armazena essas informacoes de instalacao:
HKEY_CLASSES_ROOT\Installer\Products\

A saída abaixo refere-se ao FindSQLInstalls.vbs
================================================================================
PRODUCT NAME   : Microsoft SQL Server 2008 Database Engine Services
================================================================================
  Product Code: {9FFAE13C-6160-4DD0-A67A-DAC5994F81BD}
  Version     : 10.2.4000.0
  Most Current Install Date: 20110211
  Target Install Location: 
  Registry Path: 
   HKEY_CLASSES_ROOT\Installer\Products\C31EAFF906160DD46AA7AD5C99F418DB\SourceList
     Package    : sql_engine_core_inst.msi
  Install Source: \x64\setup\sql_engine_core_inst_msi\
  LastUsedSource: m;1;G:\x64\setup\sql_engine_core_inst_msi\
  
  !!! Verificar as chaves de registro que são lidas !!!
  
Missing patches - arquivos *.msp originados de atualizacoes e nao de instalacoes. O diretorio origem será sempre um diretorio com o nome de HASH.
Exemplo: Origem:D:\cda162709d239766830bae5ce12b\HotFixSQL\Files\sqlrun_sql.msp , Destino: C:\WINDOWS\Installer\145258.msp

O procedimento abaixo ajuda a localizar o arquivo pendente nas chaves de registro e a como criar essas entradas a partir da identificacao do pacote pendente e da extracao desses arquivos (o procedimento recria o diretorio e nao altera a chave de registro):

To find more details about the missing .msp file in the Windows Installer cache, follow these steps:
Search for the missing .msp file in the following Windows Installer Patches registry subkey:
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Patches\
Find the Patch GUID.
Search for the Patch GUID in the following Windows Installer Products registry subkey:
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\
For the sample setup log, the information about the missing .msp file and its corresponding patch details are present in the following registry entries:

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Patches\A3B085EA74A9A7640A496636F7EF9A44

Value: 0 
Name: LocalPackage 
Data: C:\WINDOWS\Installer\145258.msp 

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Products\1EB3A031CC585314E87AA527E46EECC2\Patches\A3B085EA74A9A7640A496636F7EF9A44
Value: 6 
Name: DisplayName 
Data: GDR 2050 for SQL Server Database Services 2005 ENU (KB932555)

Now you have all the information points to start the steps to resolve the missing files in the Windows Installer cache.
------------------------------------------------------------------------------------------------------------------------
A execucao de Service Pack 3 sob Windows Server 2008R2, nao foram localizados arquivos pendentes no CACHE de instalação, mas durante a atualizaçao foram solicitados arquivos .msi (rsfx.msi) foi identificado esse arquivo no REGISTRY do Windows, a chave InstallSource continha uma referencia para o diretorio D:\0ca91e857a4f12dd390f0821a3, após alteracao dessa chave inserindo o path original de instalaçao referenciando o pacote rsfx.msi não houve problemas para dar continuidade com a atualizacao.

No caso descrito acima, foi encontrado uma exceção na regra do arquivo FindSQLInstalls.vbs, o retorno da execução não informou a necessidade do arquivo, o que abre precedente para que a validacao pré execuçao de service pack/hotfix também deve levar em consideraçao a acessibilidade dos paths registrados nas chaves de registro do windows ( nome da chave:InstallSource, caminho da chave:HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Installer\UserData\S-1-5-18\Patches\ )

Para os casos onde os arquivos onde o SOURCE LOCATION informado no relatorio do FindSQLInstalls.vbs não são arquivos com o nome em formato de HASH o procedimento de COPIA e RENAME dos arquivos para o CACHE foram positivos.

DEEP DIVE:
PARA MSP

ENTENDER QUAIS SAO OS GUIDS CORRESPONDENTES A QUAIS VERSOES DE ATUALIZACOES COMO IDENTIFICAR OS SOURCES PATHS E TESTA-LOS CASO EXISTAM.

PARA MSI
ENTENDER OS REGISTROS QUE CARREGAM OS SOURCES PATHS DA INSTALACAO E SE O USUARIO QUE ESTA EXECUTANDO POSSUI PERMISSAO PARA ACESSA-LOS (E COM QUAL CONTA É ACESSADO, COM A CONTA DE SERVICO OU COM A CONTA DE EXECUCAO)

Para a solucao de Infra:
Proteger diretório C:\Windows\Installer\ e criar estrutura restrita e organizada de diretorios de instalacao e atualizacao.

RESOLUCAO:
-------------------------------------------------------------------------------------------------------------------
Procedure 1: Use a script (FindSQLInstalls.vbs) --link acima.

Note !!!!! -> The FindSQLInstalls.vbs script collects the information to correct invalid package paths.<- !!!!! (Paths Invalidos são armazenados no REGISTRY do Windows? Quais entradas? Quais chaves armazenam essas informacoes?) 

!!!! -> And, this script is used against the source locations to make sure that all MSP packages are in the Windows Installer cache directory. <- !!!!! (O script é usado contra a localização dos pacotes MSP de que forma, onde esses sources locations são armazenados? São feitos testes se os diretórios originais desses arquivos EXISTEM? Caso existam, esses arquivos são copiados? Se os arquivos/diretórios originais, sabendo que esses diretorios são criados ao executarem algum service pack, hotfix e sao extraidos no diretorio local e sao criados com nomes usando algum HASH, existe algum teste de existencia desses diretorios?? 

!!Any missing packages will be re-added if the original source media is available.
Nesse caso se a media original partir de um diretorio compartilhado, por exemplo, um share de aplicativos para instalação, o usuário que executa esse VBS deve ter acesso ao diretorio de origem.
Como posso localizar o diretório ORIGEM da instalaçao da Instancia e das Features?


# Windows

-> iSCSI Target 2012 R2
https://social.technet.microsoft.com/wiki/contents/articles/12370.windows-server-2012-set-up-your-first-domain-controller-step-by-step.aspx

http://www.windowsnetworking.com/articles-tutorials/windows-server-2012/failover-clustering-windows-server-2012-r2-part1.html

https://blogs.technet.microsoft.com/amitd/2014/06/17/configure-windows-2012-r2-as-iscsi-target/

http://www.windowsnetworking.com/articles-tutorials/windows-server-2012/configuring-iscsi-storage-part2.html

-> Win Storage 2008 R2
https://blogs.technet.microsoft.com/amitd/2014/06/17/configure-windows-2012-r2-as-iscsi-target/
http://blog.uraniumbackup.us/how-to-iscsi-target-on-win-server-2008-r2/2/

-passo-a-passo
http://blog.uraniumbackup.us/how-to-iscsi-target-on-win-server-2008-r2/2/
-Host Target.

https://blogs.technet.microsoft.com/josebda/2007/12/18/configuring-the-microsoft-iscsi-software-target/

-> Lista de ERROS conhecidos para instalação do Windows Server 2008R2 <-
https://support.microsoft.com/pt-br/kb/955725

--> “RequireKerberos” Error During the SQL 2008 Installation on Windows 2008 Cluster
https://winadminnotes.wordpress.com/2010/10/28/requirekerberos-error-during-the-sql-2008-installation-on-windows-2008-cluster/

TROUBLESHOOTING:
 1 - Baixar SP1 :https://www.microsoft.com/en-us/download/confirmation.aspx?id=20302
 2 - Realizar "extract" do SP para um diretorio no server: <cmd> <diretorio> SQLServer2008SP1-KB968369-x64-ENU.exe /x:C:\<diretorio extract>
 3 - Executar o Suport Files: C:\SP1\x64\setup\1033\sqlsupport.msi
 
 4 - Executar via CMD: d:\>Setup.exe /PCUSource=C:\!BIN\SP\SP1\Extract
 https://winadminnotes.wordpress.com/2010/10/28/requirekerberos-error-during-the-sql-2008-installation-on-windows-2008-cluster/
 

 
