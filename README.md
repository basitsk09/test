You're getting a 500 error because your Express server is incorrectly trying to process the file data as JSON.
The Java service sends a binary file (the Excel report), but your Express code (const data = await response.json()) attempts to parse that binary data as if it were text, which fails and triggers the catch block.
The solution is to make your Express server stream the file directly from the Java service to the React client without trying to interpret it.
## Backend Fix (Express)
You need to change your Express route to pipe the response. Using axios on the backend is easier for streaming than node-fetch.
// backend express route

const axios = require("axios"); // Make sure axios is installed in your Express project

router.post("/downloadBranchList", extractToken, async (req, res) => {
  if (!req.headers.authorization) {
    return res.status(401).json({ message: "Unauthorized" });
  }

  try {
    const serviceUrl = devBaseServiceUrl + "/LHODashBoard/downloadBranchList";

    // Use axios to call the Java service and get the response as a stream
    const response = await axios({
      method: 'POST',
      url: serviceUrl,
      data: {           // The body to send to the Java service
        user: req.user,
        data: req.data,
      },
      responseType: 'stream', // This is the key! Get the response as a stream.
    });

    // Pass the headers from the Java service (like Content-Type) to the client
    res.setHeader('Content-Type', response.headers['content-type']);
    res.setHeader('Content-Disposition', response.headers['content-disposition']);

    // Pipe the file stream from the Java service directly to the response
    response.data.pipe(res);

  } catch (error) {
    console.error("Failed to proxy download request:", error);
    res.status(500).json({
      status: "Failed to download file.",
    });
  }
});

## Frontend Fix (React)
Your axios call on the frontend has a minor syntax error. The axios.post method signature is axios.post(url, data, config). You are passing the config object in the data position.
Here is the corrected syntax:
// frontend axios call

try {
  const response = await axios.post(
    "/Server/EditBranch/downloadBranchList",
    {}, // The request body (data), an empty object is fine if not needed
    {   // The config object, containing headers and responseType
      headers: {
        Authorization: `Bearer ${localStorage.getItem("token")}`,
      },
      responseType: "blob", 
    }
  );

  // ... your existing code to handle the blob download
  
} catch (error) {
  // ... error handling
}

By making these changes, your Express server will correctly act as a proxy for the file, and the frontend will receive the blob as expected.
