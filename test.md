To add formulas and create a complete Orchestration Flow in OpenText Extreme Cloud Native, I’ll walk you through where to click and how to configure everything step-by-step, including adding JavaScript formulas within the workflow tasks. Here’s a detailed click-by-click guide to help you set this up.

Step-by-Step Guide to Create the Orchestration Flow with Formulas

Step 1: Log in to OpenText Extreme Cloud Native

	1.	Open the OpenText Extreme Cloud Native portal in your browser.
	2.	Log in with your administrator or workflow designer account.

Step 2: Navigate to the Orchestration Module

	1.	In the dashboard, click on “Orchestration” from the menu.
	2.	Select “New Orchestration Flow” to create a new flow.

Step 3: Configure the Orchestration Flow

	1.	Name your flow, e.g., Multi-File Processor.
	2.	Trigger Configuration:
	•	Select “File Upload Trigger”.
	•	Set the monitored folder path to /incoming-files/.
	•	Allow multiple file types (e.g., .json, .csv, .log, .bat).
	3.	Click Save to proceed.

Step 4: Add and Configure Tasks in the Flow

1. Start Task – File Upload Trigger

	•	Click on the “Start” node in the workflow.
	•	Set the trigger type to File Upload.
	•	Select the folder: /incoming-files/.

Click Save.

2. JavaScript Task – File Type Detection

	•	Click “+ Add Task” → Choose JavaScript Task.
	•	Name this task: Detect File Type.
	•	Paste the following code in the JavaScript editor:

const filename = inputFile.getName();
const fileType = filename.split('.').pop().toLowerCase();

workflow.setVariable('fileType', fileType);
workflow.setVariable('fileContent', inputFile.getContent());

	•	Click Save.

3. JavaScript Task – Apply Formulas and Transform Data

	1.	Click “+ Add Task” → Select JavaScript Task.
	2.	Name the task: Apply Formula and Transform.
	3.	Paste the following code inside the JavaScript editor:

let content = workflow.getVariable('fileContent');
let fileType = workflow.getVariable('fileType');
let parsedData;

if (fileType === 'json') {
    // Parse JSON and apply a formula: Sum of A + B
    parsedData = JSON.parse(content);
    parsedData.forEach(row => {
        row['A+B'] = (row['A'] || 0) + (row['B'] || 0);  // Formula applied here
    });
} else if (fileType === 'csv' || fileType === 'log') {
    // Handle CSV or LOG files: Split by lines and columns
    parsedData = content.split('\n').map(line => line.split(','));
} else {
    // Handle BAT files or unknown formats
    parsedData = [{ Content: content }];
}

// Store the transformed data as JSON
workflow.setVariable('transformedData', JSON.stringify(parsedData));

	4.	Where to Click to Add the Formula:
	•	The formula logic (e.g., row['A+B'] = (row['A'] || 0) + (row['B'] || 0)) is added inside this task.
	•	If you need different formulas, you can edit this part as per your requirements.
	5.	Click Save.

4. JavaScript Task – Convert to Delimited Format

	1.	Click “+ Add Task” → Select JavaScript Task.
	2.	Name this task: Convert to CSV or Pipe Format.
	3.	Paste the following code inside the editor:

let transformedData = JSON.parse(workflow.getVariable('transformedData'));
let delimiter = workflow.getVariable('delimiter') || ',';  // Default to comma

let headers = Array.isArray(transformedData[0]) 
    ? transformedData[0] 
    : Object.keys(transformedData[0]);

let rows = transformedData.map(row => 
    headers.map(header => row[header] || '').join(delimiter)
);

let delimitedOutput = [headers.join(delimiter), ...rows].join('\n');
workflow.setVariable('delimitedOutput', delimitedOutput);

	4.	Click Save.

5. Storage or Export Task – Save or Email the Output

	1.	Click “+ Add Task” → Select File Storage Task or Email Task.
	2.	Option 1: File Storage Task
	•	Set Destination Folder: /processed-files/.
	•	Use the following code to save the output:

let output = workflow.getVariable('delimitedOutput');
fileStorage.save('/processed-files/output.txt', output, 'text/plain');


	3.	Option 2: Email Task
	•	Configure the email task with:
	•	Recipient: recipient@example.com
	•	Subject: Converted File
	•	Body: Find the attached file.
	•	Use the following code to attach the file:

let output = workflow.getVariable('delimitedOutput');
email.send({
    to: 'recipient@example.com',
    subject: 'Converted File',
    body: 'Find the attached file.',
    attachments: [{
        filename: 'output.txt',
        content: output,
        mimeType: 'text/plain'
    }]
});


	4.	Click Save.

Step 5: Configure Workflow Variables

	1.	In the Orchestration Designer, click Variables on the right panel.
	2.	Add the following variables:
	•	fileType: Stores the detected file type.
	•	fileContent: Stores the uploaded file content.
	•	transformedData: Stores the transformed data (JSON format).
	•	delimiter: Choose between , or | (default: ,).
	•	delimitedOutput: Stores the final CSV/pipe-delimited output.

Step 6: Test the Orchestration Flow

	1.	Upload files (JSON, CSV, LOG, BAT) to /incoming-files/.
	2.	Monitor the workflow logs to confirm:
	•	File type detection is working.
	•	The formula is applied to JSON data correctly.
	•	CSV or pipe-delimited output is generated.
	3.	Check the /processed-files/ folder or email inbox for the final output.

Summary

In this step-by-step guide, we configured an Orchestration Flow in OpenText Extreme Cloud Native to:

	1.	Detect multiple file types (JSON, CSV, LOG, BAT).
	2.	Apply formulas inside the JavaScript task.
	3.	Convert data into delimited formats (CSV or pipe).
	4.	Store or email the output automatically.

This process ensures seamless automation and efficient file handling within the cloud-native environment. Let me know if you have any further questions or need more help!