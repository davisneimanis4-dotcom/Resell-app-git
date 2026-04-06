<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Reseller Command</title>
    <style>
        :root { --primary: #0a84ff; --bg: #000000; --card: #1c1c1e; --text: #ffffff; --green: #30d158; --red: #ff453a; --gray: #8e8e93; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background-color: var(--bg); margin: 0; padding: 0; color: var(--text); padding-bottom: 80px; }
        
        /* App Layout */
        .header { background-color: var(--card); padding: 20px 15px 15px; text-align: center; font-size: 20px; font-weight: 700; border-bottom: 1px solid #333; position: sticky; top: 0; z-index: 100;}
        .tab-content { display: none; padding: 15px; }
        .tab-content.active { display: block; }
        
        /* Bottom Navigation */
        .bottom-nav { position: fixed; bottom: 0; width: 100%; background: rgba(28, 28, 30, 0.9); backdrop-filter: blur(10px); display: flex; justify-content: space-around; padding: 15px 0 25px; border-top: 1px solid #333; z-index: 100;}
        .nav-item { color: var(--gray); font-size: 12px; text-align: center; cursor: pointer; flex: 1; font-weight: 600;}
        .nav-item.active { color: var(--primary); }
        .nav-icon { font-size: 22px; display: block; margin-bottom: 4px; }

        /* Form Elements */
        .card { background: var(--card); border-radius: 14px; padding: 20px; margin-bottom: 15px; }
        label { display: block; margin-top: 12px; font-weight: 600; font-size: 13px; color: var(--gray); }
        input, select { width: 100%; box-sizing: border-box; padding: 14px; margin-top: 8px; border: 1px solid #333; border-radius: 10px; font-size: 16px; background-color: #000; color: white; -webkit-appearance: none; }
        
        button { width: 100%; background-color: var(--primary); color: white; border: none; padding: 16px; border-radius: 12px; font-size: 16px; font-weight: 700; margin-top: 20px; cursor: pointer; transition: 0.2s; }
        button:active { filter: brightness(0.8); }
        button.secondary { background-color: #333; color: white; }
        
        /* Result Box */
        .result { margin-top: 20px; padding: 15px; border-radius: 12px; text-align: center; font-size: 16px; font-weight: bold; display: none; }
        .buy { background-color: rgba(48, 209, 88, 0.2); color: var(--green); border: 1px solid var(--green); }
        .pass { background-color: rgba(255, 69, 58, 0.2); color: var(--red); border: 1px solid var(--red); }
        
        /* Camera */
        .camera-btn { background-color: #333; display: flex; justify-content: center; align-items: center; gap: 8px; }
        input[type="file"] { display: none; }
        #imagePreview { width: 100%; height: 180px; object-fit: cover; border-radius: 10px; margin-top: 15px; display: none; border: 1px solid #333;}

        /* Inventory & Dashboard */
        .dashboard-stats { display: flex; gap: 10px; margin-bottom: 20px; }
        .stat-box { flex: 1; background: var(--card); padding: 15px; border-radius: 12px; text-align: center; border: 1px solid #333;}
        .stat-value { font-size: 20px; font-weight: 700; margin-top: 5px; color: var(--green); }
        .stat-value.spent { color: var(--primary); }
        
        .item-card { background: var(--card); border-radius: 12px; padding: 15px; margin-bottom: 12px; display: flex; gap: 15px; border-left: 4px solid var(--green); }
        .item-card.sold { border-left: 4px solid var(--primary); opacity: 0.8; }
        .item-card img { width: 70px; height: 70px; object-fit: cover; border-radius: 8px; background: #333;}
        .item-info { flex: 1; font-size: 13px; color: var(--gray); }
        .item-info strong { font-size: 16px; display: block; margin-bottom: 4px; color: white;}
        .sell-btn { background: var(--green); padding: 8px; font-size: 12px; margin-top: 8px; }

        /* Comms Tab */
        .template-box { background: #000; border: 1px solid #333; padding: 15px; border-radius: 10px; margin-top: 10px; font-size: 14px; line-height: 1.4; }
        .copy-btn { margin-top: 5px; padding: 12px; }
    </style>
</head>
<body>

    <div class="header" id="appHeader">Scanner</div>

    <div id="tab-source" class="tab-content active">
        <div class="card">
            <button class="camera-btn" onclick="document.getElementById('cameraInput').click()">
                <span class="nav-icon">📷</span> Capture Item
            </button>
            <input type="file" id="cameraInput" accept="image/*" capture="environment" onchange="processImage(event)">
            <img id="imagePreview" src="">

            <label>Item Description</label>
            <input type="text" id="itemName" placeholder="e.g. Vintage Levi's 501">

            <label>Platform / Fees</label>
            <select id="platform">
                <option value="vinted">Vinted (Includes Buyer Protection Fee)</option>
                <option value="marktplaats">Marktplaats / Thrift Store (0 Fees)</option>
            </select>

            <label>Asking Price (€) - Total investment</label>
            <input type="number" id="askingPrice" placeholder="0.00" step="0.01">

            <label>Historical Sold Value (€) - Expected return</label>
            <input type="number" id="salePrice" placeholder="0.00" step="0.01">

            <button onclick="analyzeDeal()">Analyze Deal</button>
            <div id="resultBox" class="result"></div>
            
            <button id="saveBtn" class="secondary" style="display:none; margin-top: 10px;" onclick="saveToVault()">💾 Save to Vault</button>
        </div>
    </div>

    <div id="tab-vault" class="tab-content">
        <div class="dashboard-stats">
            <div class="stat-box">
                <div style="font-size: 12px; color: var(--gray);">Total Sunk Cost</div>
                <div class="stat-value spent" id="totalSpent">€0.00</div>
            </div>
            <div class="stat-box">
                <div style="font-size: 12px; color: var(--gray);">Expected Profit</div>
                <div class="stat-value" id="totalProfit">€0.00</div>
            </div>
        </div>
        <button class="secondary" style="margin-bottom: 20px; padding: 12px;" onclick="exportCSV()">📥 Export Data to CSV</button>
        <div id="inventoryList"></div>
    </div>

    <div id="tab-comms" class="tab-content">
        <div class="card">
            <h3 style="margin-top:0;">Defense Tactics (ROI Focus)</h3>
            <p style="font-size: 13px; color: var(--gray);">Use these to firmly decline lowballs when you are willing to wait for the right price.</p>
            
            <label>Dutch: Holding Firm</label>
            <div class="template-box" id="nl-hold">Bedankt voor je bod! Omdat ik de voorkeur geef aan de juiste waarde in plaats van een snelle verkoop, en dit een kwaliteitsitem is, houd ik vast aan mijn prijs. Laat het me weten als je daarmee akkoord kunt gaan!</div>
            <button class="copy-btn secondary" onclick="copyText('nl-hold')">Copy Dutch</button>

            <label>English: Holding Firm</label>
            <div class="template-box" id="en-hold">Thanks for your offer! Because I prefer to get the right value rather than a quick sale, and this is a quality piece, I'm holding firm on my price. Let me know if you can agree to that!</div>
            <button class="copy-btn secondary" onclick="copyText('en-hold')">Copy English</button>
        </div>

        <div class="card">
            <h3 style="margin-top:0;">Sourcing (Max Offer)</h3>
            
            <label>Dutch: Max Offer</label>
            <div class="template-box" id="nl-buy">Beste, ik heb veel interesse in dit item. Mijn absolute maximum, rekening houdend met de marges en verzending, is €[BEDRAG]. Ik kan het direct overmaken als je akkoord gaat.</div>
            <button class="copy-btn secondary" onclick="copyText('nl-buy')">Copy Dutch</button>

            <label>English: Max Offer</label>
            <div class="template-box" id="en-buy">Hi, I'm highly interested in this item. My absolute maximum, factoring in margins and shipping, is €[AMOUNT]. I can transfer immediately if you agree.</div>
            <button class="copy-btn secondary" onclick="copyText('en-buy')">Copy English</button>
        </div>
    </div>

    <div class="bottom-nav">
        <div class="nav-item active" onclick="switchTab('source', 'Scanner', this)">
            <span class="nav-icon">🎯</span> Source
        </div>
        <div class="nav-item" onclick="switchTab('vault', 'The Vault', this)">
            <span class="nav-icon">💼</span> Vault
        </div>
        <div class="nav-item" onclick="switchTab('comms', 'Comms', this)">
            <span class="nav-icon">💬</span> Comms
        </div>
    </div>

    <script>
        let currentItem = {};

        // --- Navigation Logic ---
        function switchTab(tabId, title, element) {
            document.querySelectorAll('.tab-content').forEach(tab => tab.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(nav => nav.classList.remove('active'));
            document.getElementById(`tab-${tabId}`).classList.add('active');
            element.classList.add('active');
            document.getElementById('appHeader').innerText = title;
            if(tabId === 'vault') renderVault();
        }

        // --- Image Compression Logic (Crucial for LocalStorage) ---
        function processImage(event) {
            const file = event.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                const img = new Image();
                img.onload = function() {
                    const canvas = document.createElement('canvas');
                    const ctx = canvas.getContext('2d');
                    const MAX_WIDTH = 300;
                    const scaleSize = MAX_WIDTH / img.width;
                    canvas.width = MAX_WIDTH;
                    canvas.height = img.height * scaleSize;
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    
                    const compressedDataUrl = canvas.toDataURL('image/jpeg', 0.6); // Compress to 60% quality
                    document.getElementById('imagePreview').src = compressedDataUrl;
                    document.getElementById('imagePreview').style.display = 'block';
                    currentItem.image = compressedDataUrl;
                }
                img.src = e.target.result;
            }
            reader.readAsDataURL(file);
        }

        // --- Core Calculator Logic ---
        function analyzeDeal() {
            const name = document.getElementById('itemName').value || "Unnamed Item";
            const platform = document.getElementById('platform').value;
            const asking = parseFloat(document.getElementById('askingPrice').value) || 0;
            const sale = parseFloat(document.getElementById('salePrice').value) || 0;
            const targetProfit = 15.00; // Hardcoded strict minimum target

            let fees = platform === 'vinted' && asking > 0 ? 0.70 + (asking * 0.05) : 0;
            const totalInvestment = asking + fees;
            const expectedProfit = sale - totalInvestment;
            
            let maxOffer = platform === 'vinted' ? (sale - targetProfit - 0.70) / 1.05 : sale - targetProfit;

            const resultBox = document.getElementById('resultBox');
            resultBox.style.display = 'block';

            currentItem = { ...currentItem, id: Date.now(), name, asking, fees, totalInvestment, expectedProfit, status: 'active' };

            if (expectedProfit >= targetProfit) {
                resultBox.className = 'result buy';
                resultBox.innerHTML = `🟢 ACQUIRE<br>Expected Profit: €${expectedProfit.toFixed(2)}<br><span style="font-size:13px; font-weight:normal; color:#fff;">Max offer to secure €15 profit: <b>€${Math.max(maxOffer, 0).toFixed(2)}</b></span>`;
                document.getElementById('saveBtn').style.display = 'block';
            } else {
                resultBox.className = 'result pass';
                resultBox.innerHTML = `🔴 REJECT (Below €15 Margin)<br>Expected Profit: €${expectedProfit.toFixed(2)}<br><span style="font-size:13px; font-weight:normal; color:#fff;">Target Max Offer: <b>€${Math.max(maxOffer, 0).toFixed(2)}</b></span>`;
                document.getElementById('saveBtn').style.display = 'block'; // Still allow saving if negotiating
            }
        }

        // --- Inventory Management ---
        function saveToVault() {
            if (!currentItem.totalInvestment) return;
            let inventory = JSON.parse(localStorage.getItem('beastVault')) || [];
            inventory.push(currentItem);
            localStorage.setItem('beastVault', JSON.stringify(inventory));
            
            // Reset form
            document.getElementById('itemName').value = '';
            document.getElementById('askingPrice').value = '';
            document.getElementById('salePrice').value = '';
            document.getElementById('imagePreview').style.display = 'none';
            document.getElementById('resultBox').style.display = 'none';
            document.getElementById('saveBtn').style.display = 'none';
            currentItem = {};
            alert("Added to Vault!");
        }

        function renderVault() {
            const inventory = JSON.parse(localStorage.getItem('beastVault')) || [];
            const list = document.getElementById('inventoryList');
            let totalSpent = 0;
            let totalProfit = 0;
            list.innerHTML = "";

            inventory.slice().reverse().forEach(item => {
                if(item.status === 'active') {
                    totalSpent += item.totalInvestment;
                    totalProfit += item.expectedProfit;
                }
                
                const statusHtml = item.status === 'sold' 
                    ? `<div style="color:var(--primary); font-weight:bold; margin-top:5px;">✓ SOLD (Profit: €${item.realizedProfit.toFixed(2)})</div>` 
                    : `<button class="sell-btn" onclick="markSold(${item.id})">Mark as Sold</button>`;

                const imgSrc = item.image || 'data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="70" height="70"><rect width="70" height="70" fill="%23333"/></svg>';

                list.innerHTML += `
                    <div class="item-card ${item.status}">
                        <img src="${imgSrc}">
                        <div class="item-info">
                            <strong>${item.name}</strong>
                            Sunk Cost: €${item.totalInvestment.toFixed(2)}<br>
                            Est. Profit: €${item.expectedProfit.toFixed(2)}
                            ${statusHtml}
                        </div>
                    </div>
                `;
            });

            document.getElementById('totalSpent').innerText = `€${totalSpent.toFixed(2)}`;
            document.getElementById('totalProfit').innerText = `€${totalProfit.toFixed(2)}`;
        }

        function markSold(id) {
            let inventory = JSON.parse(localStorage.getItem('beastVault')) || [];
            const itemIndex = inventory.findIndex(i => i.id === id);
            
            let actualSale = prompt(`What was the final sale price for ${inventory[itemIndex].name}? (€)`);
            if(actualSale !== null && actualSale !== "") {
                inventory[itemIndex].status = 'sold';
                inventory[itemIndex].realizedProfit = parseFloat(actualSale) - inventory[itemIndex].totalInvestment;
                localStorage.setItem('beastVault', JSON.stringify(inventory));
                renderVault();
            }
        }

        // --- Utilities ---
        function copyText(elementId) {
            let text = document.getElementById(elementId).innerText;
            navigator.clipboard.writeText(text).then(() => alert("Copied to clipboard!"));
        }

        function exportCSV() {
            const inventory = JSON.parse(localStorage.getItem('beastVault')) || [];
            if(inventory.length === 0) return alert("Vault is empty!");
            
            let csvContent = "data:text/csv;charset=utf-8,Item Name,Status,Cost,Expected Profit,Realized Profit\n";
            inventory.forEach(i => {
                let row = `"${i.name}",${i.status},${i.totalInvestment.toFixed(2)},${i.expectedProfit.toFixed(2)},${i.realizedProfit ? i.realizedProfit.toFixed(2) : 0}`;
                csvContent += row + "\n";
            });
            
            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", "reseller_vault_export.csv");
            document.body.appendChild(link);
            link.click();
        }

        // Init
        if(window.location.hash === '#vault') switchTab('vault', 'The Vault', document.querySelectorAll('.nav-item')[1]);
    </script>
</body>
</html>
