import { useState, useEffect, useRef } from "react";
import {
  Snackbar,
  IconButton,
  Button,
  Divider,
  Dialog,
  DialogTitle,
  DialogContent,
  Paper,
  Typography,
  Grid,
  Card,
  CardHeader,
  CardContent,
  List,
  ListItem,
  ListItemIcon,
  ListItemText,
  Chip,
} from "@mui/material";
import { Box, styled } from "@mui/system";
import DialogActions from "@mui/material/DialogActions";
import CloseIcon from "@mui/icons-material/Close";
import EditIcon from "@mui/icons-material/Edit";
import PreviewIcon from "@mui/icons-material/Preview";
import RestartAltIcon from "@mui/icons-material/RestartAlt";
import SaveIcon from "@mui/icons-material/Save";
import CancelIcon from "@mui/icons-material/Cancel";
import CheckCircleOutlineIcon from "@mui/icons-material/CheckCircleOutline";
import ReactQuill from "react-quill";
import Quill from "quill";
import "react-quill/dist/quill.snow.css";
import "./quill.css";
import axios from "axios";
import { encrypt } from "../../Security/AES-GCM256";
import ImageResize from "quill-image-resize-module-react";

// 100 * 100 image size
const Clipboard = Quill.import("modules/clipboard");
class CustomClipboard extends Clipboard {
  async onPaste(e) {
    e.preventDefault();

    // Get plain text or HTML from clipboard
    const clipboardData = e.clipboardData || window.clipboardData;
    const text = clipboardData.getData("text/plain");
    const html = clipboardData.getData("text/html");

    if (html && html.includes("<img")) {
      // If there's an image, force width and height
      const modifiedHtml = html.replace(
        /<img([^>]*)>/g,
        '<img$1 width="100" height="100">'
      );
      this.quill.clipboard.dangerouslyPasteHTML(
        this.quill.getSelection().index,
        modifiedHtml
      );
    } else {
      super.onPaste(e);
    }
  }
}
Quill.register("modules/clipboard", CustomClipboard, true);

// --- Quill Blots and Registrations ---
const Embed = Quill.import("blots/embed");

class VariableBlot extends Embed {
  static create(value) {
    const node = super.create();
    if (!value || typeof value !== "string") value = "INVALID_VAR";
    node.setAttribute("contenteditable", "false");
    node.classList.add("variable-blot");
    const text = document.createElement("span");
    text.innerText = value;
    const closer = document.createElement("span");
    closer.innerText = "X";
    closer.classList.add("variable-delete-icon");
    node.appendChild(text);
    node.appendChild(closer);
    Object.assign(node.style, {
      position: "relative",
      backgroundColor: "#A9A9A9",
      color: "black",
      padding: "5px 20px 5px 8px",
      fontWeight: "500",
      margin: "0 4px",
      cursor: "default",
      userSelect: "none",
      display: "inline-block",
    });
    return node;
  }

  // --- START: Corrected Logic ---
  // This static method is now more defensive.
  static value(domNode) {
    try {
      const spans = domNode.querySelectorAll("span");
      const textSpan = spans?.[0];
      if (textSpan && textSpan.innerText.startsWith("{{")) {
        return textSpan.innerText.trim();
      }
    } catch (_) {}
    return "";
  }

  // The instance method correctly defers to the safe static method.
  value() {
    return this.statics.value(this.domNode);
  }
  // --- END: Corrected Logic ---
}
VariableBlot.blotName = "variable";
VariableBlot.tagName = "span";
Quill.register(VariableBlot);

const Size = Quill.import("formats/size");
Size.whitelist = [
  "2px",
  "4px",
  "6px",
  "8px",
  "10px",
  "12px",
  "14px",
  "16px",
  "18px",
  "20px",
  "22px",
  "24px",
  "26px",
];
Quill.register(Size, true);

const Font = Quill.import("formats/font");
Font.whitelist = [
  "arial",
  "times-new-roman",
  "courier-new",
  "sans-serif",
  "serif",
  "monospace",
  "georgia",
  "tahoma",
  "verdana",
  "sofia",
];
Quill.register(Font, true);

Quill.register("modules/imageResize", ImageResize);

