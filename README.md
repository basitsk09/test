Thanks for sharing the screenshot. The error:

TypeError: Cannot read properties of undefined (reading 'length')

is thrown during this stack trace:

at Delta.insert
at Array.reduce
at Block.delta
at Editor.insertEmbed

This strongly points to this line in your code:

quill.insertEmbed(currentIndex, "variable", text, Quill.sources.USER);

So the issue is likely because text (which is the variable you're trying to insert) is undefined.


---

✅ Fix: Defensive Check Before insertEmbed

In your handleEditorDrop, update this block:

const text = e.dataTransfer.getData("text");
if (!text) return;

To something more strict:

const text = e.dataTransfer.getData("text/plain");
if (!text || typeof text !== "string" || !text.startsWith("{{")) {
  setSbar({
    isOpen: true,
    Msg: "❌ Invalid or missing variable dropped.",
  });
  return;
}

Also make sure your draggable chips always provide valid data, like:

<Chip
  ...
  draggable={isEditing}
  onDragStart={(e) => e.dataTransfer.setData("text/plain", label)}
/>


---

✅ Additionally, check this:

Make sure your VariableBlot.create(value) method handles undefined values safely:

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
  ...
}


---

✅ Summary

🔍 Issue	✅ Fix

text is possibly undefined	Add strict checks before insertEmbed
Chip drag is possibly broken	Use e.dataTransfer.setData("text/plain", label)
VariableBlot.create() assumes valid input	Add fallback/default value



---

Let me know if you'd like a quick patch for all these in your current code.

