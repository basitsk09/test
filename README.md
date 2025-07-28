 @RequestMapping(value = "/downloadExcel", method = RequestMethod.GET)
    public ModelAndView downloadExcel(HSSFWorkbook workbook, HttpServletRequest request, HttpServletResponse response) {
        List<ExcelCol> list = new ArrayList<ExcelCol>();
        // return a view which will be resolved by an excel view resolver
        return new ModelAndView("excelView", "listBooks", list);
    }
	
	
public class ExcelBuilder extends AbstractExcelView {

    static Logger log = Logger.getLogger(ExcelBuilder.class.getName());

    @Override
    protected void buildExcelDocument(
            Map<String, Object> model,
            HSSFWorkbook workbook,
            HttpServletRequest request,
            HttpServletResponse response
    ) throws Exception {

        try {


        // get data model which is passed by the Spring container
        List<ExcelCol> list = (List<ExcelCol>) model.get("listBooks");
        log.info("LIST SIZE : " +list.size());

        //log.info(list.get(0).getCol3());

        String fileName = "AuditStatusSample.xls"; //Your file name here.

        /*String col3Data = list.get(0).getCol3();
        if (col3Data != null) {
            fileName = "UploadError.xls";
        }*/

        if (list.size() > 0) {
            fileName = "UploadError.xls";
        }


        //response.setContentType("application/vnd.ms-excel"); //Tell the browser to expect an excel file
        response.setContentType("text/plain");
        response.setHeader("Content-Disposition", "attachment; filename="+fileName); //Tell the browser it should be named as the custom file name

        HSSFSheet sheet;
        if (list.size() > 0) {
            // create a new Excel sheet
            sheet = workbook.createSheet("Error");
        } else {
            // create a new Excel sheet
            sheet = workbook.createSheet("Sample");
        }

        sheet.setDefaultColumnWidth(30);
        //https://stackoverflow.com/questions/9187048/how-to-set-excel-default-row-height-in-apache-poi
        //sheet.setDefaultRowHeight((short) (2 * sheet.getDefaultRowHeightInPoints()));
        sheet.setDefaultRowHeightInPoints((2 * sheet.getDefaultRowHeightInPoints()));

        Font font = workbook.createFont();
        font.setFontName("Arial");
        font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
        font.setColor(HSSFColor.BLACK.index);
        //font.setFontHeightInPoints((short) (2 * font.getFontHeightInPoints()));
        font.setFontHeightInPoints((short) (15));

        // create style for header cells
        CellStyle style = workbook.createCellStyle();
        style.setAlignment(CellStyle.ALIGN_CENTER);
        style.setVerticalAlignment(CellStyle.ALIGN_CENTER);
        style.setFillForegroundColor(HSSFColor.CORNFLOWER_BLUE.index);
        style.setFillPattern(CellStyle.SOLID_FOREGROUND);
        style.setFont(font);
        //style.setWrapText(true);

        // create header row
        HSSFRow header = sheet.createRow(0);
        header.setHeightInPoints((2 * sheet.getDefaultRowHeightInPoints()));

        header.createCell(0).setCellValue("BranchCode");
        header.getCell(0).setCellStyle(style);

        header.createCell(1).setCellValue("AuditStatus");
        header.getCell(1).setCellStyle(style);

        if (list.size() > 0) {
            header.createCell(2).setCellValue("Error");
            header.getCell(2).setCellStyle(style);
        } else {
            // https://stackoverflow.com/questions/18716032/merging-cells-in-excel-using-apache-poi
            sheet.addMergedRegion(new CellRangeAddress(1,3,4,5));
        }

        //Note Start  ");//
        font.setFontHeightInPoints((short) (10));
        CellStyle styleNote = workbook.createCellStyle();
        styleNote.setFont(font);
        styleNote.setWrapText(true);

        HSSFRow note = sheet.createRow(1);
        //https://stackoverflow.com/questions/48040638/how-to-insert-a-linebreak-as-the-data-of-a-cell-using-apache-poi
        note.createCell(4)
                .setCellValue(
                        "Note: "
                        + "\n 1. Branch Code should be valid 5 digit. "
                        + "\n 2. Audit Status should be among :"
                        + "\n     N - For Marking Branch As Non Audited"
                        + "\n     I - For Marking As IFCOFR Audited"
                        + "\n     A - For marking As Audited (Non-IFCOFR)"
                );


        note.getCell(4).setCellStyle(styleNote);
        //Note End


        // create data rows
        int rowCount = 1;

        for (ExcelCol eCol : list) {
            //log.info("FoRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRRR : " + rowCount);
            HSSFRow aRow = sheet.createRow(rowCount++);
            //aRow.setHeight((short)-100);
            //aRow.setHeightInPoints((2 * sheet.getDefaultRowHeightInPoints()));

            aRow.createCell(0).setCellValue(eCol.getCol1());
            aRow.createCell(1).setCellValue(eCol.getCol2());
            aRow.createCell(2).setCellValue(eCol.getCol3());
        }

        workbook.write(response.getOutputStream());

        response.getOutputStream().close();

        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
