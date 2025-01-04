# Chrome-Extension-for-Automatic-Invoice-Retrieval-from-Shein
Chrome extension that automatically saves invoices from Shein purchases. The extension should seamlessly integrate with the Shein platform and efficiently capture and store invoice details for easy access. Strong knowledge of Chrome API, web scraping, and user authentication is essential. If you have experience in developing similar extensions, we would love to hear from you!

We will give you access to our shein account so you can test it out.

The video below explains it in more details it's really short 2 minutes. Thank you for your time.
-----------------
To create a Chrome extension that automatically saves invoices from Shein purchases, you’ll need to use JavaScript, Chrome Extensions APIs, and web scraping techniques to extract invoice details from the Shein website. The extension will listen for relevant Shein purchase pages, extract the necessary invoice information, and store the details either in local storage or a cloud service like Firebase.

Here’s an outline of how to implement the Chrome extension for saving invoices from Shein:
1. Create Chrome Extension Structure

Your Chrome extension will need a few key components:

    Manifest file (manifest.json): This is the metadata and configuration file for your extension.
    Background Script: For handling the main logic of when the extension is triggered.
    Content Script: For interacting with Shein’s web pages and extracting invoice details.
    Popup (optional): A user interface for managing saved invoices.

2. Setting up the Manifest File (manifest.json)

Create a manifest.json file to configure your extension:

{
  "manifest_version": 3,
  "name": "Shein Invoice Saver",
  "version": "1.0",
  "description": "Automatically saves invoices from Shein purchases",
  "permissions": [
    "activeTab",
    "storage",
    "identity",
    "tabs"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.shein.com/*"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "host_permissions": [
    "https://www.shein.com/*"
  ],
  "oauth2": {
    "client_id": "YOUR_CLIENT_ID",
    "scopes": ["email"]
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}

This manifest defines:

    The permissions your extension needs (activeTab, storage, identity, etc.).
    A background service worker (background.js) that listens for extension events.
    A content script (content.js) that runs on Shein pages to scrape invoice data.
    OAuth2 credentials for user authentication if you need to store the data in the cloud (e.g., Firebase or Google Drive).

3. Background Script (background.js)

The background script will handle the user authentication (optional) and store invoices locally or send them to a cloud service.

chrome.runtime.onInstalled.addListener(() => {
  console.log('Shein Invoice Saver extension installed');
});

// Store invoice data in local storage
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'saveInvoice') {
    let invoiceData = message.data;
    
    // You can either store this in LocalStorage or send it to a cloud service
    chrome.storage.local.get({ invoices: [] }, function (result) {
      let invoices = result.invoices;
      invoices.push(invoiceData);
      chrome.storage.local.set({ invoices: invoices }, function () {
        console.log('Invoice saved!');
      });
    });

    sendResponse({ status: 'success' });
  }
});

4. Content Script (content.js)

The content script will be responsible for scraping the invoice details from the Shein order page. This script runs on every Shein page, extracting relevant information like order number, product names, prices, and the total amount.

For this, you need to inspect Shein's order confirmation page (or invoice page) and determine which elements contain the relevant data.

// content.js
function extractInvoiceData() {
  const orderNumber = document.querySelector('.order-number-selector')?.textContent;
  const productNames = Array.from(document.querySelectorAll('.product-name-selector')).map(el => el.textContent);
  const productPrices = Array.from(document.querySelectorAll('.product-price-selector')).map(el => el.textContent);
  const totalAmount = document.querySelector('.total-amount-selector')?.textContent;

  if (orderNumber && productNames.length > 0 && totalAmount) {
    const invoiceData = {
      orderNumber,
      products: productNames.map((name, index) => ({
        name,
        price: productPrices[index]
      })),
      totalAmount
    };

    // Send extracted data to background script for saving
    chrome.runtime.sendMessage({
      action: 'saveInvoice',
      data: invoiceData
    }, (response) => {
      if (response.status === 'success') {
        alert('Invoice saved!');
      }
    });
  }
}

// Trigger invoice extraction when on an order page
if (window.location.href.includes('/order')) {
  extractInvoiceData();
}

5. Popup Interface (popup.html and popup.js)

Create a simple popup for displaying saved invoices. In popup.html, create a basic interface for users to see their stored invoices:

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Invoices</title>
</head>
<body>
  <h1>Your Invoices</h1>
  <ul id="invoice-list"></ul>
  <script src="popup.js"></script>
</body>
</html>

In popup.js, load and display saved invoices:

// popup.js
chrome.storage.local.get({ invoices: [] }, function (result) {
  const invoiceList = document.getElementById('invoice-list');
  result.invoices.forEach(invoice => {
    const listItem = document.createElement('li');
    listItem.textContent = `Order: ${invoice.orderNumber} - Total: ${invoice.totalAmount}`;
    invoiceList.appendChild(listItem);
  });
});

6. Handling OAuth and User Authentication (Optional)

If you want to store invoices in a cloud service like Firebase or Google Drive, you’ll need to handle user authentication. Add the OAuth flow in the background.js or as a separate module to facilitate login and storing data.

For example, you can use Firebase Authentication to allow users to sign in, and then save invoices to Firebase Firestore.
7. Testing Your Extension

    Go to chrome://extensions/ in your browser.
    Enable Developer mode.
    Click Load unpacked and select your extension’s folder.
    Open Shein and navigate to an order/invoice page.
    The extension will automatically scrape the data and save it to local storage.

8. Future Improvements

    Cloud Storage: Implement cloud storage for storing invoices across devices.
    UI Enhancements: Improve the popup with better styling or even a table to view detailed invoice information.
    Invoice Download: Allow users to download invoices as PDF or CSV files.
    Invoice Filtering/Search: Add search and filter options for users to quickly find invoices.

Important Notes

    Testing: Thoroughly test on Shein’s website and ensure the selectors used in content.js are correct for your region and account type.
    Compliance: Ensure that your extension does not violate Shein's terms of service and that you have the necessary permissions to interact with their site.
    Security: Safely store authentication tokens and handle any sensitive data with caution.

This extension will automate the task of saving invoices from Shein purchases, improving the experience for property managers or anyone who frequently makes purchases on Shein.
