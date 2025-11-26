> **Note** — The folder `linguist-samples/` contains tiny real files so GitHub can correctly display all languages used in this repo.  
> The actual content and examples remain in this README.

![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Quick SQL Logic For Backups And Restores
**Post Date: December 12, 2016**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some quick SQL Logic to create some backups, and do some restores. Nothing new, or crazy here. Just something quick and easy for people to throw into their automation logic. Sure; these are not hard to find, but it's helpful none the less to have it out there.</p> 

![Quick SQL Backup Logic]( https://mikesdatawork.files.wordpress.com/2016/12/image001.png "Quick SQL Backups")
 
     
## SQL-Logic
```SQL
use master;
set nocount on
 
/********************************/
/********Post Date: ABOUT BACKUPS********/
/********************************/
 
-- backup single database (overwrite backup file if exists (format), verify backup (checksum)
backup database [MyDatabase] to disk = 'E:\SQLBACKUPS\MyDatabase.bak' with format, checksum
 
-- backup all databases  (overwrite backup file if exists (format), verify backup (checksum), the question mark(?) denotes @MyDatabaseName automatically.  this script does not require any changes.  just run.
exec master..sp_msforeachdb
'if (''?'') not in (''tempdb'')
    begin
        backup database [?] to disk = ''E:\SQLBACKUPS\?.bak'' with format, checksum;
    end'
 
-- verify backup date and location (occurred today)
select
    'database'      = upper(bs.database_name)
,   'backed up on'  = left(bs.backup_finish_date, 19)
,   'file location' = upper(bmf.physical_device_name)
from
    msdb..backupset bs join msdb..backupmediafamily bmf on bs.media_set_id = bmf.media_set_id
where
    -- check backups that occurred today
    bs.backup_finish_date > (select dateadd(day, datediff(day, 0, getdate()), 0))
    -- get full database backup history only
    and bs.type = 'D'
order by
    bs.backup_finish_date desc
 
 
/********************************/
-- Post Date: ABOUT RESTORES
/********************************/
 
-- close open sessions against a database prior to restore.
declare       @close_open_sessions varchar(max)
set           @close_open_sessions = ''
select        @close_open_sessions = @close_open_sessions + 
'kill ' + cast(spid as varchar) + ';' + char(10)
from          sysprocesses where db_name(dbid) = 'MyDatabase'
exec          (@close_open_sessions)
  
-- restore database.  get logical and physical names from backup file with 'restore filelistonly from disk = 'E:\SQLBACKUPS\MyDatabase.bak' or from live database before the restore with 'select name, filename from [MyDatabase]..sysfiles'
restore database [MyDatabaseTest] from disk = 'E:\SQLBACKUPS\MyDatabase.bak'
with
       replace
,      recovery
,      move 'MyDatabase_Data' to 'E:\Program Files\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\DATA\MyDatabase_Data_01.MDF'
,      move 'MyDatabase_Log'  to 'E:\Program Files\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQL\DATA\MyDatabase_Log_01.MDF'
,      stats = 25
go
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

 
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")
