let express = require("express");
let morgan = require("morgan");
let fetch = require("node-fetch");
const fs = require("fs");
const path = require("path");
//const Base64Decode = require("base64-stream");
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

        //const filePath = path.join(__dirname, "zips", filename);
        //const buffer = Buffer.from(base64Data, "base64");

        // Make sure the directory exists
        //  fs.mkdirSync(path.dirname(filePath), { recursive: true });

        // fs.mkdirSync(path.dirname(filePath), { recursive: true });

        const fileBuffer = Buffer.from(base64Data, "base64");

        const filePath = path.join(__dirname, "zips", filename);

        await fs.promises.mkdir(path.dirname(filePath), { recursive: true });
        await fs.promises.writeFile(filePath, fileBuffer);

        console.log("File saved successfully at:", filePath);
        res
          .status(200)
          .json({ message: "ZIP file saved successfully", path: filePath });
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
