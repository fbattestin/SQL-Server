----  See: https://social.msdn.microsoft.com/Forums/sqlserver/en-US/360b064b-2cc7-4da3-a650-3741b2208785/memory-issues-cause-shutdown-of-instance?forum=sqldatabaseengine
----  Note: those are very sloppy, poorly written/readable queries

---- Re-Written from above link and readable!
;WITH CachedPlans AS(
	SELECT p.cacheobjtype cacheobjtype
		, p.objtype objtype
		, p.usecounts usecounts
		, size_in_bytes size_in_bytes
		, s.dbid dbid
		, s.objectid objectid
	, CONVERT (nvarchar(100), REPLACE(REPLACE(CASE
		WHEN s.[text] IS NULL THEN NULL
		WHEN CHARINDEX ('noexec', SUBSTRING (s.[text], 1, 200)) > 0 THEN SUBSTRING (s.[text], 1, 40)
		WHEN (CHARINDEX ('sp_executesql', SUBSTRING (s.[text], 1, 200)) > 0) THEN SUBSTRING (s.[text], CHARINDEX ('exec', SUBSTRING (s.[text], 1, 200)), 60)
		WHEN CHARINDEX ('exec ', SUBSTRING (s.[text], 1, 200)) > 0 THEN SUBSTRING (s.[text], CHARINDEX ('exec', SUBSTRING (s.[text], 1, 4000)),
			CHARINDEX (' ', SUBSTRING (SUBSTRING (s.[text], 1, 200) + ' ', CHARINDEX ('exec', SUBSTRING (s.[text], 1, 500)), 200), 9) )
		WHEN SUBSTRING (s.[text], 1, 2) IN ('sp', 'xp', 'usp') THEN SUBSTRING (s.[text], 1, CHARINDEX (' ', SUBSTRING (s.[text], 1, 200) + ' '))
		WHEN SUBSTRING (s.[text], 1, 30) LIKE '%UPDATE %' OR SUBSTRING (s.[text], 1, 30) LIKE '%INSERT %' OR SUBSTRING (s.[text], 1, 30) LIKE '%DELETE %'
		THEN SUBSTRING (s.[text], 1, 30)
		ELSE SUBSTRING (s.[text], 1, 45)
	END
	, CHAR (10), ' '), CHAR (13), ' ')) AS short_qry_text
	FROM sys.dm_exec_cached_plans p
		CROSS APPLY sys.dm_exec_sql_text (p.plan_handle) s
)
SELECT COUNT(*) AS PlanCount
	, SUM(size_in_bytes)/(1024*1024) TotalSizeInMB
	, cacheobjtype CachedObjectType
	, objtype ObjectType
	--, usecounts
	, DB_NAME(dbid) DatabaseName
	, objectid ObjectID
	, short_qry_text ShortQueryText
FROM CachedPlans
GROUP BY cacheobjtype, objtype, usecounts, dbid, objectid, short_qry_text
HAVING COUNT(*) > 100
ORDER BY COUNT(*) DESC
