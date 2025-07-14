const { pipeline } = require("stream/promises");
const { createWriteStream } = require("fs");
const { Readable, Transform } = require("stream");
const JSONStream = require("JSONStream");

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
        const errorBody = await springResponse.text();
        console.error("Backend error:", errorBody);
        return res
          .status(springResponse.status)
          .send("Failed to fetch ZIP file from backend.");
      }

      const filePath = path.join(__dirname, "zips", filename);
      await fs.promises.mkdir(path.dirname(filePath), { recursive: true });
      
      // Transform stream to decode base64 chunks
      const base64Decoder = new Transform({
        transform(chunk, encoding, callback) {
          this.push(Buffer.from(chunk.toString(), "base64"));
          callback();
        },
      });

      // Build the pipeline
      await pipeline(
        // 1. Get the readable stream from the fetch response
        springResponse.body,
        // 2. Parse the stream to extract the 'pdfContent' value
        JSONStream.parse("result.pdfContent"),
        // 3. Decode the base64 chunks
        base64Decoder,
        // 4. Write the binary chunks to a file
        createWriteStream(filePath)
      );

      console.log("File saved successfully at:", filePath);
      res
        .status(200)
        .json({ message: "ZIP file saved successfully", path: filePath });
    } catch (err) {
      console.error(err);
      res.status(500).send("Failed to process ZIP file");
    }
  }
);
