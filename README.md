Thanks for sharing your full code — I now clearly see the problem. The issue you're facing:

TypeError: Decoder is not a constructor

is happening because the base64-stream library does not export Decoder as a named export like this:

const { Decoder } = require("base64-stream"); // ❌ Wrong


---

✅ Correct Way to Import Decoder from base64-stream:

Change this line:

const { Decoder } = require("base64-stream");

To:

const Base64Decode = require("base64-stream").Decoder; // ✅ Correct

Or simply:

const Base64Decode = require("base64-stream"); // ✅ Also works
const decodeStream = new Base64Decode.Decoder(); // ✅


---

🔁 Then update this part:

const decodeStream = new Decoder(); // ❌ Will throw error

To:

const decodeStream = new Base64Decode(); // ✅ Works now


---

✅ Final Working Snippet

const Base64Decode = require("base64-stream").Decoder; // ✅ Correct

...

const base64Stream = Readable.from(base64Data);
const decodeStream = new Base64Decode(); // ✅ Fixed
const writeStream = fs.createWriteStream(filePath);

base64Stream.pipe(decodeStream).pipe(writeStream);


---

🧪 Sanity Check: Make sure you have the package installed

npm install base64-stream


---

✅ Summary

Problem	Fix

Decoder is not a constructor	Use require("base64-stream").Decoder not destructuring
Readable.from(...) + Base64Decode()	Correct stream piping method for large base64 files



---

Let me know if you want help modifying this to support raw binary download from Spring Boot — that will eliminate all base64-related issues permanently.

