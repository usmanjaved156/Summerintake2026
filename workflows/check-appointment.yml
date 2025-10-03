const { chromium } = require('playwright');
const nodemailer = require('nodemailer');
const crypto = require('crypto');
const fs = require('fs');

const TARGET_URL = process.env.TARGET_URL;
const HASH_FILE = 'previous-hash.txt';

async function sendNotification(subject, message) {
  const transporter = nodemailer.createTransport({
    host: process.env.SMTP_HOST,
    port: parseInt(process.env.SMTP_PORT),
    secure: false,
    auth: {
      user: process.env.SMTP_USER,
      pass: process.env.SMTP_PASS,
    },
  });

  await transporter.sendMail({
    from: process.env.SMTP_USER,
    to: process.env.NOTIFICATION_EMAIL,
    subject: subject,
    html: message,
  });
}

async function checkAppointment() {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  
  try {
    console.log('Navigating to:', TARGET_URL);
    await page.goto(TARGET_URL, { waitUntil: 'networkidle' });
    
    // Wait for page to load
    await page.waitForTimeout(3000);
    
    // Get page content
    const content = await page.content();
    
    // Calculate hash
    const currentHash = crypto
      .createHash('md5')
      .update(content)
      .digest('hex');
    
    console.log('Current hash:', currentHash);
    
    // Check for Summer Intake 2026
    const hasSummerIntake = content
      .toLowerCase()
      .includes('summer intake 2026');
    
    // Read previous hash
    let previousHash = '';
    if (fs.existsSync(HASH_FILE)) {
      previousHash = fs.readFileSync(HASH_FILE, 'utf8').trim();
    }
    
    console.log('Previous hash:', previousHash);
    console.log('Has Summer Intake 2026:', hasSummerIntake);
    
    // Check if changed
    const hasChanged = previousHash && currentHash !== previousHash;
    
    if (hasSummerIntake || hasChanged) {
      let message = '<h2>Visa Appointment Alert!</h2>';
      
      if (hasSummerIntake) {
        message += '<p><strong>🎉 Summer Intake 2026 is now available!</strong></p>';
      }
      
      if (hasChanged) {
        message += '<p>The page content has changed.</p>';
      }
      
      message += `<p>Check the page: <a href="${TARGET_URL}">${TARGET_URL}</a></p>`;
      message += `<p>Time: ${new Date().toISOString()}</p>`;
      
      await sendNotification('🚨 Visa Appointment Alert', message);
      
      // Save hash
      fs.writeFileSync(HASH_FILE, currentHash);
      
      console.log('Alert sent! Test case FAILED (appointment found)');
      process.exit(1); // Fail the workflow
    } else {
      console.log('No changes detected. Test case PASSED.');
      fs.writeFileSync(HASH_FILE, currentHash);
      process.exit(0);
    }
    
  } catch (error) {
    console.error('Error:', error);
    await sendNotification(
      '❌ Monitoring Error',
      `<p>Error checking appointment page:</p><pre>${error.message}</pre>`
    );
    process.exit(1);
  } finally {
    await browser.close();
  }
}

checkAppointment();
