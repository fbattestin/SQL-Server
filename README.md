# SQL-Server

White Paper Data Compression - SQL Server 2008r2
https://technet.microsoft.com/en-us/library/dd894051(v=sql.100).aspx

SQL Server Utilities - Windows PS
https://github.com/DevNambi/SqlServerUtilities/tree/master/Save-SqlHistory

# Microsoft SQL Server BPA ( Best Pretices Analyzer ) 
https://www.microsoft.com/en-us/download/details.aspx?id=15289&751be11f-ede8-5a0c-058c-2ee190a24fa6=True
https://www.microsoft.com/en-us/download/details.aspx?id=29302
É necessário o Microsoft Baseline Configuration Analyzer 2.0
https://www.microsoft.com/en-us/download/details.aspx?id=16475


# Buffer Pool
http://www.sqlskills.com/blogs/paul/performance-issues-from-wasted-buffer-pool-memory/


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

-> Add Feature AD

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

