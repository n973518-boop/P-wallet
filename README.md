<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的支出小本本 - 支付工具增強版</title>
    
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <style>
        :root {
            --bg-main: #f4f7f6;
            --card-shadow: 0 10px 25px rgba(0,0,0,0.05);
            --text-dark: #2d3436;
            --text-gray: #636e72;
            --edit-blue: #0984e3;
            --delete-red: #d63031;
            --save-pink: #6c5ce7;
            /* 支付工具顏色 */
            --color-cash: #27ae60;
            --color-card: #e67e22;
            --color-e-pay: #9b59b6;
        }

        body {
            background-color: var(--bg-main);
            font-family: 'PingFang TC', 'Microsoft JhengHei', system-ui, sans-serif;
            color: var(--text-dark);
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            margin: 0;
        }

        .card {
            background: #ffffff;
            padding: 25px;
            border-radius: 20px;
            box-shadow: var(--card-shadow);
            width: 100%;
            max-width: 450px;
            margin-bottom: 25px;
            box-sizing: border-box;
        }

        h2, h3 { font-weight: 700; margin-top: 0; }
        label { font-size: 0.85rem; font-weight: 600; color: var(--text-gray); margin: 15px 0 5px 5px; display: block; }

        input, select {
            width: 100%; padding: 12px; border: 1px solid #dfe6e9;
            border-radius: 10px; font-size: 1rem; background: #fdfdfd; box-sizing: border-box;
        }

        #saveBtn {
            display: block; width: 200px; padding: 14px;
            background: linear-gradient(135deg, #6c5ce7 0%, #a29bfe 100%);
            color: white; border: none; border-radius: 12px;
            font-weight: 700; cursor: pointer; margin: 25px auto 0;
            transition: all 0.3s ease;
        }

        /* 支付工具小標籤 */
        .pay-tag {
            font-size: 0.75rem;
            padding: 2px 8px;
            border-radius: 5px;
            color: white;
            font-weight: bold;
            display: inline-block;
            margin-top: 4px;
        }
        .tag-cash { background-color: var(--color-cash); }
        .tag-card { background-color: var(--color-card); }
        .tag-epay { background-color: var(--color-e-pay); }

        .total-section {
            background: #2d3436; color: white; padding: 20px;
            border-radius: 15px; text-align: center; margin-bottom: 20px;
        }

        #totalAmount { font-size: 2.2rem; font-weight: 800; display: block; color: #fab1a0; }

        .expense-item {
            background: white; padding: 15px; border-radius: 15px;
            margin-bottom: 12px; display: flex; justify-content: space-between;
            align-items: center; border: 1px solid rgba(0,0,0,0.03); list-style: none;
        }

        .item-amount { font-size: 1.2rem; font-weight: 800; margin-bottom: 8px; }

        .delete-btn {
            width: auto; padding: 6px 12px; font-size: 0.8rem; font-weight: 700;
            border-radius: 8px; border: none; color: white; cursor: pointer; margin-left: 5px;
        }

        .stat-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #eee; }
    </style>