const AlignStyle = Quill.import("attributors/style/align");
Quill.register(AlignStyle, true);
const BackgroundStyle = Quill.import("attributors/style/background");
Quill.register(BackgroundStyle, true);
const ColorStyle = Quill.import("attributors/style/color");
Quill.register(ColorStyle, true);
const DirectionStyle = Quill.import("attributors/style/direction");
Quill.register(DirectionStyle, true);
const FontStyle = Quill.import("attributors/style/font");
Quill.register(FontStyle, true);
const SizeStyle = Quill.import("attributors/style/size");
Quill.register(SizeStyle, true);

// --- Remove X from export helper ---
function removeVariableDeleteIcons(html) {
  const container = document.createElement("div");
  container.innerHTML = html;
  container.querySelectorAll(".variable-delete-icon").forEach((el) => {
    el.parentNode && el.parentNode.removeChild(el);
  });
  return container.innerHTML;
}

// --- Remove all styling and classes for backend ---
function stripStyling(html) {
  const container = document.createElement("div");
  container.innerHTML = html;
  const all = container.querySelectorAll("*");
  all.forEach((el) => {
    // el.removeAttribute("style");
    el.removeAttribute("class");
  });
  return container.innerHTML;
}

const iv = crypto.getRandomValues(new Uint8Array(12));
const ivBase64 = btoa(String.fromCharCode.apply(null, iv));
const salt = crypto.getRandomValues(new Uint8Array(16));
const saltBase64 = btoa(String.fromCharCode.apply(null, salt));

