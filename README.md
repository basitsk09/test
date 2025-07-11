const fs = require("fs");
const path = require("path");
const { Decoder } = require("base64-stream"); // OK
const { Readable } = require("stream");

router.post("/submitLFARZipDownload", async (req, res) => {
  const filename =
    req.user.circleCode +
    "_" +
    req.user.quarterEndDate.replaceAll("/", "") +
    "_" +
    req.data.reportName +
    ".zip";

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

    const base64Data = base64Zip.replace(
      /^data:application\/zip;base64,/,
      ""
    );

    const filePath = path.join(__dirname, "zips", filename);
    fs.mkdirSync(path.dirname(filePath), { recursive: true });

    const base64Stream = Readable.from(base64Data);
    const decodeStream = Decoder(); // ✅ NO `new`
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
});