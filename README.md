
let express = require("express");
let morgan = require("morgan");
let fetch = require("node-fetch");
const fs = require("fs");
const path = require("path");
const Base64Decode = require("base64-stream"); // ? Correct
const { Readable } = require("stream");



router.post(
  "/submitLFARZipDownload",
  extractToken,
  DataValidator,
  LegalRequest,
  async (req, res) => {
    let filename =
      req.user.circleCode +
      "_" +
      req.user.quarterEndDate.replaceAll("/", "") +
      "_" +
      req.data.reportName +
      ".zip";
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

      const data = await springResponse.json();
      let base64Zip = data.result.pdfContent;

      try {
        const base64Data = base64Zip.replace(
          /^data:application\/zip;base64,/,
          ""
        );

        const filePath = path.join(__dirname, "zips", filename);
        //const buffer = Buffer.from(base64Data, "base64");

        // Make sure the directory exists
        //  fs.mkdirSync(path.dirname(filePath), { recursive: true });

        fs.mkdirSync(path.dirname(filePath), { recursive: true });

        const base64Stream = Readable.from(base64Data);
        const decodeStream = new Base64Decode.Decoder();
        const writeStream = fs.createWriteStream(filePath);

        base64Stream.pipe(decodeStream).pipe(writeStream);

        writeStream.on("finish", () => {
          res.json({ message: "ZIP file saved successfully", path: filePath });
        });

        writeStream.on("error", (err) => {
          console.error("Write error:", err);
          res.status(500).json({ message: "Error writing file" });
        });
      } catch (err) {
        console.error(err);
        res.status(500).send("Failed to fetch ZIP file");
      }
    } catch (err) {
      console.error(err);
      res.status(500).send("Failed to fetch ZIP file");
    }
  }
);

/////////////////////////////
TypeError: Base64Decode.Decoder is not a constructor
    at D:\@NewCrsFrontend\Server\routes\DownloadReportService\DownloadReports.js:316:30
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
POST /Server/downloadReports/submitLFARZipDownload 500 99170.823 ms - 24


