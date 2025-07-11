const fs = require("fs");
const path = require("path");
const base64 = require("base64-stream");

router.post("/submitLFARZipDownload", async (req, res) => {
  try {
    const springResponse = await fetch(/* your fetch config */);
    const data = await springResponse.json();
    const base64Zip = data.result.pdfContent;

    const filePath = path.join(__dirname, "zips", filename);

    // Remove Base64 header if present
    const cleanedBase64 = base64Zip.replace(/^data:application\/zip;base64,/, "");

    // Ensure the directory exists
    fs.mkdirSync(path.dirname(filePath), { recursive: true });

    // Create streams
    const base64Stream = require('stream').Readable.from(cleanedBase64);
    const decodeStream = new base64.Decoder();
    const writeStream = fs.createWriteStream(filePath);

    // Pipe the Base64 stream through decoder into file
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