Of course. You can achieve the same file streaming behavior using only the native fetch API in your Express backend.
The key is to read the response body from fetch as a stream and write its chunks directly to the Express response, rather than trying to parse it as JSON.
Express Backend Fix (Using Fetch Only)
Here’s the updated code for your Express route. This version correctly handles the binary file stream from your Java service.
// backend express route

router.post("/downloadBranchList", extractToken, async (req, res) => {
  if (!req.headers.authorization) {
    return res.status(401).json({ message: "Unauthorized" });
  }

  try {
    const serviceUrl = devBaseServiceUrl + "/LHODashBoard/downloadBranchList";

    // Call the Java service
    const serviceResponse = await fetch(serviceUrl, {
      method: "POST",
      body: JSON.stringify({
        user: req.user,
        data: req.data,
      }),
      headers: {
        "Content-Type": "application/json",
        // Forward the auth token if your Java service needs it
        "Authorization": req.headers.authorization, 
      },
    });

    // Check if the Java service returned an error
    if (!serviceResponse.ok) {
      const errorText = await serviceResponse.text();
      return res.status(serviceResponse.status).send(errorText);
    }

    // Forward the file headers from the Java service to the client
    res.setHeader('Content-Type', serviceResponse.headers.get('content-type'));
    res.setHeader('Content-Disposition', serviceResponse.headers.get('content-disposition'));

    // Manually pipe the stream from fetch to the Express response
    // This is the correct way to handle a file stream with fetch
    for await (const chunk of serviceResponse.body) {
      res.write(chunk);
    }
    res.end();

  } catch (error) {
    console.error("Failed to proxy download request:", error);
    res.status(500).json({
      status: "Failed to download file due to a proxy error.",
    });
  }
});

Your frontend code does not need any changes from the previously corrected version. This backend fix ensures the file is properly streamed through your Express server to the browser.
