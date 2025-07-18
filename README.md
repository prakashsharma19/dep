<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>PPH Dept.</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 0;
    }
    header {
      background-color: #003366;
      color: #fff;
      padding: 15px 30px;
      font-size: 24px;
    }
    .container {
      padding: 30px;
      max-width: 800px;
      margin: auto;
      background-color: #fff;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
      border-radius: 10px;
      margin-top: 40px;
    }
    #dropArea {
      border: 2px dashed #ccc;
      border-radius: 6px;
      padding: 20px;
      text-align: center;
      margin-bottom: 15px;
      color: #777;
      background-color: #fafafa;
      cursor: pointer;
    }
    #fileInput {
      display: none;
    }
    #fileList {
      font-size: 14px;
      color: #444;
      margin-bottom: 15px;
    }
    .file-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 5px 10px;
      background: #f1f1f1;
      border-radius: 4px;
      margin-bottom: 5px;
    }
    .file-item button {
      background: red;
      color: #fff;
      border: none;
      border-radius: 50%;
      width: 16px;
      height: 16px;
      line-height: 14px;
      font-size: 10px;
      cursor: pointer;
      padding: 0;
    }
    input[type="file"], input[type="text"], button {
      display: block;
      width: 100%;
      margin-bottom: 15px;
      padding: 10px;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #ccc;
    }
    button {
      background-color: #003366;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background-color: #0055aa;
    }

    .keyword-container {
      display: flex;
      align-items: flex-start;
      justify-content: space-between;
    }

    #searchTerm {
      flex: 1;
      width: auto;
    }

    .suggestions {
      margin-left: 20px;
      width: 200px;
    }

    .suggestions ul {
      list-style: none;
      padding: 0;
      margin: 0;
    }

    .suggestions li {
      background: #f1f1f1;
      margin-bottom: 5px;
      padding: 5px 8px;
      border-radius: 4px;
      cursor: pointer;
      white-space: nowrap;
      font-size: 12px;
      color: blue;
    }

    .suggestions li:hover {
      background: #e0e0e0;
    }
  </style>
</head>
<body>
  <header>PPH Dept.</header>
  <div class="container">
    <h2>Department-wise Entries</h2>
    <div id="dropArea">Drag &amp; Drop files here or click to browse</div>
    <input type="file" id="fileInput" multiple accept=".txt">
    <div id="fileList"></div>
    <div class="keyword-container">
      <input type="text" id="searchTerm" placeholder="Enter Any Keyword (e.g., Department/Institute/Subjects)">
      <div class="suggestions">
        <ul id="suggestionsList"></ul>
      </div>
    </div>
    <button onclick="extractEntries()">Extract and Download</button>
  </div>

  <script>
    const dropArea = document.getElementById('dropArea');
    const fileInput = document.getElementById('fileInput');
    const fileListDisplay = document.getElementById('fileList');
    let allFiles = [];

    const defaultSuggestions = ['Department of Mathematics', 'Department of Statistics'];

    function getStoredSuggestions() {
      try {
        return JSON.parse(localStorage.getItem('keywordSuggestions')) || [];
      } catch (e) {
        return [];
      }
    }

    function updateSuggestionsUI() {
      const list = document.getElementById('suggestionsList');
      if (!list) return;
      const stored = getStoredSuggestions();
      const all = Array.from(new Set(defaultSuggestions.concat(stored)));
      list.innerHTML = all
        .map(s => `<li onclick="selectSuggestion('${s.replace(/'/g, "\\'")}')">${s}</li>`)
        .join('');
    }

    function selectSuggestion(text) {
      document.getElementById('searchTerm').value = text;
    }

    function saveSuggestion(keyword) {
      if (!keyword) return;
      const stored = getStoredSuggestions();
      if (!stored.includes(keyword)) {
        stored.push(keyword);
        localStorage.setItem('keywordSuggestions', JSON.stringify(stored));
        updateSuggestionsUI();
      }
    }

    document.addEventListener('DOMContentLoaded', updateSuggestionsUI);

    function updateFileList() {
      fileListDisplay.innerHTML = allFiles.map((f, i) => `
        <div class="file-item">
          <span>${f.name}</span>
          <button onclick="removeFile(${i})">&times;</button>
        </div>`).join('');
    }

    function removeFile(index) {
      allFiles.splice(index, 1);
      updateFileList();
    }

    dropArea.addEventListener('dragover', (e) => {
      e.preventDefault();
      dropArea.style.borderColor = '#003366';
    });

    dropArea.addEventListener('dragleave', () => {
      dropArea.style.borderColor = '#ccc';
    });

    dropArea.addEventListener('drop', (e) => {
      e.preventDefault();
      dropArea.style.borderColor = '#ccc';
      allFiles = allFiles.concat(Array.from(e.dataTransfer.files));
      updateFileList();
    });

    dropArea.addEventListener('click', () => {
      fileInput.click();
    });

    fileInput.addEventListener('change', () => {
      allFiles = allFiles.concat(Array.from(fileInput.files));
      updateFileList();
    });

    function extractEntries() {
      const keywordInput = document.getElementById('searchTerm').value.trim();
      const keyword = keywordInput.toLowerCase();
      if (!allFiles.length || !keyword) {
        alert("Please select file(s) and enter a keyword.");
        return;
      }

      saveSuggestion(keywordInput);

      let output = '';
      let entryCount = 0;
      let filesRead = 0;

      allFiles.forEach(file => {
        const reader = new FileReader();
        reader.onload = function(e) {
          const lines = e.target.result.split(/\r?\n/);
          let block = [];
          for (let line of lines) {
            if (line.trim() === '') {
              if (block.join('\n').toLowerCase().includes(keyword)) {
                output += block.join('\n') + '\n\n';
                entryCount++;
              }
              block = [];
            } else {
              block.push(line);
            }
          }
          if (block.length && block.join('\n').toLowerCase().includes(keyword)) {
            output += block.join('\n') + '\n\n';
            entryCount++;
          }

          filesRead++;
          if (filesRead === allFiles.length) {
            if (!output) {
              alert("No matches found.");
              return;
            }

            const mainFileName = allFiles[0].name.split('.')[0].split(/[- ]/)[0];
            const cleanedKeyword = keywordInput.replace(/\s+/g, ' ').trim();
            const fileName = `${mainFileName} - ${cleanedKeyword} (${entryCount} Entries).txt`;

            const blob = new Blob([output], {type: "text/plain"});
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.download = fileName;
            link.click();
          }
        };
        reader.readAsText(file);
      });
    }
  </script>
</body>
</html>
