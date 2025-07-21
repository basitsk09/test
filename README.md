import React, { useState, useEffect } from "react";
import {
  TextField,
  Grid,
  Button,
  Typography,
  Divider,
  TableContainer,
  Table,
  TableHead,
  TableRow,
  TableCell,
  TableBody,
  RadioGroup,
  Radio,
  FormControlLabel,
  FormControl,
  FormLabel,
  Paper,
  Box,
  Container,
  Dialog,
  DialogContent,
  CircularProgress,
  Snackbar,
  Alert,
} from "@mui/material";
import SearchIcon from "@mui/icons-material/Search";
import DownloadIcon from "@mui/icons-material/CloudDownload";
import SaveIcon from "@mui/icons-material/Save";
import DiscardIcon from "@mui/icons-material/Cancel";
import axios from "axios";
import { useNavigate } from "react-router-dom";
import { encrypt } from "../Security/AES-GCM256";

// Encryption constants
const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

export default function FrtSingleBranchAuditStatus() {
  document.title = "CRS | FRT Single Branch Audit";
  const navigate = useNavigate();

  // --- State Management ---
  const [branchCode, setBranchCode] = useState("");
  const [branchDetails, setBranchDetails] = useState(null);
  const [initialAuditStatus, setInitialAuditStatus] = useState("");
  const [selectedAuditStatus, setSelectedAuditStatus] = useState("");
  const [showDetails, setShowDetails] = useState(false);
  const [userRole, setUserRole] = useState("");
  const [loading, setLoading] = useState(false);
  const [snackbar, setSnackbar] = useState(null);
  const user = JSON.parse(localStorage.getItem("user"));

  useEffect(() => {
    const loggedInUser = JSON.parse(localStorage.getItem("user"));
    if (loggedInUser && loggedInUser.user_role) {
      if (loggedInUser.user_role !== "96" && loggedInUser.user_role !== "94") {
        navigate("/");
      }
      setUserRole(loggedInUser.user_role);
    } else {
      navigate("/");
    }
  }, [navigate]);

  // --- Handlers ---
  const handleSnackbarClose = () => setSnackbar(null);

  const handleReset = () => {
    setBranchCode("");
    setBranchDetails(null);
    setShowDetails(false);
    setSelectedAuditStatus("");
    setInitialAuditStatus("");
  };

  /**
   * Fetches branch details from the server based on the entered branch code.
   * Updated to use the new payload structure.
   */
  const handleSearch = async () => {
    if (!branchCode || branchCode.length < 5) {
      setSnackbar({
        children: "Please enter a valid 5-digit branch code.",
        severity: "error",
      });
      return;
    }
    setLoading(true);
    handleReset();
    setBranchCode(branchCode);

    try {
      let jsonFormData = JSON.stringify({
        branchCode: branchCode,
        quarterEndDate: user.quarterEndDate,
        reqType: "A",
        reqSubType: "S",
      });
      jsonFormData = await encrypt(iv, salt, jsonFormData);
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

      const response = await axios.post(
        "/Server/EditBranch/fetchBranchDetails",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      if (
        response.data?.result?.branchData &&
        response.data.result.branchData.CODE
      ) {
        console.log(response.data?.result?.isRequestPending);
        if (response.data?.result?.isRequestPending) {
          // setVisibleData(false);
          setSnackbar({
            children: "Request is already created for " + branchCode + ".",
            severity: "error",
          });
          //handleDialogClose();
        } else {
          const details = response.data.result.branchData;
          setBranchDetails(details);

          // Directly use the AUDITABLE field which can be 'A', 'N', or 'I'
          const status = details.AUDITABLE || "N";
          setInitialAuditStatus(status);
          setSelectedAuditStatus(status);
          setShowDetails(true);
        }
      } else {
        setSnackbar({
          children: "Branch does not exist.",
          severity: "warning",
        });
      }
    } catch (error) {
      console.error("Search failed:", error);
      setSnackbar({
        children: "An error occurred while searching. Please try again.",
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

  /**
   * Submits the changed audit status to the server.
   * Updated to map the new state structure to the expected save payload.
   */
  const handleSave = async () => {
    if (selectedAuditStatus === initialAuditStatus) {
      setSnackbar({
        children: "There is no change in audit status to save.",
        severity: "info",
      });
      return;
    }

    setLoading(true);
    // Construct the RO Details string from network, module, and region
    const roDetails = `${branchDetails.NETWORK || ""} ${
      branchDetails.MODULE || ""
    } ${branchDetails.REGION || ""}`.trim();

    // Create the payload expected by the '/FRTUser/saveMe' endpoint

    let jsonFormData = JSON.stringify({
      branchCode: branchDetails.CODE,
      branchName: branchDetails.NAME,
      requestId: branchDetails.requestId, // This might be undefined, which the backend handles as an insert
      circleCode: branchDetails.CIRCLE,
      roCode: roDetails,
      auditSts: selectedAuditStatus,
    });

    console.log("savePayload", jsonFormData);
    await encrypt(iv, salt, jsonFormData).then(function (result) {
      jsonFormData = result;
    });
    let savePayload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };

    try {
      // NOTE: This assumes the save endpoint and its payload structure remain the same.
      const response = await axios.post(
        "/Server/EditBranch/crsAuditStatusChangeRequest",
        savePayload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      if (response.data && response.data !== "NODATA") {
        setSnackbar({
          children: "Request Created Successfully",
          severity: "success",
        });
        handleReset();
      } else {
        throw new Error("Failed to save data.");
      }
    } catch (error) {
      console.error("Save failed:", error);
      setSnackbar({
        children: "The request could not be saved due to a technical error.",
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };

  const handleDownload = () => {
    const form = document.createElement("form");
    form.method = "post";
    form.action = "/FRTUser/downloadForm";
    document.body.appendChild(form);
    form.submit();
    document.body.removeChild(form);
  };

  // --- Helper to format RO Details ---
  const formatRoDetails = (details) => {
    if (!details) return "--";
    return `${details.NETWORK || "---"} ${details.MODULE || "---"} ${
      details.REGION || "---"
    }`;
  };

  // --- JSX ---
  return (
    <>
      <Box sx={{ p: 2 }}>
        <Typography variant="h5" gutterBottom>
          {user.user_role === "94"
            ? "View Branch Details"
            : "View/Update Branch Audit Status"}
        </Typography>
        <Divider />
      </Box>

      <Container maxWidth="xl" sx={{ mt: 2 }}>
        <Paper elevation={3} sx={{ p: 3 }}>
          <Grid container spacing={3} alignItems="center">
            {/* Search and Download Section */}
            <Grid item xs={12} md={3}>
              <TextField
                label="Enter Branch Code"
                variant="outlined"
                fullWidth
                value={branchCode}
                onChange={(e) => setBranchCode(e.target.value)}
                inputProps={{ maxLength: 5 }}
              />
            </Grid>
            <Grid item xs={12} md={2}>
              <Button
                variant="contained"
                startIcon={<SearchIcon />}
                onClick={handleSearch}
                fullWidth
              >
                Search
              </Button>
            </Grid>
            <Grid
              item
              xs={12}
              md={4}
              sx={{ textAlign: { xs: "left", md: "right" } }}
            >
              <Typography variant="body1">
                List Of All Branches With Audit Status:
              </Typography>
            </Grid>
            <Grid item xs={12} md={3}>
              <Button
                variant="contained"
                color="secondary"
                startIcon={<DownloadIcon />}
                onClick={handleDownload}
                fullWidth
              >
                Download List
              </Button>
            </Grid>
          </Grid>

          {/* Details and Actions Section - visible after search */}
          {showDetails && (
            <Box mt={4}>
              <Typography variant="h6" gutterBottom>
                Existing Branch Audit Status
              </Typography>
              <TableContainer component={Paper} elevation={2}>
                <Table>
                  <TableHead sx={{ backgroundColor: "#b9def0" }}>
                    <TableRow>
                      <TableCell align="center">Branch</TableCell>
                      <TableCell align="center">Name</TableCell>
                      <TableCell align="center">Circle</TableCell>
                      <TableCell align="center">RO Details</TableCell>
                      <TableCell align="center">Audited Status</TableCell>
                      <TableCell align="center">IFCOFR</TableCell>
                    </TableRow>
                  </TableHead>
                  <TableBody>
                    <TableRow>
                      <TableCell align="center">
                        {branchDetails?.CODE}
                      </TableCell>
                      <TableCell align="center">
                        {branchDetails?.NAME}
                      </TableCell>
                      <TableCell align="center">
                        {branchDetails?.CIRCLE}
                      </TableCell>
                      <TableCell align="center">
                        {formatRoDetails(branchDetails)}
                      </TableCell>
                      <TableCell align="center">
                        {branchDetails?.AUDITABLE === "N"
                          ? "Non-Audited"
                          : "Audited"}
                      </TableCell>
                      <TableCell align="center">
                        {branchDetails?.AUDITABLE === "I"
                          ? "Yes"
                          : branchDetails?.AUDITABLE === "A"
                          ? "No"
                          : "--"}
                      </TableCell>
                    </TableRow>
                  </TableBody>
                </Table>
              </TableContainer>

              {/* Radio Button Section - visible only for Maker role */}
              {userRole === "96" && (
                <Box mt={4}>
                  <Grid container spacing={2}>
                    <Grid item xs={12}>
                      <FormControl component="fieldset">
                        <FormLabel component="legend">
                          Update Audit Status
                        </FormLabel>
                        <RadioGroup
                          row
                          value={
                            selectedAuditStatus.charAt(0) === "I"
                              ? "A"
                              : selectedAuditStatus.charAt(0)
                          }
                          onChange={(e) =>
                            setSelectedAuditStatus(e.target.value)
                          }
                        >
                          <FormControlLabel
                            value="N"
                            control={<Radio />}
                            label="Non-Audited Branch"
                          />
                          <FormControlLabel
                            value="A"
                            control={<Radio />}
                            label="Audited Branch"
                          />
                        </RadioGroup>
                      </FormControl>
                    </Grid>

                    {/* IFCOFR Section - visible only when 'Audited' is selected */}
                    {selectedAuditStatus.charAt(0) !== "N" && (
                      <Grid item xs={12}>
                        <FormControl component="fieldset">
                          <FormLabel component="legend">IFCOFR Audit</FormLabel>
                          <RadioGroup
                            row
                            value={selectedAuditStatus}
                            onChange={(e) =>
                              setSelectedAuditStatus(e.target.value)
                            }
                          >
                            <FormControlLabel
                              value="I"
                              control={<Radio />}
                              label="Yes (IFCOFR Audited)"
                            />
                            <FormControlLabel
                              value="A"
                              control={<Radio />}
                              label="No"
                            />
                          </RadioGroup>
                        </FormControl>
                      </Grid>
                    )}
                  </Grid>

                  {/* Save/Discard Buttons */}
                  <Box mt={3} display="flex" gap={2}>
                    <Button
                      variant="contained"
                      color="success"
                      startIcon={<SaveIcon />}
                      onClick={handleSave}
                    >
                      Save
                    </Button>
                    <Button
                      variant="contained"
                      color="error"
                      startIcon={<DiscardIcon />}
                      onClick={handleReset}
                    >
                      Discard
                    </Button>
                  </Box>
                </Box>
              )}
            </Box>
          )}
        </Paper>
      </Container>

      {/* Loading Dialog */}
      <Dialog open={loading}>
        <DialogContent sx={{ display: "flex", alignItems: "center", gap: 2 }}>
          <CircularProgress />
          <Typography>Processing...</Typography>
        </DialogContent>
      </Dialog>

      {/* Snackbar for Notifications */}
      {snackbar && (
        <Snackbar
          open
          anchorOrigin={{ vertical: "top", horizontal: "center" }}
          onClose={handleSnackbarClose}
          autoHideDuration={6000}
        >
          <Alert
            onClose={handleSnackbarClose}
            severity={snackbar.severity}
            variant="filled"
            sx={{ width: "100%" }}
          >
            {snackbar.children}
          </Alert>
        </Snackbar>
      )}
    </>
  );
}
//////////////////////////////////////////////////////////////////////////

This is reference for other jaspers functioanlity

 public Map<String, Object> dashPreview(Map<String, Object> payload) throws SQLException {
        Map<String, Object> user = (Map<String, Object>) payload.get("user");
        Map<String, Object> data = (Map<String, Object>) payload.get("data");
        Map<String, Object> result = new HashMap<>();
        log.info("data: " + data);
        log.info("user: " + user);
        log.info(user.get("circleCode").toString());
        log.info(user.get("roCode").toString());
        String code;
        if (user.get("user_role").equals("75")) {
            code = user.get("circleCode").toString() + user.get("roCode").toString();
        } else {
            code = user.get("circleCode").toString();
        }
        String quarterDate = user.get("quarterEndDate").toString();
        String jrxmlName = data.get("jrxmlName").toString();
        Map<String, Object> pdfCont = new HashMap<>();

        Connection conn = null;
        String rootPath = Objects.requireNonNull(Thread.currentThread().getContextClassLoader().getResource("")).getPath();
        log.info("");
        String configPath = rootPath + "common.properties";
        byte[] pdfContent;
        byte[] xlsxContent;
        try {
            Properties config = new Properties();
            config.load(new FileInputStream(configPath));
            conn = dataSource.getConnection();
            String jasperFilePath = rootPath + "jasper" + File.separator + jrxmlName + ".jasper";
            Map<String, Object> param = new HashMap<>();
            String outFileDest = config.getProperty("REPORT_HOME_DIR") + "created/" + user.get("circleCode").toString();
            File outFile = new File(outFileDest);
            if (!outFile.exists()) {
                boolean pathCreation = outFile.mkdirs();
                if (pathCreation) {
                    log.info("outFilePath created");
                }
            }
            String outFilePath = outFileDest + File.separator + jrxmlName + ".pdf";
            String outFileXlsx = outFileDest + File.separator + jrxmlName + ".xlsx";
            param.put("CODE", code);
            param.put("DATE", quarterDate);
            pdfContent = jasperBuilder(jasperFilePath, param, conn, outFilePath, outFileDest);
            xlsxContent = jasperBuilder(jasperFilePath, param, conn, outFileXlsx, outFileDest);
            pdfCont.put("pdfContent", pdfContent);
            pdfCont.put("xlsxContent", xlsxContent);
            pdfCont.put("pdfName", user.get("circleCode").toString()+"_"+data.get("reportName").toString());
            log.info("pdfContent added to map");
        } catch (IOException | SQLException e) {
            log.severe("Exception Occurred " + e);
        } finally {
            if (conn != null) {
                conn.close();
                log.info("connection closed");
            }
        }
        return pdfCont;
    }

    private static byte[] jasperBuilder(String jasperFilePath, Map<String, Object> param, Connection conn, String outFilePath, String outFileDest) throws IOException {
        byte[] pdfContent = new byte[0];
        JasperPrint jasperPrint;
        try {
            jasperPrint = JasperFillManager.fillReport(jasperFilePath, param, conn);
            File file = new File(outFileDest);
            log.info("file: {}" + file);
            if (!file.exists()) {
                log.info("file path does not exist!");
                file.mkdirs();
            }
            log.info("file path exists");

            if (outFilePath.split("\\.")[1].equalsIgnoreCase("pdf")) {
                JasperExportManager.exportReportToPdfFile(jasperPrint, outFilePath);
            } else if (outFilePath.split("\\.")[1].equalsIgnoreCase("xlsx")) {
                JRXlsxExporter excelExporter = new JRXlsxExporter();
                OutputStream out = new FileOutputStream(outFilePath);
                excelExporter.setParameter(JRXlsExporterParameter.JASPER_PRINT, jasperPrint);
                excelExporter.setParameter(JRXlsExporterParameter.OUTPUT_STREAM, out);
                excelExporter.setParameter(JRXlsExporterParameter.IS_DETECT_CELL_TYPE, Boolean.TRUE);
                excelExporter.setParameter(JRXlsExporterParameter.IS_WHITE_PAGE_BACKGROUND, Boolean.FALSE);
                excelExporter.setParameter(JRXlsExporterParameter.IS_REMOVE_EMPTY_SPACE_BETWEEN_ROWS, Boolean.TRUE);
                excelExporter.setParameter(JRXlsExporterParameter.IS_IGNORE_CELL_BORDER, Boolean.TRUE);
                excelExporter.exportReport();
            }
        } catch (JRException e) {
            throw new RuntimeException(e);
        }
        log.info("file exported");
        File file2 = new File(outFilePath);

        if (outFilePath.split("\\.")[1].equalsIgnoreCase("xlsx")) {
            try (FileInputStream fis = new FileInputStream(file2);
                 Workbook workbook = new XSSFWorkbook(fis);
                 ByteArrayOutputStream bos = new ByteArrayOutputStream()) {

                // Write the workbook to the ByteArrayOutputStream
                workbook.write(bos);
                pdfContent = bos.toByteArray(); // Convert to a byte array
            } catch (Exception e) {
                log.severe("Exception Occurred " + e);
            }
        } else {
            try (PDDocument document = PDDocument.load(file2); ByteArrayOutputStream outputStream = new ByteArrayOutputStream()) {
                document.save(outputStream);
                pdfContent = outputStream.toByteArray();
            } catch (IOException e) {
                log.severe("{} File not found. Kindly try again." + outFilePath);
                //   e.printStackTrace();
            }
            log.info("pdfContent generated");
        }
        return pdfContent;
    }
////////////////////////////////////////////////////////////////////////////////////////////

this is old code to download jasper

  @RequestMapping(value = "/downloadForm", method = RequestMethod.POST)
    public ModelAndView download(@ModelAttribute("command") FrtSingleBranch frt, BindingResult result,
                                 RedirectAttributes redirectAttributes, HttpServletRequest request, HttpServletResponse response)
            throws SQLException, JRException, IOException {
        HttpSession session = request.getSession();
        SessionBean sessionBean = new SessionBean(request.getSession());
        String userCapacity = sessionBean.getUserCapability();
        if (session == null || session.getAttribute(CommonConstants.USER_ID) == null || request.getSession().getId() == null ||
                !(userCapacity.equalsIgnoreCase("96") || userCapacity.equalsIgnoreCase("94"))) {
            log.error("***Unauthorised Access***" + session.getAttribute(CommonConstants.USER_CAPABILITY));
            return new ModelAndView("500");
        }
        log.info("inside db");
        String userId = session.getAttribute(CommonConstants.USER_ID).toString();
        String opt = session.getAttribute(CommonConstants.OPT).toString();
        String displayResponce = "";

        Configuration config;
        Connection con = null;
        try {
            con = dataSource.getConnection();

            String jrxmlName = "Frt_Branch_List";
            log.info("jrxmlName " + jrxmlName);

            DateFormat dateFormat = new SimpleDateFormat("ddMMyyyy HHmmss");
            Date date = new Date();
            String timeStamp = dateFormat.format(date).replace(" ", "");
            String fileName = jrxmlName + ".xlsx";
            String outFilePath = AutoClean.cleanedPath(request.getSession(), AutoClean.TIMESTAMP, fileName, "", request);
            String JasperFilePath = AutoClean.cleanedPath(request.getSession(), AutoClean.JASPER_FILE_PATH, jrxmlName, "", request);

            Map<String, Object> param = new HashMap<String, Object>();
            param.put("IS_IGNORE_PAGINATION", true);
            JasperPrint jasperPrint;
            jasperPrint = JasperFillManager.fillReport(JasperFilePath, param, con);


            OutputStream out2 = new FileOutputStream(new File(outFilePath));
            JRXlsxExporter excelExporter = new JRXlsxExporter();
            excelExporter.setExporterInput(new SimpleExporterInput(jasperPrint));
            excelExporter.setExporterOutput(new SimpleOutputStreamExporterOutput(out2));
            SimpleXlsxReportConfiguration configuration = new SimpleXlsxReportConfiguration();
            configuration.setDetectCellType(true);
            configuration.setWhitePageBackground(false);
            configuration.setRemoveEmptySpaceBetweenRows(true);
            configuration.setIgnoreCellBorder(true);
            excelExporter.setConfiguration(configuration);
            excelExporter.exportReport();
            log.info("exportdoneeee");

            File file2 = new File(outFilePath);
            byte[] pdfContent = FileUtils.readFileToByteArray(file2);
            String contentType = "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
            response.setContentType(contentType);
            response.setHeader("Content-Disposition", "attachment;filename=" + CleanPath.cleanString("Audited Branch Status" + ".xlsx"));
            OutputStream out1 = null;
            out1 = (OutputStream) response.getOutputStream();
            out1.write(pdfContent);
            out1.close();
            response.flushBuffer();
            log.info("Download Success");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != con) {
                con.close();
                log.warn("*** Connection Closed ***");
            }
        }
        return null;
    }
//////////////////////////////////////////////////////////////////////////////////////

My existing dependencies

  <dependency>
            <groupId>net.sf.jasperreports</groupId>
            <artifactId>jasperreports</artifactId>
            <version>6.17.0</version>
            <!--<version>7.0.0</version>-->
        </dependency>

        <!-- https://mvnrepository.com/artifact/net.sf.jasperreports/jasperreports-fonts -->
        <dependency>
            <groupId>net.sf.jasperreports</groupId>
            <artifactId>jasperreports-fonts</artifactId>
            <version>6.17.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/net.sf.jasperreports/jasperreports-functions -->
        <dependency>
            <groupId>net.sf.jasperreports</groupId>
            <artifactId>jasperreports-functions</artifactId>
            <version>6.17.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/net.sf.jasperreports/jasperreports-metadata -->
        <dependency>
            <groupId>net.sf.jasperreports</groupId>
            <artifactId>jasperreports-metadata</artifactId>
            <version>6.17.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-configuration2 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-configuration2</artifactId>
            <version>2.8.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-collections4 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-collections4</artifactId>
            <version>4.4</version>
        </dependency>


        <!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.16.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-digester3 -->
        <dependency>
            <groupId>commons-digester</groupId>
            <artifactId>commons-digester</artifactId>
            <version>2.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/commons-configuration/commons-configuration -->
        <dependency>
            <groupId>commons-configuration</groupId>
            <artifactId>commons-configuration</artifactId>
            <version>1.6</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.pdfbox/pdfbox -->
        <dependency>
            <groupId>org.apache.pdfbox</groupId>
            <artifactId>pdfbox</artifactId>
            <version>2.0.29</version>
        </dependency>
        <dependency>
            <groupId>com.lowagie</groupId>
            <artifactId>itext</artifactId>
            <version>2.1.7</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.poi/poi -->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>5.4.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>5.2.3</version> <!-- Use the latest version available -->
        </dependency>

        <dependency>
            <groupId>org.apache.xmlbeans</groupId>
            <artifactId>xmlbeans</artifactId>
            <version>5.1.1</version> <!-- Use the latest version available -->
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
