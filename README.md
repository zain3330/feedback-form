# Google Sheet Script 

const DATA_ENTRY_SHEET_NAME = "survey";
const TIME_STAMP_COLUMN_NAME = "Timestamp"; 
const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(DATA_ENTRY_SHEET_NAME);
const headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];

function doPost(e) {
  const lock = LockService.getScriptLock();
  if (!lock.tryLock(500)) { // Attempt to acquire lock faster
    return ContentService.createTextOutput(
      JSON.stringify({ result: 'error', error: 'Lock timeout' })
    ).setMimeType(ContentService.MimeType.JSON);
  }

  try {
    // Parse submitted data and append it to the sheet
    const data = parseFormData(e.postData.contents);
    data[TIME_STAMP_COLUMN_NAME] = new Date();
    appendToGoogleSheet(data);

    // Return success response
    return ContentService.createTextOutput(
      JSON.stringify({ result: 'success', data: data })
    ).setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    // Handle error response
    return ContentService.createTextOutput(
      JSON.stringify({ result: 'error', error: error.toString() })
    ).setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

function parseFormData(postData) {
  const data = {};
  postData.split('&').forEach(param => {
    const [key, value] = param.split('=');
    data[key] = decodeURIComponent(value.replace(/\+/g, ' '));
  });
  return data;
}

function appendToGoogleSheet(data) {
  const rowData = headers.map(header => data[header] || '');
  sheet.appendRow(rowData); // Directly append data in a single call
}

