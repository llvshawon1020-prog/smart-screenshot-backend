const express = require("express");
const multer = require("multer");
const fs = require("fs");
const vision = require("@google-cloud/vision");
const Fuse = require("fuse.js");
const path = require("path");
const app = express();
const upload = multer({ dest: "uploads/" });
const client = new vision.ImageAnnotatorClient({ keyFilename: "vision-key.json" });

const products = [
  { code: "EZ123", name: "Basic T-Shirt", color: "Sky Blue", url: "/product/EZ123" },
  { code: "EZ124", name: "Basic T-Shirt", color: "Red", url: "/product/EZ124" },
  { code: "SHO900", name: "Sneaker X", color: "Black", url: "/product/SHO900" }
];

const fuse = new Fuse(products, { keys: ["code", "name", "color"], threshold: 0.3 });

function parseText(text) {
  const clean = text.replace(/\n/g, " ").replace(/[:|,]/g, " ");
  const code = clean.match(/(?:code|sku|item)\s*[-:#]?\s*([A-Z0-9-]{3,})/i)?.[1] || null;
  const color = clean.match(/(?:color|colour)\s*[-:#]?\s*([A-Za-z ]{3,30})/i)?.[1] || null;
  return { code, color, raw: text };
}

app.post("/api/upload", upload.single("screenshot"), async (req, res) => {
  try {
    const [result] = await client.textDetection(req.file.path);
    const text = result.textAnnotations?.[0]?.description || "";
    const parsed = parseText(text);
    const found = parsed.code ? fuse.search(parsed.code)[0]?.item : fuse.search(parsed.raw)[0]?.item;
    fs.unlink(req.file.path, () => {});
    res.json({ parsed, found });
  } catch (err) {
    res.status(500).json({ error: "Vision OCR failed", detail: err.message });
  }
});

app.listen(4000, () => console.log("Backend running"));
