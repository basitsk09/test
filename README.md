package com.crs.commonReportsService.repositories;

import com.crs.commonReportsService.models.Rw01;
import jakarta.transaction.Transactional;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.*;

public interface Rw01Repository  extends JpaRepository<Rw01,String> {

    @Transactional
    @Query(value = "select prm.crs_sys_param_name,prm.crs_sys_param_desc ,sum(nvl(sy.YSA_CHECK_AMT,0)) as YSA_CHECK_AMT from crs_sys_param prm left OUTER JOIN  \n" +
            "                  CRS_YSA_CHECK sy on(prm.crs_sys_param_id=sy.ysa_check_code and sy.YSA_CHECK_DATE=:quarterEndDate and sy.YSA_CHECK_BRANCH =:branch_code) " +
            "where prm.crs_sys_param_type='YSA_CHECK' and prm.CRS_SYS_PARAM_ID!='40211'\n" +
            "                 group by prm.crs_sys_param_name,prm.crs_sys_param_desc ORDER BY 1",nativeQuery = true)
 List<Map<String, Object>> getPreRequisteHeaderAmount(@Param("quarterEndDate")Date quarterEndDate,@Param("branch_code")String branch_code);
    //List<String> getPreRequisteHeaderAmount(@Param("quarterEndDate")Date quarterEndDate,@Param("branch_code")String branch_code);

  /*  @Query(value="select ax.PL_SUPL_ID,ax.PL_SUPL_CY  from CRS_PL_SUPL ax where ax.PL_SUPL_DATE=:quarterEndDate and ax.PL_SUPL_BRANCH=:branch_code  ",nativeQuery = true)
    List<Map<String, Object>>  findByPlSuplBranchAndPlSuplDate( @Param("quarterEndDate")Date quarterEndDate,@Param("branch_code")String branch_code);*/

    @Query(value="select ax.PL_SUPL_ID, nvl(ax.PL_SUPL_CY ,0) as PL_SUPL_CY from CRS_PL_SUPL ax where ax.PL_SUPL_DATE=:quarterEndDate and ax.PL_SUPL_BRANCH=:branch_code  ",nativeQuery = true)
    List<String> findByPlSuplBranchAndPlSuplDate( @Param("quarterEndDate")Date quarterEndDate,@Param("branch_code")String branch_code);



    @Query(value="select ax.PL_SUPL_ID, nvl(ax.PL_SUPL_CY ,0) as PL_SUPL_CY from CRS_PL_SUPL ax where ax.PL_SUPL_DATE=:quarterEndDate and ax.PL_SUPL_BRANCH=:branch_code  ",nativeQuery = true)
    List<Map<String, Object>> findByPlSuplBranchAndPlSuplDate_qqq( @Param("quarterEndDate")Date quarterEndDate,@Param("branch_code")String branch_code);

   Rw01 findByPlSuplDateAndPlSuplBranchAndPlSuplId (Date quarterEndDate,String branch_code,String plSuplId);

    @Transactional

    @Query(nativeQuery = true,value = "SELECT \n" +
            "TAB_ADD_ROW_FLAG ," +
            "TAB_FOOTER_FLAG ," +
            "TAB_FOOTER_VALUE ," +
            "TAB_NAME ," +
            "TAB_VALUE_FK AS TAB_VALUE  "+
            "FROM COMMON_TAB_PARAM WHERE REPORT_ID_FK =:reportId order by TAB_VALUE_FK")
    List<Map<String,Object>> getTabData(@Param("reportId") String reportId);

   /* @Transactional
    @Query(value="select column_id,column_name,column_type,column_disabled,column_nodisplay," +
            "total_column_disabled, heading_values, column_maxlength,select_values,tab_value " +
            "from common_screen_param where report_id =:reportId and tab_value=:TAB_VALUE ",nativeQuery = true)
    List<Map<String, Object>> findColumnData(@Param("reportId")String reportId, @Param("TAB_VALUE") String tabvalue);*/


    @Transactional
    @Query(value="select   COLUMN_ID , " +

            " case when  COLUMN_NAME='PREVIOUS THREE MONTHS ENDED'  OR   COLUMN_NAME='CURRENT THREE MONTHS ENDED' " +
            "      then COLUMN_NAME||:appendValue " +
            "      else COLUMN_NAME " +
            " end as COLUMN_NAME , " +

            " case when '06'=:quarter and COLUMN_ID='4' " +
            "      then REPLACE(\n" +
            "          (case \n" +
            "                               when '06'=:quarter and COLUMN_ID='4' then REPLACE(HEADING_VALUES,'-NEXTYEAR', :quarteryear+1)\n" +
            "                              else\n" +
            "                               HEADING_VALUES end ),'-YEAR', :quarteryear)  else  HEADING_VALUES end \n" +
            "                               as HEADING_VALUES , " +
            "                   COLUMN_MAXLENGTH ,COLUMN_TYPE,COLUMN_DISABLED,COLUMN_NODISPLAY,TOTAL_COLUMN_DISABLED " +
            "                   FROM COMMON_SCREEN_PARAM WHERE REPORT_ID= :reportId and TAB_VALUE=:TAB_VALUE",nativeQuery = true)
    List<Map<String, Object>> findColumnData_OLD(@Param("appendValue")String appendValue,@Param("quarter")String quarter,@Param("quarteryear")String quarteryear,
                                             @Param("reportId")String reportId, @Param("TAB_VALUE") String tabvalue);


