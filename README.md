
  if (
    req.user.user_role !== "1" &&
    req.user.module !== "LFAR" &&
    req.user.user_role !== "11"
  ) {
    return res.status(401).json({
      message: "Unauthorized api call, request not from legal source",
    });
  }


  try {
    let serviceUrl = "";
    if (runningEnvironment === "dev") {
      serviceUrl = URL_CONST.DEV_URL[req.user.module].downloadComplianceZip;
    } else {
      serviceUrl = URL_CONST.PROD_URL[req.user.module].downloadComplianceZip;
    }
    console.log("URL Triggered " + serviceUrl)
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
    console.error(e);
    res.status(500).json({
      status: "Failed to bring response, no connection between LB and APP",
    });
  }


})
