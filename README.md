const express = require("express");
const fs = require("fs");
const axios = require("axios");
const cors = require("cors");

const app = express();
app.use(express.json());
app.use(cors());
app.use(express.static("public"));

const DISCORD_WEBHOOK = "PASTE_YOUR_WEBHOOK_HERE";

app.post("/claim", async (req, res) => {
  const { discord, wallet, twitter } = req.body;

  if (!discord || !wallet) {
    return res.status(400).json({ message: "Missing required fields" });
  }

  const newClaim = {
    discord,
    wallet,
    twitter,
    date: new Date()
  };

  // Save to JSON file
  const data = JSON.parse(fs.readFileSync("claims.json"));
  data.push(newClaim);
  fs.writeFileSync("claims.json", JSON.stringify(data, null, 2));

  // Send to Discord
  try {
    await axios.post(DISCORD_WEBHOOK, {
      content: `🟢 **New OG Claim**
Discord: ${discord}
Wallet: ${wallet}
Twitter: ${twitter || "N/A"}`
    });
  } catch (err) {
    console.log("Webhook error");
  }

  res.json({ message: "Claim submitted successfully!" });
});

app.listen(3000, () => console.log("Server running on port 3000"));