</head>
<body>

    <div class="card">
        <h2>✍️ 新增支出</h2>
        <label>日期</label><input type="date" id="expenseDate">
        
        <label>支出類別</label>
        <select id="expenseCategory">
            <option value="food">🍱 餐飲費用</option>
            <option value="snacks">☕ 咖啡、甜點、零食</option>
            <option value="clothing">👕 服飾</option>
            <option value="housing">🏠 住家費用</option>
            <option value="subscription">📺 軟體訂閱</option>
            <option value="entertainment">🎮 娛樂</option>
            <option value="lego">🧱 LEGO</option>
            <option value="sport">🏃 運動</option>
            <option value="travel">✈️ 旅行</option>
            <option value="medical">🏥 醫療</option>
            <option value="insurance">🛡️ 保險</option>
            <option value="interest">📈 利息費用</option>
            <option value="fee">💸 手續費</option>
        </select>

        <label>支付工具</label>
        <select id="paymentMethod">
            <option value="cash">💵 現金</option>
            <option value="credit_card">💳 信用卡</option>
            <option value="e_payment">📱 電子支付 (LinePay/街口)</option>
        </select>

        <label>金額</label><input type="number" id="expenseAmount">
        <label>明細</label><input type="text" id="expenseDetail">
        <label>Lab</label><input type="text" id="expenseLab">
        <button id="saveBtn">儲存紀錄</button>
    </div>

    <div class="card">
        <h3>📊 支出統計分析</h3>
        <div style="display: flex; gap: 10px; margin-bottom: 15px;">
            <select id="statsYear" onchange="updateCharts()"></select>
            <select id="statsMonth" onchange="updateCharts()">
                <option value="all">全年統計</option>
                <option value="01">1月</option><option value="02">2月</option>
                <option value="03">3月</option><option value="04">4月</option>
                <option value="05">5月</option><option value="06">6月</option>
                <option value="07">7月</option><option value="08">8月</option>
                <option value="09">9月</option><option value="10">10月</option>
                <option value="11">11月</option><option value="12">12月</option>
            </select>
        </div>
        <div style="position: relative; height:250px;"><canvas id="expenseChart"></canvas></div>
        <div id="statsBreakdown" style="margin-top: 15px;"></div>
    </div>

    <div class="card">
        <h3>🌸 支出小本本</h3>
        <div class="total-section">總計：<span id="totalAmount">0</span> 元</div>
        <ul id="expenseList" style="padding:0;"></ul>
    </div>

    <script>
        // Firebase 設定
        const firebaseConfig = {
            apiKey: "AIzaSyCtooK8RTyqf9Dw7mt1XJNtzzSUr7cwFSs",
            authDomain: "my-wallet-4f965.firebaseapp.com",
            databaseURL: "https://my-wallet-4f965-default-rtdb.firebaseio.com/",
            projectId: "my-wallet-4f965",
            storageBucket: "my-wallet-4f965.firebasestorage.app",
            messagingSenderId: "1040337086266",
            appId: "1:1040337086266:web:326d03ee51e7dd3e7a5203"
        };

        if (!firebase.apps.length) firebase.initializeApp(firebaseConfig);
        const database = firebase.database();
        let myChart = null;
        let allRawData = [];

        function initYearFilter() {
            const yearSelect = document.getElementById('statsYear');
            if (yearSelect.options.length > 0) return;
            const currentYear = new Date().getFullYear();
            for (let i = 0; i < 3; i++) {
                let year = currentYear - i;
                yearSelect.options[yearSelect.options.length] = new Option(`${year}年`, year);
            }
        }

        database.ref('expenses').orderByChild('date').on('value', (snapshot) => {
            allRawData = [];
            const listContainer = document.getElementById('expenseList');
            listContainer.innerHTML = "";
            snapshot.forEach((child) => { allRawData.push({ id: child.key, ...child.val() }); });

            const categoryNames = { "food": "🍱 餐飲", "snacks": "☕ 點心", "clothing": "👕 服飾", "housing": "🏠 住家", "subscription": "📺 訂閱", "entertainment": "🎮 娛樂", "lego": "🧱 LEGO", "sport": "🏃 運動", "travel": "✈️ 旅遊", "medical": "🏥 醫療", "insurance": "🛡️ 保險", "interest": "📈 利息", "fee": "💸 手續費", "default": "💰 其他" };
            
            const payLabels = { "cash": "💵 現金", "credit_card": "💳 卡片", "e_payment": "📱 電支" };
            const payClasses = { "cash": "tag-cash", "credit_card": "tag-card", "e_payment": "tag-epay" };

            [...allRawData].reverse().forEach((data) => {
                const name = categoryNames[data.category] || categoryNames["default"];
                const payText = payLabels[data.payment] || "💵 現金";
                const payClass = payClasses[data.payment] || "tag-cash";

                const listItem = `
                    <li class="expense-item">
                        <div>
                            <strong>${name.split(' ')[0]} ${data.date}</strong><br>
                            <small>${data.detail || ''} (${data.lab || '一般'})</small><br>
                            <span class="pay-tag ${payClass}">${payText}</span>
                        </div>
                        <div style="text-align: right;">
                            <div class="item-amount">$${data.amount.toLocaleString()}</div>
                            <button onclick="editItem('${data.id}')" class="delete-btn" style="background:#0984e3">修改</button>
                            <button onclick="deleteItem('${data.id}')" class="delete-btn" style="background:#d63031">刪除</button>
                        </div>
                    </li>`;
                listContainer.insertAdjacentHTML('beforeend', listItem);
            });

            initYearFilter();
            updateTotal();
            updateCharts();
        });

        window.updateCharts = () => {
            const year = document.getElementById('statsYear').value;
            const month = document.getElementById('statsMonth').value;
            const filtered = allRawData.filter(d => d.date.startsWith(`${year}-${month === 'all' ? '' : month}`));
            
            const categoryTotals = {};
            let total = 0;
            filtered.forEach(d => {
                categoryTotals[d.category] = (categoryTotals[d.category] || 0) + d.amount;
                total += d.amount;
            });

            const ctx = document.getElementById('expenseChart').getContext('2d');
            if (myChart) myChart.destroy();
            myChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(categoryTotals),
                    datasets: [{ data: Object.values(categoryTotals), backgroundColor: ['#fd79a8','#74b9ff','#55efc4','#ffe119','#a29bfe','#fab1a0','#81ecec','#6c5ce7'] }]
                },
                options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom' } } }
            });

            let html = `<strong>📊 ${month === 'all' ? '全年' : month+'月'} 分析</strong><hr style="border:0.5px solid #eee;">`;
            for (let cat in categoryTotals) {
                const p = ((categoryTotals[cat] / total) * 100).toFixed(1);
                html += `<div class="stat-row"><span>${cat}</span><span>$${categoryTotals[cat].toLocaleString()} (${p}%)</span></div>`;
            }
            document.getElementById('statsBreakdown').innerHTML = total > 0 ? html : "尚無資料";
        };

        function updateTotal() {
            let total = allRawData.reduce((sum, d) => sum + Number(d.amount), 0);
            document.getElementById('totalAmount').innerText = total.toLocaleString();
        }

        document.getElementById('saveBtn').addEventListener('click', () => {
            if (document.getElementById('saveBtn').innerText.includes("更新")) return;
            const data = {
                date: document.getElementById('expenseDate').value,
                category: document.getElementById('expenseCategory').value,
                payment: document.getElementById('paymentMethod').value,
                amount: Number(document.getElementById('expenseAmount').value),
                detail: document.getElementById('expenseDetail').value,
                lab: document.getElementById('expenseLab').value
            };
            if (data.date && data.amount) database.ref('expenses').push(data).then(() => {
                document.getElementById('expenseAmount').value = "";
                document.getElementById('expenseDetail').value = "";
            });
        });

        window.deleteItem = (id) => { if (confirm("確定刪除嗎？")) database.ref('expenses/' + id).remove(); };

        window.editItem = (id) => {
            const data = allRawData.find(d => d.id === id);
            document.getElementById('expenseDate').value = data.date;
            document.getElementById('expenseCategory').value = data.category;
            document.getElementById('paymentMethod').value = data.payment || "cash";
            document.getElementById('expenseAmount').value = data.amount;
            document.getElementById('expenseDetail').value = data.detail;
            document.getElementById('expenseLab').value = data.lab;
            
            const btn = document.getElementById('saveBtn');
            btn.innerText = "更新紀錄";
            btn.style.background = "#0984e3";
            btn.onclick = () => {
                database.ref('expenses/' + id).update({
                    date: document.getElementById('expenseDate').value,
                    category: document.getElementById('expenseCategory').value,
                    payment: document.getElementById('paymentMethod').value,
                    amount: Number(document.getElementById('expenseAmount').value),
                    detail: document.getElementById('expenseDetail').value,
                    lab: document.getElementById('expenseLab').value
                }).then(() => {
                    btn.innerText = "儲存紀錄"; btn.style.background = ""; btn.onclick = null;
                    document.getElementById('expenseAmount').value = ""; document.getElementById('expenseDetail').value = "";
                });
            };
            window.scrollTo({ top: 0, behavior: 'smooth' });
        };
    </script>
</body>
</html>
