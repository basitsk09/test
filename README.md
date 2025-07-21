
 try {
      const response = await axios.post(
        "/Server/EditBranch/downloadBranchList", // Correct new endpoint
        // No request body is needed, but sending an empty object for POST
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
          responseType: "blob", // IMPORTANT: This tells axios to handle the response as binary data
        }
      );
	  
//////////////////////////////////////////////////////

express


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
    const data = await response.json();
    res.json(data);
  } catch (e) {
    res.status(500).json({
      status: "Failed to bring response, no connection between LB and APP",
    });
  }
});
