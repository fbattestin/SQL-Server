# SQL-Server

White Paper Data Compression - SQL Server 2008r2
https://technet.microsoft.com/en-us/library/dd894051(v=sql.100).aspx

SQL Server Utilities - Windows PS
https://github.com/DevNambi/SqlServerUtilities/tree/master/Save-SqlHistory

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
