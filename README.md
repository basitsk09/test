router.post("/downloadBranchList", extractToken, async (req, res) => {
  let token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({
      message: "Unauthorized api call, request not from legal source",
    });
  }
  try {
    const serviceUrl = devBaseServiceUrl + "/LHODashBoard/downloadBranchList";

    // The result is stored in 'response'
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

    // Check if the Java service returned an error
    if (!response.ok) {
        const errorText = await response.text();
        return res.status(response.status).send(errorText);
    }

    // Use the correct variable name 'response' here
    res.setHeader("Content-Type", response.headers.get("content-type"));
    res.setHeader(
      "Content-Disposition",
      response.headers.get("content-disposition")
    );

    // And here
    for await (const chunk of response.body) {
      res.write(chunk);
    }
    res.end();
    
  } catch (e) {
    console.error("Proxy Error:", e); // Log the actual error for debugging
    res.status(500).json({
      status: "Failed to bring response, no connection between LB and APP",
    });
  }
});
