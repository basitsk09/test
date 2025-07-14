const express = require("express");
const fetch = require("node-fetch");
const fs = require("fs");
const path = require("path");
const { pipeline } = require("stream/promises");
const { createWriteStream } = require("fs");
const { Transform } = require("stream");
const JSONStream = require("JSONStream");

// --- Configuration ---
const app = express();
const port = 3000;

// This is a placeholder for your backend URL.
// IMPORTANT: Replace this with the actual URL of your Spring service.
const BACKEND_URL = "http://your-backend-service.com/api/submitDownload";

// --- Middleware (Simulated for completeness) ---
// This is a placeholder for your 'extractToken', 'DataValidator', and 'LegalRequest' middleware.
// In this example, we'll just create dummy user/data objects.
const simulateRequestMiddleware = (req, res, next) => {
  req.user = {
    circleCode: "MH",
    quarterEndDate: "31/03/2025",
    module: "your_module_name", // Make sure this key exists in URL_CONST
  };
  req.data = {
    reportName: "LFAR_Report",
    // ... other data your backend expects
  };
  next();
};

// --- Main Route to Download and Save the File ---
app.post(
  "/submitLFARZipDownload",
  simulateRequestMiddleware, // Using the simulated middleware
  async (req, res) => {
    const filename = `${req.user.circleCode}_${req.user.quarterEndDate.replaceAll(
      "/",
      ""
    )}_${req.data.reportName}.zip`;

    console.log(`Preparing to download file: ${filename}`);

    try {
      // Step 1: Make the POST request to your backend service
      const springResponse = await fetch(BACKEND_URL, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          user: req.user,
          data: req.data,
        }),
      });

      // Handle cases where the backend returns an error
      if (!springResponse.ok) {
        const errorBody = await springResponse.text();
        console.error("Backend responded with an error:", errorBody);
        return res
          .status(springResponse.status)
          .send("Failed to fetch ZIP file from the backend service.");
      }

      // Prepare the file path and create the directory
      const zipsDirectory = path.join(__dirname, "zips");
      const filePath = path.join(zipsDirectory, filename);
      await fs.promises.mkdir(zipsDirectory, { recursive: true });

      // Step 2: Create a Transform stream to decode base64 chunks
      const base64Decoder = new Transform({
        transform(chunk, encoding, callback) {
          // This receives a chunk of the base64 string, decodes it into binary,
          // and passes it to the next stream.
          this.push(Buffer.from(chunk.toString(), "base64"));
          callback();
        },
      });

      console.log(`Streaming response to: ${filePath}`);

      // Step 3: Create the stream pipeline
      await pipeline(
        // 1. The body of the response from your backend
        springResponse.body,
        // 2. Parse the JSON stream to extract only the 'pdfContent' value
        JSONStream.parse("result.pdfContent"),
        // 3. Pipe the base64 string chunks through the decoder
        base64Decoder,
        // 4. Write the resulting binary chunks to the file
        createWriteStream(filePath)
      );

      console.log(`✅ File saved successfully at: ${filePath}`);
      res
        .status(200)
        .json({ message: "ZIP file saved successfully", path: filePath });
    } catch (err) {
      console.error("❌ An error occurred during the streaming process:", err);
      res.status(500).send("Failed to process the ZIP file.");
    }
  }
);

// --- Start the Server ---
app.listen(port, () => {
  console.log(`🚀 Server listening at http://localhost:${port}`);
});