const EmailTemplate = () => {
  const [headerContent, setHeaderContent] = useState(
    "Non-editable header will appear here"
  );
  const [bodyContent, setBodyContent] = useState(
    "Editable body will appear here"
  );
  const [sbar, setSbar] = useState({ isOpen: false, Msg: "" });
  const [wordCount, setWordCount] = useState(0);
  const [isEditing, setIsEditing] = useState(false);
  const [isPreviewing, setIsPreviewing] = useState(false);
  const [templateContent, settemplateContent] = useState(null);

  const draftVar = [
    "{{DATE}}",
    "{{REF_NO}}",
    "{{FIRM_NAME}}",
    "{{FRN_NO}}",
    "{{FIRM_ADDR}}",
    "{{ASSIGNMENT_TYPE}}",
    "{{GSTN}}",
  ];

  const bodyEditorRef = useRef(null);
  const headerEditorRef = useRef(null);

  const modules = {
    toolbar: [
      [{ font: Font.whitelist }],
      [{ size: Size.whitelist }],
      [
        {
          color: [
            "red",
            "green",
            "blue",
            "black",
            "orange",
            "#ffffff",
            "#000000",
          ],
        },
        { background: ["red", "green", "blue", "yellow", "gray"] },
      ],
      [
        { align: [] },
        { align: "center" },
        { align: "right" },
        { align: "justify" },
      ],
      [{ script: "sub" }, { script: "super" }],
      [{ header: [1, 2, 3, false] }],
      [{ list: "ordered" }, { list: "bullet" }],
      ["bold", "italic", "underline", "strike"],
      ["link", "clean"],
      [{ indent: "-1" }, { indent: "+1" }],
    ],
    clipboard: { matchVisual: false },
  };

  const formats = [
    "header",
    "font",
    "size",
    "bold",
    "italic",
    "underline",
    "strike",
    "blockquote",
    "list",
    "bullet",
    "indent",
    "link",
    "image",
    "color",
    "background",
    "align",
    "script",
    "variable",
    "bb",
  ];

  // useEffect(() => {
  //   if (!isEditing) return;
  //   const quill = bodyEditorRef.current?.getEditor();
  //   if (!quill) return;
  //   const editorRoot = quill.root;

  //   const handleEditorDrop = (e) => {
  //     e.preventDefault();
  //     const text = e.dataTransfer.getData("text");
  //     if (!text) return;

  //     const range = quill.getSelection(true);
  //     if (!range) return;

  //     const currentIndex = range.index;

  //     if (currentIndex > 0) {
  //       // Use Quill's API to get the content of the single blot/char before the cursor
  //       const oneCharBefore = quill.getContents(currentIndex - 1, 1);
  //       const previousOp = oneCharBefore.ops?.[0]; // Safely access the first operation

  //       // Check if the previous operation was the same variable blot
  //       if (
  //         previousOp &&
  //         typeof previousOp.insert === "object" &&
  //         previousOp.insert.variable &&
  //         previousOp.insert.variable === text
  //       ) {
  //         setSbar({
  //           isOpen: true,
  //           Msg: `? Cannot add the same variable consecutively.`,
  //         });
  //         return;
  //       }
  //     }

  //     quill.insertEmbed(currentIndex, "variable", text, Quill.sources.USER);
  //     quill.setSelection(currentIndex + 1, Quill.sources.USER);
  //   };

  //   const handleEditorDragOver = (e) => {
  //     e.preventDefault();
  //   };

  //   editorRoot.addEventListener("drop", handleEditorDrop);
  //   editorRoot.addEventListener("dragover", handleEditorDragOver);

  //   return () => {
  //     editorRoot.removeEventListener("drop", handleEditorDrop);
  //     editorRoot.removeEventListener("dragover", handleEditorDragOver);
  //   };
  // }, [isEditing, setSbar]);
  useEffect(() => {
    if (!isEditing) return;
    const quill = bodyEditorRef.current?.getEditor();
    if (!quill) return;
    const editorRoot = quill.root;

    const handleEditorDrop = (e) => {
      e.preventDefault();
      const text = e.dataTransfer.getData("text/plain");
      if (!text || typeof text !== "string" || !text.startsWith("{{")) {
        setSbar({
          isOpen: true,
          Msg: "? Invalid or missing variable dropped.",
        });
        return;
      }

      // --- START: New logic to calculate the correct drop index ---
      let dropRange;
      let currentIndex;

      // Use browser APIs to find the text range from the mouse coordinates
      if (document.caretRangeFromPoint) {
        dropRange = document.caretRangeFromPoint(e.clientX, e.clientY);
      } else {
        // Fallback for Firefox
        const position = document.caretPositionFromPoint(e.clientX, e.clientY);
        dropRange = document.createRange();
        if (position) {
          dropRange.setStart(position.offsetNode, position.offset);
        }
      }

      // If we couldn't determine the range, fallback to the current selection
      if (!dropRange) {
        const fallbackRange = quill.getSelection(true);
        if (!fallbackRange) return;
        currentIndex = fallbackRange.index;
      } else {
        // Find the Quill blot (leaf) at that DOM range
        const leaf = Quill.find(dropRange.startContainer, true);
        if (!leaf) {
          // If no leaf found, use fallback
          const fallbackRange = quill.getSelection(true);
          if (!fallbackRange) return;
          currentIndex = fallbackRange.index;
        } else {
          // Calculate the precise Quill index from the blot and offset
          const blotIndex = quill.getIndex(leaf);
          currentIndex = blotIndex + dropRange.startOffset;
        }
      }
      // --- END: New logic ---

      // The rest of the logic is the same, but now uses the correct `currentIndex`
      //   if (currentIndex > 0) {
      //     const oneCharBefore = quill.getContents(currentIndex - 1, 1);
      //     const previousOp = oneCharBefore.ops?.[0];

      // if (
      //   previousOp &&
      //   typeof previousOp.insert === "object" &&
      //   previousOp.insert.variable &&
      //   previousOp.insert.variable === text
      // ) {
      //   setSbar({
      //     isOpen: true,
      //     Msg: `❌ Cannot add the same variable consecutively.`,
      //   });
      //   return;
      // }
      // Get the full line where the drop is happening
      const [lineBlot] = quill.getLine(currentIndex);
      if (lineBlot) {
        const lineStart = quill.getIndex(lineBlot);
        const lineLength = lineBlot.length();
        const lineContent = quill.getContents(lineStart, lineLength);

        // Check if the same variable already exists on this line
        const duplicateVariable = lineContent.ops?.some(
          (op) => typeof op.insert === "object" && op.insert.variable === text
        );

        if (duplicateVariable) {
          setSbar({
            isOpen: true,
            Msg: `❌ Cannot add the same variable more than once in a line.`,
          });
          return;
        }
      }
      //  }

      quill.insertEmbed(currentIndex, "variable", text, Quill.sources.USER);
      quill.setSelection(currentIndex + 1, Quill.sources.USER);
    };

    const handleEditorDragOver = (e) => {
      e.preventDefault();
    };

    editorRoot.addEventListener("drop", handleEditorDrop);
    editorRoot.addEventListener("dragover", handleEditorDragOver);

    return () => {
      editorRoot.removeEventListener("drop", handleEditorDrop);
      editorRoot.removeEventListener("dragover", handleEditorDragOver);
    };
  }, [isEditing, setSbar]);

  useEffect(() => {
    const editor = bodyEditorRef.current?.getEditor();
    if (!editor || !isEditing) return;

    const handleClick = (e) => {
      if (e.target.classList.contains("variable-delete-icon")) {
        const blotNode = e.target.closest(".variable-blot");
        if (blotNode) {
          const blot = Quill.find(blotNode, true);
          if (blot) {
            const index = editor.getIndex(blot);
            editor.deleteText(index, blot.length(), "user");
          }
        }
      }
    };

    editor.root.addEventListener("click", handleClick);
    return () => {
      editor.root.removeEventListener("click", handleClick);
    };
  }, [isEditing]);

  useEffect(() => {
    fetchData();
  }, []);

  // const fetchData = async () => {
  //   try {
  //     const response = await axios.post(
  //       "Server/IAMServices/EmailTemplate",
  //       {},
  //       {
  //         headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
  //       }
  //     );
  //     let res = response.data.result;
  //     console.log(res);
  //     let deliminator = "INTIMATION</h4>";
  //     const splitIndex = res.indexOf("INTIMATION</h4>") + deliminator.length;
  //     // console.log(splitIndex);
  //     // const splitIndex = res.indexOf("Dear Sir/Madam");
  //     if (splitIndex !== -1) {
  //       setHeaderContent(res.substring(0, splitIndex));
  //       setBodyContent(res.substring(splitIndex));
  //     } else {
  //       setHeaderContent(res);
  //       setBodyContent("");
  //     }
  //     setWordCount(countWord(res));
  //   } catch (e) {
  //     // Optionally fallback to handleReset
  //   }
  // };
  const fetchData = async () => {
    try {
      const response = await axios.post(
        "Server/IAMServices/EmailTemplate",
        {},
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      let htmlResponse = response.data.result;

      // Create a temporary, in-memory element to parse the HTML string
      const container = document.createElement("div");
      container.innerHTML = htmlResponse;

      // Find the h4 element that acts as our structural delimiter
      const delimiterNode = container.querySelector("h4");

      if (delimiterNode) {
        let headerHtml = "";
        let bodyHtml = "";
        let bodyHasStarted = false;

        // Loop through all top-level nodes in the response
        for (const node of container.childNodes) {
          if (bodyHasStarted) {
            // Once the flag is set, everything else goes into the body
            bodyHtml += node.outerHTML || node.textContent;
          } else {
            // Until the flag is set, everything goes into the header
            headerHtml += node.outerHTML || node.textContent;
          }

          // After processing the current node, check if it was our delimiter.
          // If so, all subsequent nodes belong to the body.
          if (node === delimiterNode) {
            bodyHasStarted = true;
          }
        }
        setHeaderContent(headerHtml);
        setBodyContent(bodyHtml);
      } else {
        // Fallback if an h4 tag is not found in the template
        console.error("Could not find the h4 delimiter in the template.");
        setHeaderContent(htmlResponse);
        setBodyContent("");
      }
    } catch (e) {
      console.error("Failed to fetch email template:", e);
    }
  };

  const handleDragStart = (e, label) => {
    e.dataTransfer.setData("text/plain", label);
  };
  function countWord(str) {
    const text = str.replace(/<[^>]*>/g, "");
    const words = text.trim().split(/\s+/).filter(Boolean);
    return words.length;
  }

  // function verifyLine(line) {
  //   let array1 = line?.match(/\{{2}[A-Z_]+\}{2}/g);
  //   let set1 = new Set(array1);
  //   if (array1?.length != set1?.length) {
  //     let pos = bodyEditorRef.current.editor.getSelection();
  //     bodyEditorRef.current.editor.deleteText(pos - 1, 1, "silent");
  //   }
  // }
  const handleBodyChange = (content, delta, source, editor) => {
    if (source == "api") {
      return;
    }
    console.log(delta);
    let htmlContent = editor.getHTML();
    console.log(htmlContent);
    let length = countWord(htmlContent);
    let lines = htmlContent.split(/[!?.]|(<p><br><\/p>)/gi);
    console.log(lines);
    // lines.forEach((line) => verifyLine(line));

    // Validation for special characters
    const insertedOp = delta.ops?.find((op) => typeof op.insert === "string");
    if (insertedOp) {
      const inserted = insertedOp.insert;
      const len = insertedOp.insert.length;

      // Block characters that are NOT allowed
      const illegalChar = inserted.match(/[^a-zA-Z0-9_\s.,;!?'"()\{\}\-@]/);

      if (illegalChar) {
        setTimeout(() => {
          const quillEditor = bodyEditorRef.current?.getEditor();
          if (quillEditor) {
            const selection = quillEditor.getSelection();
            let position = selection?.index || quillEditor.getLength();
            quillEditor.deleteText(position - len, len, "silent");
          }
        }, 10);
        setSbar({
          isOpen: true,
          Msg: "❌ Special characters are not allowed!",
        });
      }
    }
    if (insertedOp && insertedOp.insert === "\n") {
      const quillEditor = bodyEditorRef.current?.getEditor();
      if (quillEditor) {
        const selection = quillEditor.getSelection();
        let index = selection?.index || quillEditor.getLength();

        // Get text up to the cursor
        const text = quillEditor.getText(0, index);
        // If the last 4 characters are all newlines, block the 4th
        const match = text.match(/(\n{4,})$/);

        if (match) {
          setTimeout(() => {
            quillEditor.deleteText(index - 1, 1, "silent");
            quillEditor.setSelection(index - 1, 0, "silent"); // cursor stays where user tried to type
          }, 0);
          setSbar({
            isOpen: true,
            Msg: "❌ You cannot enter more than 3 consecutive blank lines.",
          });
          return;
        }
      }
    }

    // Word limit validation
    if (length > 2000) {
      const quillEditor = bodyEditorRef.current?.getEditor();
      if (quillEditor) {
        const selection = quillEditor.getSelection();
        let position = selection?.index || quillEditor.getLength();
        quillEditor.deleteText(position - 1, 1, "silent");
        setSbar({
          isOpen: true,
          Msg: "Word limit exceeded. Please reduce content.",
        });
        return;
      }
    }

    setBodyContent(htmlContent);
    setWordCount(length);
  };
  // ---- MAIN CHANGE: strip styling before sending to backend ----
  const handleSave = async () => {
    setIsEditing(false);
    let i = bodyContent.search(/<p><br><\/p>/g);
    let fullContent =
      i === 0
        ? headerContent + bodyContent
        : headerContent + "<p><br/></p>" + bodyContent;

    // Remove X icon for save/export
    let cleanContent = removeVariableDeleteIcons(fullContent);

    // STRIP STYLING for backend only
    let backendContent = stripStyling(cleanContent);

    let str = backendContent.match(/\{{2}[A-Z_]+\}{2}/g);
    let checked = str?.filter((w) => draftVar.some((s) => s === w));
    if (str != null && str.length > checked.length) {
      setSbar({
        isOpen: true,
        Msg: "Invalid Variable Names added! Ensure variables are correct without whitespaces.",
      });
      handleCloseDialog();
      return;
    }
    let SanStr = backendContent.replace(/[^\x00-\x7F]/g, "");
    let dataToSend = { MASTER_TEMPLATE: `<html>${SanStr}</html>` };
    console.log(dataToSend);

    try {
      let jsonFormData = JSON.stringify(dataToSend);
      await encrypt(iv, salt, jsonFormData).then(function (result) {
        jsonFormData = result;
      });
      let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };
      const response = await axios.post(
        "Server/IAMServices/saveMasterTemplate",
        payload,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      if (response.data.statusCode === 200 && response.data.result !== "") {
        setSbar({ isOpen: true, Msg: "✅ Template saved successfully!" });
      }
    } catch (error) {
      setSbar({ isOpen: true, Msg: "❌ Error saving template." });
    }
    setIsPreviewing(false);
    handleCloseDialog();
  };

  const handlePreview = async () => {
    let i = bodyContent.search(/<p><br><\/p>/g);
    let fullContent =
      i === 0
        ? headerContent + bodyContent
        : headerContent + "<p><br/></p>" + bodyContent;

    // Remove the delete "X" icons (but DO NOT remove styling for frontend preview)
    const cleanContent = removeVariableDeleteIcons(fullContent);

    let data = {
      DATE: "xx/xx/xxxx",
      REF_NO: "9999xxx",
      FIRM_NAME: "xxxxxxxxxx",
      FRN_NO: "999999999",
      GSTN: "99xx99xx",
      FIRM_ADDR: "xxxxxxxx,xxxxxxxx",
      ASSIGNMENT_TYPE: "xxxxxxx",
    };
    const tempDiv = document.createElement("div");
    tempDiv.innerHTML = cleanContent;
    const variableBlots = tempDiv.querySelectorAll(".variable-blot");
    variableBlots.forEach((blot) => {
      const variableName = blot.textContent.replace("X", "").trim();
      const key = variableName.substring(2, variableName.length - 2);
      if (data[key]) {
        blot.replaceWith(document.createTextNode(data[key]));
      }
    });
    let StrToSend = tempDiv.innerHTML.replace(/<br\s*\/?>/gi, "<br></br>");
    settemplateContent(StrToSend);
    setIsPreviewing(true);
  };

  const handleCloseDialog = () => setIsPreviewing(false);

  const handleReset = async () => {
    try {
      const response = await axios.post(
        "Server/IAMServices/resetTemplate",
        {},
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );
      let res = response.data.result;
      const splitIndex = res.indexOf("Dear Sir/Madam");
      if (splitIndex !== -1) {
        setHeaderContent(res.substring(0, splitIndex));
        setBodyContent(res.substring(splitIndex));
      } else {
        setHeaderContent(res);
        setBodyContent("");
      }
      setSbar({ isOpen: true, Msg: "✅ Template has been reset to default." });
    } catch (e) {
      setSbar({ isOpen: true, Msg: "❌ Error resetting template." });
    }
  };

  const handleHeaderCopy = (e) => {
    e.preventDefault();
    setSbar({
      isOpen: true,
      Msg: "❌ Copying from the header section is not allowed!",
    });
  };

  return (
    <>
      <Typography gutterBottom variant="h5" component="div" sx={{ mb: 1 }}>
        Empanelment Intimation Letter Template
      </Typography>
      <Divider sx={{ mb: 1 }} />
      <Box sx={{ p: 1, minHeight: "80%" }}>
        <Grid container spacing={3}>
          <Grid item xs={12} md={4}>
            <Card sx={{ mb: 3 }}>
              <CardHeader
                title="Variables"
                titleTypographyProps={{ variant: "h6" }}
              />
              <CardContent sx={{ display: "flex", flexWrap: "wrap", gap: 1 }}>
                {draftVar.map((label) => (
                  <Chip
                    key={label}
                    label={label}
                    color="primary"
                    draggable={isEditing}
                    onDragStart={(e) => handleDragStart(e, label)}
                    sx={{
                      borderRadius: "0",
                      cursor: isEditing ? "move" : "not-allowed",
                      fontWeight: "medium",
                    }}
                  />
                ))}
              </CardContent>
            </Card>
            <Card>
              <CardHeader
                title="Instructions"
                titleTypographyProps={{ variant: "h6" }}
              />
              <CardContent>
                <List dense>
                  {[
                    "Special characters are not allowed.",
                    "Only use variables from the list above.",
                    "Header section is non-editable.",
                    "Click 'Edit' to modify the template body.",
                    "Maximum word limit is 2000 words.",
                  ].map((text) => (
                    <ListItem key={text} disablePadding>
                      <ListItemIcon sx={{ minWidth: 32 }}>
                        <CheckCircleOutlineIcon
                          color="primary"
                          fontSize="small"
                        />
                      </ListItemIcon>
                      <ListItemText primary={text} />
                    </ListItem>
                  ))}
                </List>
              </CardContent>
            </Card>
          </Grid>
          <Grid item xs={12} md={8}>
            <Card>
              <CardHeader
                title="Empanelment Intimation Letter Draft"
                titleTypographyProps={{ variant: "h6" }}
              />
              <CardContent>
                <Typography
                  variant="subtitle1"
                  sx={{ fontWeight: "medium", color: "text.secondary" }}
                >
                  Non Configurable
                </Typography>
                <Paper
                  variant="outlined"
                  sx={{
                    bgcolor: "#f5f5f5",
                    p: 1,
                    mb: 3,
                    border: "1px solid #e0e0e0",
                    borderRadius: 1,
                  }}
                  onCopy={handleHeaderCopy}
                >
                  <ReactQuill
                    style={{ height: "23vh" }}
                    theme="bubble"
                    value={headerContent}
                    ref={headerEditorRef}
                    readOnly={true}
                    modules={{ toolbar: false }}
                  />
                </Paper>
                <Typography
                  variant="subtitle1"
                  sx={{ fontWeight: "medium", color: "text.secondary" }}
                >
                  Configurable
                </Typography>
                <Paper variant="outlined">
                  <ReactQuill
                    theme="snow"
                    style={{ height: "23vh", marginBottom: "4%" }}
                    modules={modules}
                    formats={formats}
                    value={bodyContent}
                    ref={bodyEditorRef}
                    onChange={handleBodyChange}
                    readOnly={!isEditing}
                  />

                  <Box
                    sx={{
                      display: "flex",
                      justifyContent: "space-between",
                      alignItems: "center",
                      mt: 2,
                      mb: 1,
                    }}
                  >
                    <Typography variant="body2" color="textSecondary">
                      Word Count: {wordCount} / 2000
                    </Typography>
                    <Box sx={{ display: "flex", gap: 1 }}>
                      {!isEditing ? (
                        <Button
                          variant="contained"
                          startIcon={<EditIcon />}
                          onClick={() => setIsEditing(true)}
                        >
                          Edit
                        </Button>
                      ) : (
                        <Button
                          variant="outlined"
                          color="error"
                          startIcon={<CancelIcon />}
                          onClick={() => setIsEditing(false)}
                        >
                          Cancel
                        </Button>
                      )}
                      <Button
                        variant="outlined"
                        color="info"
                        startIcon={<PreviewIcon />}
                        onClick={handlePreview}
                      >
                        Preview
                      </Button>
                      <Button
                        variant="outlined"
                        color="secondary"
                        startIcon={<RestartAltIcon />}
                        onClick={handleReset}
                      >
                        Reset
                      </Button>
                    </Box>
                  </Box>
                </Paper>
              </CardContent>
            </Card>
          </Grid>
        </Grid>
        <Dialogue
          open={isPreviewing}
          close={handleCloseDialog}
          save={handleSave}
          content={templateContent}
        />
        <Snackbar
          open={sbar.isOpen}
          autoHideDuration={6000}
          onClose={() => setSbar({ isOpen: false, Msg: "" })}
          message={sbar.Msg}
          anchorOrigin={{ vertical: "bottom", horizontal: "right" }}
        />
      </Box>
    </>
  );
};

// --- Preview Dialog Component as separate function ---
const BootstrapDialog = styled(Dialog)(({ theme }) => ({
  "& .MuiDialogContent-root": {
    padding: theme.spacing(2),
    backgroundColor: "#f4f6f8",
  },
  "& .MuiDialogActions-root": {
    padding: theme.spacing(1.5),
    borderTop: `1px solid ${theme.palette.divider}`,
  },
  "& .MuiDialogTitle-root": {
    borderBottom: `1px solid ${theme.palette.divider}`,
  },
}));

const Dialogue = ({ open, close, save, content }) => (
  <BootstrapDialog onClose={close} open={open} fullWidth maxWidth="md">
    <DialogTitle sx={{ m: 0, p: 2, fontWeight: "bold" }}>
      Intimation Letter Draft Preview
    </DialogTitle>
    <IconButton
      aria-label="close"
      onClick={close}
      sx={{
        position: "absolute",
        right: 8,
        top: 8,
        color: (theme) => theme.palette.grey[500],
      }}
    >
      <CloseIcon />
    </IconButton>
    <DialogContent>
      <Paper
        elevation={3}
        dangerouslySetInnerHTML={{ __html: content }}
        sx={{
          width: "210mm",
          minHeight: "297mm",
          margin: "2rem auto",
          padding: "20mm",
          boxSizing: "border-box",
          backgroundColor: "#ffffff",
          overflow: "auto",
          overflowWrap: "break-word",
        }}
      />
    </DialogContent>
    <DialogActions>
      <Button onClick={close} color="secondary">
        Cancel
      </Button>
      <Button
        autoFocus
        onClick={save}
        variant="contained"
        startIcon={<SaveIcon />}
      >
        Save Draft
      </Button>
    </DialogActions>
  </BootstrapDialog>
);

export default EmailTemplate;
