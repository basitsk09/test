/**
 * @Author:V1015698
 * Common Download function
 * returns downloaded file with respect to that format type*/
// downloadFile.jsx
export default function downloadFile(response, format, name) {
  let fileType = "";
  let fileExtension = "";
  console.log("Response",  format, name)
  // Set the MIME type and file extension based on the format
  switch (format) {
    case "pdf":
      fileType = "application/pdf";
      fileExtension = "pdf";
      break;
    case "excel":
      fileType =
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
      fileExtension = "xlsx";
      break;
    case "zip":
      fileType = "application/zip";
      fileExtension = "zip";
      break;
    case "word":
      fileType =
        "application/vnd.openxmlformats-officedocument.wordprocessingml.document";
      fileExtension = "docx";
      break;
    default:
      console.error("Unsupported file format");
      return;
  }

  /**
   * Converting base64 String data to byte array.*/
  // const byteCharecters = atob(response);
  // const byteNumbers = new Array(byteCharecters.length);
  // for (let i = 0; i < byteCharecters.length; i++) {
  //   byteNumbers[i] = byteCharecters.charCodeAt(i);
  // }
  const byteArray = Uint8Array.from(atob(response), c=> c.charCodeAt(0));//new Uint8Array(byteNumbers);

  /**
   * Converting byte array to URL using Blob*/
  const blob = new Blob([byteArray], { type: fileType });
  console.log(blob);
  const url = URL.createObjectURL(blob);
  console.log(url);

  // Trigger download by creating a temporary anchor tag
  const link = document.createElement("a");
  link.href = url;
  link.download = `${name}.${fileExtension}`;
  console.log(link.download);

  document.body.appendChild(link);

  link.click();

  // Clean up
  link.remove();
  window.URL.revokeObjectURL(url);
}