 @Transactional
 @Query(value="select   COLUMN_ID , " +
         "                  case when  COLUMN_NAME='PREVIOUS THREE MONTHS ENDED'  " +
         "                  then 'PREVIOUS ' || :quarterMonth || ' ENDED' || ' ' || :appendValue  " +
         "                  when COLUMN_NAME='CURRENT THREE MONTHS ENDED' " +
         "                  then 'PREVIOUS ' || :quarterMonth || ' ENDED' || ' ' || :quarterEndDate || ' RS.  Ps. ' else COLUMN_NAME " +
         "                  end as COLUMN_NAME , "  +

         "            REPLACE(" +
         "                    REPLACE(" +
         "                           REPLACE(" +
         "                                  REPLACE(" +
         "                                          REPLACE(HEADING_VALUES,'NEXTYEAR', case when 3 = :quarter then TO_CHAR(:quarteryear) else TO_CHAR(:quarteryear+1) end), " +
         "                                  'YEAR', case when 3 = :quarter then TO_CHAR(:quarteryear-1) else TO_CHAR(:quarteryear) end), " +
         "                           'MONTH_Y', :quaterEndMonthYear), " +
         "                    'QED_STRING_FORMAT', :qedStringFormat), " +
         "            'PREVIOUS_FINANCIAL_END', :previousFinancialEnd) " +
         "           as HEADING_VALUES, " +

         "           COLUMN_MAXLENGTH ,COLUMN_TYPE,COLUMN_DISABLED,COLUMN_NODISPLAY,TOTAL_COLUMN_DISABLED " +
         "           FROM COMMON_SCREEN_PARAM WHERE REPORT_ID= :reportId and TAB_VALUE=:TAB_VALUE",nativeQuery = true)
 List<Map<String, Object>> findColumnData(
         @Param("appendValue")String appendValue,
         @Param("quaterEndMonthYear")String quaterEndMonthYear,
         @Param("quarter")String quarter,
         @Param("quarteryear")String quarteryear,
         @Param("reportId")String reportId,
         @Param("TAB_VALUE") String tabvalue,
         @Param("quarterEndDate")String quarterEndDate,
         @Param("quarterMonth")String quarterMonth,
         @Param("qedStringFormat")String qedStringFormat,
         @Param("previousFinancialEnd")String previousFinancialEnd
 );

    @Transactional
    @Query(value="select   COLUMN_ID , " +
            "                  case when  COLUMN_NAME='PREVIOUS THREE MONTHS ENDED'  " +
            "                  then 'PREVIOUS ' || :quarterMonth || ' ENDED' || ' ' || :appendValue  " +
            "                  when COLUMN_NAME='CURRENT THREE MONTHS ENDED' " +
            "                  then 'PREVIOUS ' || :quarterMonth || ' ENDED' || ' ' || :quarterEndDate || ' RS.  Ps. ' else COLUMN_NAME " +
            "                  end as COLUMN_NAME , "  +
            "  REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(HEADING_VALUES,'NEXTYEAR', :quarteryear+1), 'YEAR', :quarteryear), 'MONTH_Y', :quaterEndMonthYear), 'QED_STRING_FORMAT', :qedStringFormat), 'PREVIOUS_FINANCIAL_END', :previousFinancialEnd) as HEADING_VALUES, " +
            "                   COLUMN_MAXLENGTH ,COLUMN_TYPE,COLUMN_DISABLED,COLUMN_NODISPLAY,TOTAL_COLUMN_DISABLED " +
            "                   FROM COMMON_SCREEN_PARAM WHERE REPORT_ID= :reportId and TAB_VALUE=:TAB_VALUE",nativeQuery = true)
    List<Map<String, Object>> findColumnData_old_06_03_2025(
            @Param("appendValue")String appendValue,
            @Param("quaterEndMonthYear")String quaterEndMonthYear,
            @Param("quarter")String quarter,
            @Param("quarteryear")String quarteryear,
            @Param("reportId")String reportId,
            @Param("TAB_VALUE") String tabvalue,
            @Param("quarterEndDate")String quarterEndDate,
            @Param("quarterMonth")String quarterMonth,
            @Param("qedStringFormat")String qedStringFormat,
            @Param("previousFinancialEnd")String previousFinancialEnd
    );

