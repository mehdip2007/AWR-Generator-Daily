# AWR-Generator-Daily
the following shell script is used to generate Oracle AWR for daily needs and email it 

#!/bin/bash
login_str=scott/tiger@ORCL
sqlplus -s ${login_str} << EOF

#-- setup columns for snapshots

	column bsnap1 new_value bsnap1s noprint;
	column esnap1 new_value esnap1s noprint;

#-- get snap id for daily 9 am to 9 pm central
#-- get snap id for 9 am daily

  select snap_id bsnap1,END_INTERVAL_TIME
	from dba_hist_snapshot
	where 
	extract(year from END_INTERVAL_TIME)=extract(year from (sysdate-1-(case trim(to_char(sysdate,'DAY')) WHEN 'TUESDAY' then 2 ELSE 0 end))) and
	extract(month from END_INTERVAL_TIME)=extract(month from (sysdate-1-(case trim(to_char(sysdate,'DAY')) WHEN 'TUESDAY' then 2 ELSE 0 end))) and
	extract(day from END_INTERVAL_TIME)=extract(day from (sysdate-1-(case trim(to_char(sysdate,'DAY')) WHEN 'TUESDAY' then 2 ELSE 0 end))) and
	extract(hour from END_INTERVAL_TIME)= 09;

#-- get snap id for 9 pm daily
	select snap_id esnap1,END_INTERVAL_TIME
	from dba_hist_snapshot
	where 
	extract(year from END_INTERVAL_TIME)=extract(year from (sysdate-1-(case trim(to_char(sysdate,'DAY')) WHEN 'TUESDAY' then 2 ELSE 0 end))) and
	extract(month from END_INTERVAL_TIME)=extract(month from (sysdate-1-(case trim(to_char(sysdate,'DAY')) WHEN 'TUESDAY' then 2 ELSE 0 end))) and
	extract(day from END_INTERVAL_TIME)=extract(day from (sysdate-1-(case trim(to_char(sysdate,'DAY')) WHEN 'TUESDAY' then 2 ELSE 0 end))) and
	extract(hour from END_INTERVAL_TIME)= 21;

#-- awr report for daily
	define report_type='html';
	define begin_snap = &bsnap1s;
	define end_snap   = &esnap1s;
	define report_name = 'daily.html';

	define num_days = 0;

	@@$ORACLE_HOME/rdbms/admin/awrrpt.sql

	undefine report_type
	undefine report_name
	undefine begin_snap
	undefine end_snap
	undefine num_days
    exit;
EOF

#--Send Email   via
  echo "AWR-Daily" |mailx -s "Daily AWR Report " -a daily.html youremail@yourhost.com
