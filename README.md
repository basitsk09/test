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