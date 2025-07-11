const express = require("express");
const morgan = require("morgan");
const fetch = require("node-fetch");
const fs = require("fs");
const path = require("path");
// const Base64Decode = require("base64-stream"); // <-- No longer needed
const { Readable } = require("stream");

router.post(
  "/submitLFARZipDownload",
  extractToken,
  DataValidator,
  LegalRequest,
  async (req, res) => {
    const filename = `${req.user.circleCode}_${req.user.quarterEndDate.replaceAll("/", "")}_${req.data.reportName}.zip`;
    console.log("Preparing to download:", filename);

    try {
      const springResponse = await fetch(
        URL_CONST.PROD_URL[req.user.module].submitDownload,
        {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            user: req.user,
            data: req.data,
          }),
        }
      );

      if (!springResponse.ok) {
        throw new Error(`API call failed with status: ${springResponse.status}`);
      }

      const data = await springResponse.json();
      const base64Zip = data.result.pdfContent;

      if (!base64Zip) {
        return res.status(404).json({ message: "No file content received from the API." });
      }

      // Clean the Base64 string and create a buffer
      const base64Data = base64Zip.replace(/^data:application\/zip;base64,/, "");
      const fileBuffer = Buffer.from(base64Data, "base64");

      const filePath = path.join(__dirname, "zips", filename);

      // Ensure directory exists and write the file asynchronously
      await fs.promises.mkdir(path.dirname(filePath), { recursive: true });
      await fs.promises.writeFile(filePath, fileBuffer);

      console.log("File saved successfully at:", filePath);
      res.status(200).json({ message: "ZIP file saved successfully", path: filePath });

    } catch (err) {
      console.error("Download process failed:", err);
      res.status(500).json({ message: "Failed to download and save the file." });
    }
  }
);
