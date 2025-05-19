 select count(*) - (select count(*) from LFAR_REPORT_MASTER) from LFAR_REPORT_MASTER lrml left join 
LFAR_REPORTS_MASTER_LIST lrm on lrm.REPORT_MASTER_ID = lrml.REPORT_ID
where BRANCH_CODE = '72080'
and QUARTER_DATE = '31/03/2025';

Error
ORA-00937: not a single-group group function
00937. 00000 -  "not a single-group group function"
*Cause:    
*Action:
Error at Line: 64 Column: 27
