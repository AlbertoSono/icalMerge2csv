# icsMerge2csv
The script bundles several .ics files into one .csv file

[TRY THE DEMO](https://htmlpreview.github.io/?https://github.com/AlbertoSono/icsMerge2csv/blob/main/ics2csv.html)

![Screenshot (48)](https://user-images.githubusercontent.com/127153603/223321052-0e3bcd60-f2a8-40a7-9f4c-36da0e4bf905.png)
```javascript
			// upload and list files with filesize
			let fileInput = document.getElementById('fileInput');
			let fileList = document.querySelector('.file-list');

			fileInput.addEventListener('change', function () {
				let files = fileInput.files;
				for (let i = 0; i < files.length; i++) {
					let file = files[i];
					let listItem = document.createElement('div');
					listItem.classList.add('file-item');
					let icon = document.createElement('i');
					icon.classList.add('calendar-icon');
					let name = document.createElement('div');
					name.classList.add('file-name');
					name.innerText = file.name;
					let size = document.createElement('div');
					size.classList.add('file-size');
					size.innerText = formatBytes(file.size);
					listItem.appendChild(icon);
					listItem.appendChild(name);
					listItem.appendChild(size);
					fileList.appendChild(listItem);
				}
			});

			function formatBytes(bytes, decimals = 2) {
				if (bytes === 0) return '0 Bytes';
				const k = 1024;
				const dm = decimals < 0 ? 0 : decimals;
				const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'];
				const i = Math.floor(Math.log(bytes) / Math.log(k));
				return parseFloat((bytes / Math.pow(k, i)).toFixed(dm)) + ' ' + sizes[i];
			}

			// iCalMerge2csv
			function mergeAndExport() {
				const inputElements = document.getElementById("fileInput").files;
				const calendarEvents = [];

				for (let i = 0; i < inputElements.length; i++) {
					const reader = new FileReader();
					reader.onload = function (event) {
						const fileContent = event.target.result;
						const events = parseICal(fileContent);
						calendarEvents.push(...events);

						if (i === inputElements.length - 1) {
							exportAsCSV(calendarEvents);
						}
					};
					reader.readAsText(inputElements[i]);
				}
			}

			function parseICal(fileContent) {
				// Split the file content by line breaks
				const lines = fileContent.trim().split("\n");

				// Find all VEVENT blocks
				const veventBlocks = [];
				let currentBlock = null;
				for (let i = 0; i < lines.length; i++) {
					const line = lines[i].trim();
					if (line === "BEGIN:VEVENT") {
						currentBlock = [];
					} else if (line === "END:VEVENT") {
						veventBlocks.push(currentBlock);
						currentBlock = null;
					} else if (currentBlock !== null) {
						currentBlock.push(line);
					}
				}

				// Parse each VEVENT block into a calendar event object
				const calendarEvents = [];
				for (let i = 0; i < veventBlocks.length; i++) {
					const veventBlock = veventBlocks[i];
					const calendarEvent = {};
					for (let j = 0; j < veventBlock.length; j++) {
						const line = veventBlock[j];
						const [key, value] = line.split(":");
						if (key === "SUMMARY") {
							calendarEvent.summary = value;
						} else if (key === "DTSTART") {
							calendarEvent.start = new Date(value);
						} else if (key === "DTEND") {
							calendarEvent.end = new Date(value);
						} else if (key === "LOCATION") {
							calendarEvent.location = value;
						} else if (key === "DESCRIPTION") {
							calendarEvent.description = value;
						}
					}
					calendarEvents.push(calendarEvent);
				}
				return calendarEvents;
			}

			function exportAsCSV(calendarEvents) {
				const csvContent = "data:text/csv;charset=utf-8," + convertToCSV(calendarEvents);
				const encodedUri = encodeURI(csvContent);
				const link = document.createElement("a");
				link.setAttribute("href", encodedUri);
				link.setAttribute("download", "merged_events.csv");
				document.body.appendChild(link); // Required for Firefox
				link.click();
			}

			function convertToCSV(calendarEvents) {
				const lines = [];
				lines.push("Summary,Start Time,End Time,Location,Description");
				for (let i = 0; i < calendarEvents.length; i++) {
					const event = calendarEvents[i];
					const startTime = event.start.toLocaleString();
					const endTime = event.end.toLocaleString();
					const summary = event.summary || "";
					const location = event.location || "";
					const description = event.description || "";
					lines.push(`${summary},${startTime},${endTime},${location},${description}`);
				}
				return lines.join("\n");
			}
```