    @Transactional
    @Query(value=" select " +
            " CASE WHEN PL_SUPL_ID in('19','35') then 'noDisplay' WHEN PL_SUPL_ID in('20') then 'Outstanding in Rs.      Ps._heading'  ELSE nvl(PL_SUPL_PY,0)|| '_D' END AS PL_SUPL_PY," +
            //"nvl(PL_SUPL_PY,0)|| '_D' AS PL_SUPL_PY," +
            "' ',' ',' ',\n" +
            "          case when '06'=:quarter and PL_SUPL_ID in( '24','25','26','27','28','29','30','31','32') then \n" +
            "          nvl(PL_SUPL_CY,0)||'_D' \n" +
            "          when '09'=:quarter and PL_SUPL_ID in( '27','28','29','30','31','32') then \n" +
            "          nvl(PL_SUPL_CY,0)||'_D' \n" +
            "          when '12'=:quarter and PL_SUPL_ID in( '30','31','32') then \n" +
            "          nvl(PL_SUPL_CY,0)||'_D' \n" +
            "          else  CASE WHEN PL_SUPL_ID in('19','35') then 'noDisplay' WHEN PL_SUPL_ID in('20') then 'Outstanding in Rs.      Ps._heading' ELSE nvl(PL_SUPL_CY,0) END \n" +
            "          end as PL_SUPL_CY ,(PL_SUPL_ID- '14'),PL_SUPL_ID,'false' \n" +
            "            from CRS_PL_SUPL  where   REPORT_MASTER_LIST_ID_FK=:submission_id order by  to_number(PL_SUPL_ID)",nativeQuery = true)
    LinkedList<List<String>> getrowdatalistOLD(@Param("quarter")String quarter,@Param("submission_id") String submission_id);


    @Transactional
    @Query(value=" select " +
            " CASE WHEN PL_SUPL_ID in('19') then 'noDisplay' WHEN PL_SUPL_ID in('22') then 'Outstanding in Rs.      Ps._heading'  ELSE nvl(PL_SUPL_PY,0)|| '_D' END AS PL_SUPL_PY," +
            //"nvl(PL_SUPL_PY,0)|| '_D' AS PL_SUPL_PY," +
            "' ',' ',' ',\n" +
            "          case when '06'=:quarter and PL_SUPL_ID in( '15', '16', '17', '18', '21',  '23', '24','25','26','27','28','29','30','31','32', '33', '34') then \n" +
            "          nvl(PL_SUPL_CY,0)||'_D' \n" +
            "          when '09'=:quarter and PL_SUPL_ID in( '29','30','31','32','33','34') then \n" +
            "          nvl(PL_SUPL_CY,0)||'_D' \n" +
            "          when '12'=:quarter and PL_SUPL_ID in( '30','31','32') then \n" +
            "          nvl(PL_SUPL_CY,0)||'_D' \n" +
            "          else  CASE WHEN PL_SUPL_ID in('19') then 'noDisplay' WHEN PL_SUPL_ID in('22') then 'Outstanding in Rs.      Ps._heading' ELSE nvl(PL_SUPL_CY,0) END \n" +
            "          end as PL_SUPL_CY ,(PL_SUPL_ID- '14'),PL_SUPL_ID,'false' \n" +
            "            from CRS_PL_SUPL  where   REPORT_MASTER_LIST_ID_FK=:submission_id order by  to_number(PL_SUPL_ID)",nativeQuery = true)
    LinkedList<List<String>> getrowdatalist(@Param("quarter")String quarter,@Param("submission_id") String submission_id);


