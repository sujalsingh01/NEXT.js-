// pages/api/compareAddresses.js
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);

export default async function handler(req, res) {
  if (req.method !== "POST") {
    return res.status(405).json({ error: "Method Not Allowed" });
  }

  try {
    const { address1, address2 } = req.body;

    if (!address1 || !address2) {
      return res.status(400).json({ error: "Both addresses are required" });
    }

    // Construct a prompt for AI
    const prompt = `Compare the following two addresses and determine if they refer to the same place. Provide a JSON output with "match": true or false, and a confidence score from 0 to 100.
    
    Address 1: ${address1}
    Address 2: ${address2}
    
    Respond in JSON format only.`;

    // Generate a response from Gemini
    const model = genAI.getGenerativeModel({ model: "gemini-pro" });
    const response = await model.generateContent(prompt);
    const text = response.response.text();

    // Parse AI-generated JSON response
    const result = JSON.parse(text);

    return res.status(200).json(result);
  } catch (error) {
    return res.status(500).json({ error: "Internal Server Error", details: error.message });
  }
}
