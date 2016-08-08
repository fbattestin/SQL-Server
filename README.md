# SQL-Server

White Paper Data Compression - SQL Server 2008r2
https://technet.microsoft.com/en-us/library/dd894051(v=sql.100).aspx

SQL Server Utilities - Windows PS
https://github.com/DevNambi/SqlServerUtilities/tree/master/Save-SqlHistory

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

When you install SQL Server, the Windows Installer stores critical files in the Windows Installer Cache (default is C:\Windows\Installer). These files are required for uninstalling and updating applications. Missing files cannot be copied between computers, because they are unique.


RECOMENDACAO MICROSOFT:  
-------------------------------------------------------------------------------------------------------------------
You should run the repair from the original installation media, using the following command line:

setup.exe /ACTION=REPAIR /INDICATEPROGRESS=TRUE

Repair the common shared components and features first, and then repeat the command to repair the instances installed. During the repair process, the setup dialog box disappears. As long as the progress window does not show an error, the repair process is proceeding as expected.

1#  Repair shared components and features.
2#  Repair Instances.

Realizar Repair da instancia, utilizando o arquivo original de instalacao. O problema é que para que o REPAIR seja realizado, esses arquivos do CACHE tambem sao necessarios. O que foi observado é que ao executar o REPAIR de uma instancia onde nenhum arquivo indicado como MISSING FILES foram localizados (foi executado o VBS FindSQLInstalls), ao realizar o REPAIR uma referencia de arquivo *.msi foi solicitada, no dialogBox o endereco original do arquivo apontava para um diretorio criptografado, como os diretorios criados pelos Hotfix ao serem executados (realizam o unzip do pacote de atualizacao no diretorio em que sao executados, e geram nomes randomicos com apararencia de criptografia). 

CAUSA:
-------------------------------------------------------------------------------------------------------------------
These problems may occur when the Windows Installer database file (.msi) or the Windows Installer patch file (.msp) is missing from the Windows Installer cache. The Windows Installer cache is located in the following folder:
%windir%\installer
When a product is installed by using Windows Installer, a stripped version of the original .msi file is stored in the Windows Installer cache. 

!!!!!!!! -> Every update to the product such as a hotfix, a cumulative update, or a service pack setup, also stores the relevant .msp or .msi file in the Windows Installer cache. <- !!!!!!!

Any future update to the product such as a hotfix, a cumulative update, or a service pack setup, !!!! relies !!!! on the information in the files that are stored in the Windows Installer cache. Without this information, the new update cannot perform the required transformations. --> Nesse caso, a referencia para esses arquivos estão escritos no registro do Windows.

