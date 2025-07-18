package com.crs.commonReportsService.models;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

// Report RW-02

@Getter
@Setter
@Entity
@Table(name = "CRS_STND_ASSETS")
public class CrsStndAssets {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY, generator = "CRS_STND_SEQ")
    @SequenceGenerator(name = "CRS_STND_SEQ", sequenceName = "CRS_STND_SEQ", allocationSize = 1)
    @Column(name = "STND_ASSETS_SEQ")
    private int stndassetsseq;

    @Column(name = "STND_ASTS_NAME_OF_BORROWER")
    private String stndastsnameofborrower;

    @Column(name = "STND_ASTS_INFRA_NON_INFRA")
    private String stndastsinfranoninfra;

    @Column(name = "STND_ASTS_INFRA_WITHIN2YRS")
    private String stndastsinfrawithin2YRS;

    @Column(name = "STND_ASTS_INFRA_ACCTS2YRS")
    private String stndastsinfraaccts2YRS;

    @Column(name = "STND_ASTS_NONINFRA_WITHIN1YR")
    private String stndastsnoninfrawithin1YR;

    @Column(name = "STND_ASTS_NONINFRA_ACCTS1YR")
    private String stndastsnoninfraaccts1YR;

    @Column(name = "STND_ASSETS_BRANCH")
    private String stndastsbranch;

    @Column(name = "STND_ASSETS_DATE")
    private String stndastsdate;

    @Column(name = "REPORT_MASTER_LIST_ID_FK")
    private int stndidfk;
}

////////////////////////////////


{

        log.info("map >>> " + map);

        // Data Receive here
        Map<String, Object> data = (Map<String, Object>) map.get("data");
        Map<String, Object> loginuserData = (Map<String, Object>) map.get("user");
        log.info("data >>> " + data);
        log.info("loginuserData >>> " + loginuserData);

        //List of ROW Data
        List<String> dataList = (List<String>) data.get("value");

        String submissionId=String.valueOf(data.get("submissionId"));

        CrsStndAssets entity = new CrsStndAssets();
        try {
            // :: Insert the Branch ::
            entity.setStndastsbranch((String) loginuserData.get("branch_code"));

            // :: Insert the QED ::
            entity.setStndastsdate((String) loginuserData.get("quarterEndDate"));

            // Assign data based on dataList

            // 1st Element :: nameOfBorrowerList
            entity.setStndastsnameofborrower(dataList.get(0));
            //2nd Element :: infraNonInfraList
            entity.setStndastsinfranoninfra(dataList.get(1));

            // Condition Check Variable INFRA or NONINFRA
            String infraNonInfraList = dataList.get(1);
            if (infraNonInfraList.equalsIgnoreCase("FOR INFRA")) {
                entity.setStndastsinfraaccts2YRS(dataList.get(3));
                entity.setStndastsinfrawithin2YRS(dataList.get(2));
            } else if (infraNonInfraList.equalsIgnoreCase("FOR NON INFRA")) {
                entity.setStndastsnoninfraaccts1YR(dataList.get(3));
                entity.setStndastsnoninfrawithin1YR(dataList.get(2));
            }

            // Report Submission ID_FK
            entity.setStndidfk(Integer.parseInt(submissionId));


            // Check if ROW -ID is empty or null for insert scenario
            if (dataList.get(5).trim().isEmpty() || dataList.get(5) == null) {
                log.info("Entity Data for Insert: " + entity);

                // Save the new entity
                CrsStndAssets savedEntity = crsStndAssetsRepository.save(entity);
                log.info("New row inserted successfully: " + savedEntity);

                // Sending data to response MAP
                Map<String,Object>resultDataMap=new HashMap<>();
                resultDataMap.put("status",true);
                resultDataMap.put("newRowNum",savedEntity.getStndassetsseq());

                // Setting Up Success & Response Data
                ResponseVO<Map<String,Object>> responseVO = new ResponseVO();
                responseVO.setStatusCode(HttpStatus.OK.value());
                responseVO.setMessage("Data Inserted successfully");
                responseVO.setResult(resultDataMap);

                return new ResponseEntity<>(responseVO, HttpStatus.OK);
            } else {
                // ID is provided, check for existence and update
                int id = Integer.parseInt(dataList.get(5));
                log.info("Received ID for Update: " + id + " Is ID Existed :" + crsStndAssetsRepository.existsBystndassetsseq(id));

                ResponseVO<Map<String,Object>> responseVO = new ResponseVO();
                Map<String,Object> resultDataMap=new HashMap<>();
                if (crsStndAssetsRepository.existsBystndassetsseq(id)) {
                    // Update existing row
                    entity.setStndassetsseq(id);  // Set the ID for update

                    CrsStndAssets updatedEntity = crsStndAssetsRepository.save(entity);
                    log.info("Data Updated successfully: " + updatedEntity);

                    resultDataMap.put("status",true);
                    resultDataMap.put("newRowNum",entity.getStndassetsseq());

                    responseVO.setStatusCode(HttpStatus.OK.value());
                    responseVO.setMessage("Data Updated successfully");
                    responseVO.setResult(resultDataMap);
                    return new ResponseEntity<>(responseVO, HttpStatus.OK);
                } else {
                    // ID not found, handle error
                    log.info("ID " + id + " not found for update");
                    responseVO.setStatusCode(HttpStatus.BAD_REQUEST.value());
                    responseVO.setMessage("Invalid ID provided. Record not found.");
                    resultDataMap.put("status",false);
                    responseVO.setResult(resultDataMap);
                    return new ResponseEntity<>(responseVO, HttpStatus.BAD_REQUEST);
                }
            }

        } catch (RuntimeException e) {
            ResponseVO<String> responseVO = new ResponseVO();
            log.info("Exception Occurred :"+e.getCause());
            responseVO.setResult("false");
            responseVO.setStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
            responseVO.setMessage("Exception Occurred: " + e.getMessage());
            return new ResponseEntity<>(responseVO, HttpStatus.INTERNAL_SERVER_ERROR);
        }

    }