    /////////////Falguni///////////////
    @Transactional
    @Query(value=" select " +
            " CASE WHEN PL_SUPL_ID in('19') then 'noDisplay' WHEN PL_SUPL_ID in('22') then 'Outstanding in Rs.      Ps._heading'  ELSE nvl(PL_SUPL_CY,0)|| '_D' END AS PL_SUPL_PY," +
            "' ',' ',' ',\n" +
            "          case when '06'=:quarter and PL_SUPL_ID in(  '15', '16', '17', '18', '21',  '23', '24','25','26','27','28','29','30','31','32', '33', '34') then \n" + // '24','25','26','27','28','29','30','31','32'
            "          '0_D'  \n" +
            "          when '09'=:quarter and PL_SUPL_ID in( '29','30','31','32','33','34') then \n" +
            "          '0.00'  \n" +
            "          when '12'=:quarter and PL_SUPL_ID in( '30','31','32') then \n" +
            "          '0.00'  \n" +
            "          else  CASE WHEN PL_SUPL_ID in('19') then 'noDisplay' WHEN PL_SUPL_ID in('22') then 'Outstanding in Rs.      Ps._heading' ELSE  '0.00' END \n" +
            "          end as PL_SUPL_CY ," +
            "   (PL_SUPL_ID- '14'), PL_SUPL_ID, 'false' \n" +
            "            from CRS_PL_SUPL  where  PL_SUPL_DATE=:PL_SUPL_DATE AND PL_SUPL_BRANCH=:PL_SUPL_BRANCH order by  to_number(PL_SUPL_ID)",nativeQuery = true)
    LinkedList<List<String>> getrowdatalistOnIndex(
            @Param("quarter")String quarter,
            @Param("PL_SUPL_DATE") Date PL_SUPL_DATE,
            @Param("PL_SUPL_BRANCH") String PL_SUPL_BRANCH
    );
    /////////////////////////////////
    ///
    ///
    @Transactional
    @Query(value=" select nvl(PL_SUPL_CY,0)|| '_D' AS PL_SUPL_PY,' ',' ',' ',\n" +
            "          case when '06'=:quarter and PL_SUPL_ID in( '24','25','26','27','28','29','30','31','32') then \n" +
            "          '0.00'  \n" +
            "          when '09'=:quarter and PL_SUPL_ID in( '27','28','29','30','31','32') then \n" +
            "          '0.00'  \n" +
            "          when '12'=:quarter and PL_SUPL_ID in( '30','31','32') then \n" +
            "          '0.00'  \n" +
            "          else  '0.00'  \n" +
            "          end as PL_SUPL_CY ," +
            "   (PL_SUPL_ID- '14'), PL_SUPL_ID, 'false' \n" +
            "            from CRS_PL_SUPL  where  PL_SUPL_DATE=:PL_SUPL_DATE AND PL_SUPL_BRANCH=:PL_SUPL_BRANCH order by  to_number(PL_SUPL_ID)",nativeQuery = true)
    LinkedList<List<String>> getrowdatalistOnIndexOLD_13March2025(
            @Param("quarter")String quarter,
            @Param("PL_SUPL_DATE") Date PL_SUPL_DATE,
            @Param("PL_SUPL_BRANCH") String PL_SUPL_BRANCH
    );

    /*@Transactional
    @Query(value=" select '','', YA_SUPL1_CY,'',YA_SUPL1_ID,'false' from CRS_YA_SUPL_1  where REPORT_MASTER_LIST_ID_FK=:submission_id order by  to_number(YA_SUPL1_ID) "
            ,nativeQuery = true)
    LinkedList<List<String>> getrowdatalisttab2(@Param("submission_id") String submission_id);*/

    @Transactional
    @Query(value=" select ' ',' ',  YA_SUPL2_LOC, YA_SUPL2_GUARANTEE, YA_SUPL2_ACCPT, YA_SUPL2_LOU,YA_SUPL2_LOU,YA_SUPL2_ID ,'false' " +
            "from CRS_YA_SUPL_2 where REPORT_MASTER_LIST_ID_FK=:submission_id order by YA_SUPL2_ID ",nativeQuery = true)
    LinkedList<List<String>> getrowdatalisttab3(@Param("submission_id") String submission_id);

    @Transactional
    @Query(value="  select  ' ',' ',YA_SUPL3_CY||'_D', ' ',YA_SUPL3_ID ,'false' " +
            "from CRS_YA_SUPL_3 where REPORT_MASTER_LIST_ID_FK='submission_id' order by YA_SUPL3_ID ",nativeQuery = true)
    LinkedList<List<String>> getrowdatalisttab4(@Param("submission_id") String submission_id);

    @Transactional
    @Query(value="  select ' ',' ', YA_SUPL4_CLAIMS, YA_SUPL4_AMT,' ', YA_SUPL4_ID ,'false' " +
            "from CRS_YA_SUPL_4 where  REPORT_MASTER_LIST_ID_FK =:submission_id order by  to_number(YA_SUPL4_ID) ",nativeQuery = true)
    LinkedList<List<String>> getrowdatalisttab5(@Param("submission_id") String submission_id);

    @Transactional
    @Query(value="  select ' ',' ', YA_SUPL4_CLAIMS, YA_SUPL4_AMT,' ', YA_SUPL4_ID ,'false' " +
            "from CRS_YA_SUPL_4 where  REPORT_MASTER_LIST_ID_FK =:submission_id order by  to_number(YA_SUPL4_ID) ",nativeQuery = true)
    LinkedList<List<String>> getrowdatalisttab6(@Param("submission_id") String submission_id);

    @Transactional
    @Query(value = "select SUBMISSION_ID from REPORT_SUBMISSION  where  BRANCH_CODE=:branch_code and REPORT_DATE=:quarterEndDate",nativeQuery = true)
    String countForRmlPrevQtr(@Param("branch_code") String branch_code,@Param("quarterEndDate") Date quarterEndDate);


}

