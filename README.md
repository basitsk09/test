const handleDownload = async () => {
    setLoading(true);
    try {
      const response = await axios.post(
        "/Server/EditBranch/downloadBranchList", // Correct new endpoint
        {}, // No request body is needed, but sending an empty object for POST
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
          responseType: "blob", // IMPORTANT: This tells axios to handle the response as binary data
        }
      );

      // Create a URL for the blob data
      const url = window.URL.createObjectURL(new Blob([response.data]));

      // Create a temporary link element to trigger the download
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", "AuditedBranchStatus.xlsx"); // Set the filename

      // Append to the DOM, click, and then remove
      document.body.appendChild(link);
      link.click();

      // Clean up by removing the link and revoking the object URL
      link.parentNode.removeChild(link);
      window.URL.revokeObjectURL(url);
    } catch (error) {
      console.error("Download failed:", error);
      setSnackbar({
        children: "Failed to download the report. Please try again later.",
        severity: "error",
      });
    } finally {
      setLoading(false);
    }
  };
  
 //////////////////////////////////////////
 
 
 router.post("/downloadBranchList", extractToken, async (req, res) => {
  let token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({
      message: "Unauthorized api call, request not from legal source",
    });
  }
  try {
    const serviceUrl = devBaseServiceUrl + "/LHODashBoard/downloadBranchList";

    const response = await fetch(serviceUrl, {
      method: "POST",
      body: JSON.stringify({
        user: req.user,
        data: req.data,
      }),
      headers: {
        "Content-Type": "application/json",
      },
    });
    // const data = await response.json();
    // res.json(data);
    res.setHeader("Content-Type", serviceResponse.headers.get("content-type"));
    res.setHeader(
      "Content-Disposition",
      serviceResponse.headers.get("content-disposition")
    );

    // Manually pipe the stream from fetch to the Express response
    // This is the correct way to handle a file stream with fetch
    for await (const chunk of serviceResponse.body) {
      res.write(chunk);
    }
    res.end();
  } catch (e) {
    res.status(500).json({
      status: "Failed to bring response, no connection between LB and APP",
    });
  }
});
