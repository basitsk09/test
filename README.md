 CrsRequestTrack crsRequestTrack = new CrsRequestTrack();
                        crsRequestTrack.setRtBranch(branchCode);
                        crsRequestTrack.setRtMaker(String.valueOf(mapUser.get("pf_number")));
                        crsRequestTrack.setRtStatus(Integer.parseInt("1"));
                        crsRequestTrack.setRtType("B");
                        crsRequestTrack.setRtSubType("A");
                        crsRequestTrack.setRtQed(quarterEndDate);
                        crsRequestTrackRepository.save(crsRequestTrack);
                        resultMap.put("status", true);
						
						
///////////////////////////////////////////////

"RT_ID" NUMBER(*,0) DEFAULT "CRS_REQUEST_TRACK_SEQ"."NEXTVAL", 
	"RT_MAKER" VARCHAR2(9), 
	"RT_DATE" DATE DEFAULT sysdate, 
	"RT_STATUS" NUMBER(1,0) NOT NULL ENABLE, 
	"RT_TYPE" VARCHAR2(1) NOT NULL ENABLE, 
	"RT_SUBTYPE" VARCHAR2(1) NOT NULL ENABLE, 
	"RT_FILE_NAME" VARCHAR2(50), 
	"RT_CHECKER" VARCHAR2(9), 
	"RT_ACTION_DT" DATE, 
	"RT_QED" DATE NOT NULL ENABLE, 
	"RT_BRANCH" VARCHAR2(5), 
	 PRIMARY KEY ("RT_ID")
  USING INDEX  ENABLE
  
  
////////////////////////////////////////////////


package com.crs.dashboardService.models;


import jakarta.persistence.*;
import lombok.Data;

@Data
@Entity
@Table(name = "CRS_REQUEST_TRACK")
public class CrsRequestTrack {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,    generator = "crs_seq")
    @SequenceGenerator(name = "crs_seq", sequenceName = "CRS_REQUEST_TRACK_SEQ", allocationSize = 1)
    @Column(name="RT_ID")
    public Long rtId;

    @Column(name="RT_MAKER")
    public String rtMaker;

    @Column(name="RT_DATE")
    public String rtDate;

    @Column(name="RT_STATUS")
    public Integer rtStatus;

    @Column(name="RT_TYPE")
    public String rtType;

    @Column(name="RT_SUBTYPE")
    public String rtSubType;

    @Column(name="RT_CHECKER")
    public String rtChecker;

    @Column(name="RT_ACTION_DT")
    public String rtActionDt;

    @Column(name="RT_QED")
    public String rtQed;

    @Column(name="RT_BRANCH")
    public String rtBranch;

    @Column(name="RT_FILE_NAME")
    public String rtFileName;

}
