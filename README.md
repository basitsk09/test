const express = require("express");
const morgan = require("morgan");
const fetch = require("node-fetch");
const fs = require("fs");
const path = require("path");
const Base64Decode = require("base64-stream");
const { Readable } = require("stream");

router.post(
  "/submitLFARZipDownload",
  extractToken,
  DataValidator,
  LegalRequest,
  async (req, res) => {
    const filename = `${req.user.circleCode}_${req.user.quarterEndDate.replaceAll("/", "")}_${req.data.reportName}.zip`;
    console.log(filename);
    console.log(req.data);

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

      // Check for a successful response from the Spring application
      if (!springResponse.ok) {
        throw new Error(`Failed to fetch from Spring service: ${springResponse.statusText}`);
      }

      const data = await springResponse.json();
      const base64Zip = data.result.pdfContent;

      if (!base64Zip) {
        return res.status(404).json({ message: "No ZIP content found in response" });
      }

      const base64Data = base64Zip.replace(/^data:application\/zip;base64,/, "");
      const filePath = path.join(__dirname, "zips", filename);

      // Ensure the 'zips' directory exists
      await fs.promises.mkdir(path.dirname(filePath), { recursive: true });

      const readableStream = Readable.from(base64Data);
      const decodeStream = new Base64Decode(); // Corrected instantiation
      const writeStream = fs.createWriteStream(filePath);

      // Pipe the streams together
      readableStream.pipe(decodeStream).pipe(writeStream);

      // Handle successful file write
      writeStream.on("finish", () => {
        console.log("ZIP file saved successfully at:", filePath);
        res.status(200).json({ message: "ZIP file saved successfully", path: filePath });
      });

      // Handle errors during the streaming process
      writeStream.on("error", (err) => {
        console.error("Error writing file:", err);
        res.status(500).json({ message: "Error saving the ZIP file" });
      });

      decodeStream.on("error", (err) => {
        console.error("Error decoding Base64 stream:", err);
        res.status(500).json({ message: "Error decoding the file content" });
      });

    } catch (err) {
      console.error("An error occurred:", err);
      res.status(500).send("Failed to process the download request.");
    }
  }
);
