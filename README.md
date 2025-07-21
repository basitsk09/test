 public List<FRTAuditStatusReq> getRequests(String quarter_date) {


            String query1 = "select cas.as_rt_id , cas.as_id, cas.as_branch, br_name branch_name,crs_auditable old_br_status," +
                    "SUBSTR(bm.region_code,1,3) ||''|| SUBSTR(bm.region_code,4,3) ||''|| SUBSTR(bm.region_code,7,3) RO,bm.circle_code, " +
                    "nvl((select ifcofr_audit_flag from crs_ifcofr_audit where ifcofr_branch=cas.as_branch and ifcofr_date=to_date(?,'dd/mm/yyyy') " +
                    "and crt.rt_id=cas.as_rt_id),'N')ifcofr_flag, " +
                    "cas.as_new_status, cas.as_req_status, crt.rt_status,crt.rt_date " +
                    "from crs_request_track crt, crs_audit_status cas, branch_master bm " +
                    "where cas.as_rt_id = crt.rt_id and cas.as_branch=bm.branchno and cas.as_req_status not in ('R','A') order by cas.as_rt_id,cas.as_id";

            List<FRTAuditStatusReq> list = jdbcTemplateObject.query(query1, new Object[]{quarter_date}, new ResultSetExtractor<List<FRTAuditStatusReq>>() {

                @Override
                public List<FRTAuditStatusReq> extractData(ResultSet rs1) throws SQLException, DataAccessException {
                    List<FRTAuditStatusReq> list = new ArrayList<>();
                    while (rs1.next()) {
                        FRTAuditStatusReq report = new FRTAuditStatusReq();
                        report.setAs_rt_id(rs1.getString("AS_RT_ID"));
                        report.setAs_id(rs1.getString("AS_ID"));
                        report.setBranchCode(rs1.getString("AS_BRANCH"));
                        report.setBranchName(rs1.getString("branch_name"));
                        String oldSts =rs1.getString("old_br_status");
                        report.setRoCode(rs1.getString("RO"));
                        report.setCircle_code(rs1.getString("circle_code"));
                        report.setIfcofr_flag(rs1.getString("ifcofr_flag"));
                        String ifcoflag =rs1.getString("ifcofr_flag");
                        if(oldSts.equalsIgnoreCase("Y")){
                            if(ifcoflag.equalsIgnoreCase("Y")){
                                report.setBeforeSts("IFCOFR Audited");
                            }else {
                                report.setBeforeSts("Audited");
                            }
                        }
                        else{
                            report.setBeforeSts("Non-Audited");
                        }

                        String newSts =rs1.getString("as_new_status");
                        if (newSts.equalsIgnoreCase("A")){
                            report.setAfterSts("Audited");
                        }else if(newSts.equalsIgnoreCase("I")){
                            report.setAfterSts("IFCOFR Audited");
                        }
                        else{
                            report.setAfterSts("Non-Audited");
                        }

                        String sts= rs1.getString("as_req_status");
                        if (sts.equalsIgnoreCase("P")){
                            report.setReqSts("Pending");
                        }
                        report.setReq_tracksts(rs1.getString("rt_status"));
                        report.setReqOn(rs1.getString("rt_date"));

                        list.add(report);
                    }
                    log.info("size of the list: "+list.size());
                    return list;
                }

            });
        log.info("size of the list: "+list.size());
            return list;

    }
/////////////////////////////////////////////////////////
new code refernece 

package com.crs.dashboardService.repositories;

import com.crs.dashboardService.models.CrsRequestTrack;
import jakarta.transaction.Transactional;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Map;

@Repository
public interface CrsRequestTrackRepository extends JpaRepository<CrsRequestTrack, String> {
    @Query(value = "select RT_ID,RT_TYPE,RT_BRANCH,RT_STATUS,(SELECT NAME from WFL_user where wfl_user_id=rt_maker) RT_MAKER,RT_SUBTYPE,RT_FILE_NAME FROM CRS_REQUEST_TRACK WHERE RT_QED= to_date(:quarterEndDate,'DD/MM/YYYY') order by RT_STATUS, RT_ID desc", nativeQuery = true)
    List<Map<String, Object>> fetchCrsRequests(@Param("quarterEndDate") String quarterEndDate);

    @Transactional
    @Modifying
    @Query(value="update crs_request_track set rt_status = :rtStatus where rt_id = :reqId", nativeQuery = true)
    int cancelCrsRequests(@Param("reqId")String reqId, @Param("rtStatus")String rtStatus);
}

////////////////////////////////////////////////////////////////////////////////
data to render

103	31430	00067	DUMMY BRANCH	A	555555555	007	N	I	P	1	03-APR-23
105	31431	00004	DUMMY BRANCH	A	555555555	004	N	I	P	1	03-APR-23
325	31462	01000	DUMMY BRANCH	A	555555555	020	N	N	P	4	08-JUL-25
325	31463	95441	DUMMY BRANCH	A	555555555	002	N	I	P	4	08-JUL-25
362	31482	10109	DUMMY BRANCH	A	555555555	010	N	I	P	4	21-JUL-25
363	31483	24545	DUMMY BRANCH	A	555555555	006	N	I	P	4	21-JUL-25
