const fs = require("fs");
const path = require("path");
const { Decoder } = require("base64-stream"); // ✅ Correct import
const { Readable } = require("stream");

router.post("/submitLFARZipDownload", async (req, res) => {
  try {
    const springResponse = await fetch(/* your fetch config */);
    const data = await springResponse.json();
    const base64Zip = data.result.pdfContent;

    // Clean Base64 prefix
    const cleanedBase64 = base64Zip.replace(/^data:application\/zip;base64,/, "");

    const filePath = path.join(__dirname, "zips", filename);

    // Ensure directory exists
    fs.mkdirSync(path.dirname(filePath), { recursive: true });

    // Create a readable stream from base64 string
    const base64Stream = Readable.from(cleanedBase64);
    const decodeStream = new Decoder(); // ✅ Now works
    const writeStream = fs.createWriteStream(filePath);

    base64Stream
      .pipe(decodeStream)
      .pipe(writeStream)
      .on("finish", () => {
        res.json({ message: "ZIP file saved successfully", path: filePath });
      })
      .on("error", (err) => {
        console.error("Write error:", err);
        res.status(500).json({ message: "Error writing file" });
      });

  } catch (err) {
    console.error(err);
    res.status(500).send("Failed to fetch ZIP file");
  }
});