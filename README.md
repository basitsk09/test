// Get the full line where the drop is happening
const [lineBlot, offsetInLine] = quill.getLine(currentIndex);
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