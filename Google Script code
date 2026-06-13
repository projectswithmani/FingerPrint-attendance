function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);
  sheet.appendRow([
    new Date(),
    data.id,
    data.name,
    data.status
  ]);
  return ContentService.createTextOutput("OK");
}

function doGet(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  sheet.appendRow([
    new Date(),
    e.parameter.id,
    e.parameter.name,
    e.parameter.status
  ]);
  return ContentService.createTextOutput("OK - Row Added");
}
