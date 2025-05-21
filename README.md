private void getLfar3Evidences(String submissionId, ZipOutputStream zipOutputStream) throws IOException, SQLException {
    List<Object[]> reportFiles = lfar3Repository.getBlobStreamBySubmissionId(submissionId);
    if (!reportFiles.isEmpty()) {
        String folderName = "Lfar3Evidences/";
        zipOutputStream.putNextEntry(new ZipEntry(folderName));
        zipOutputStream.closeEntry();
        int fileCounter = 1;

        for (Object[] row : reportFiles) {
            Blob blob = (Blob) row[0];
            String fileName = (String) row[1];

            if (blob != null && blob.length() > 0) {
                String zipPath = folderName + fileCounter + "_" + fileName;
                try (InputStream in = blob.getBinaryStream()) {
                    streamFileToZip(zipOutputStream, zipPath, in);
                }
                fileCounter++;
            }
        }
    }
}


private void streamFileToZip(ZipOutputStream zipOutputStream, String fileName, InputStream inputStream) throws IOException {
    zipOutputStream.putNextEntry(new ZipEntry(fileName));
    byte[] buffer = new byte[8192]; // 8 KB buffer
    int bytesRead;
    while ((bytesRead = inputStream.read(buffer)) != -1) {
        zipOutputStream.write(buffer, 0, bytesRead);
    }
    zipOutputStream.closeEntry();
    log.info(fileName + " added to zip (streamed)");
}

@Query("SELECT evidence_file, evidence_name FROM crs_lfar WHERE report_master_list_id_fk = :submissionId")
List<Object[]> getBlobStreamBySubmissionId(String submissionId);