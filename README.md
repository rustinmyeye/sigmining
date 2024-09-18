<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title></title>
 <style>
        body {
            background-color: black;
            color: white;
            font-family: monospace; /* Terminal-like font */
            margin: 0;
            padding: 0;
            text-align: center;
        }

        #title {
            margin-top: 20px;
            font-size: 18px;
            white-space: pre;
            font-family: monospace; /* Terminal-like font */
            color: amber;
        }

        #input-title {
            margin-top: 20px;
            font-size: 18px;
        }

        input[type="text"] {
            color: white;
            background-color: black;
            border: 1px solid white;
            padding: 5px;
            font-size: 16px;
            width: 300px;
        }

        button {
            color: white;
            background-color: black;
            border: 1px solid white;
            padding: 5px 10px;
            font-size: 16px;
            cursor: pointer;
        }

        button:hover {
            background-color: #333;
        }

        .spinner-container {
            position: relative;
            height: 30px;
            overflow: hidden;
            margin-top: 10px;
            text-align: center;
        }

        .spinner {
            position: absolute;
            white-space: nowrap;
            width: 100%;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            animation: scroll 15s linear infinite; /* Adjusted duration for smoother scroll */
        }

        .spinner span {
            display: inline-block;
            margin: 0 2px;
            font-size: 24px; /* Adjust size for better visibility */
            color: amber; /* Set color to amber */
            opacity: 1;
            animation: flicker 0.1s infinite alternate;
        }

        @keyframes scroll {
            0% { transform: translateX(100%); }
            100% { transform: translateX(-100%); }
        }

        @keyframes flicker {
            0% { opacity: 1; }
            100% { opacity: 0.6; }
        }

        #mining-stats {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div id="title">
 ___ ___ ___ ___  ___   ___  _    
/ __|_ _/ __| _ \/ _ \ / _ \| |   
\__ \| | (_ |  _/ (_) | (_) | |__ 
|___/___\___|_|  \___/ \___/|____|
                                   
    </div>
    <div id="input-title">To check your connection to the pool, enter your wallet address or worker name below:</div>
    <input type="text" id="wallet-input" placeholder="Enter wallet address or worker name">
    <button onclick="fetchMiningStats()">Go</button>
    <div class="spinner-container">
        <div id="loading-spinner" class="spinner">
            <!-- Quadruple the symbols -->
            <span>*</span><span>~</span><span>*</span><span>~</span><span>*</span><span>~</span>
            <span>*</span><span>~</span><span>*</span><span>~</span><span>*</span><span>~</span>
            <span>*</span><span>~</span><span>*</span><span>~</span><span>*</span><span>~</span>
            <span>*</span><span>~</span><span>*</span><span>~</span><span>*</span><span>~</span>
            <span>*</span><span>~</span><span>*</span><span>~</span><span>*</span><span>~</span>
            <span>*</span><span>~</span><span>*</span><span>~</span><span>*</span><span>~</span>
        </div>
    </div>
    <div id="mining-stats"></div>

   <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

<body>
    <div id="input-title">Enter your worker name or wallet address below, or visit one of the dashboards linked beneath.</div>
    <input type="text" id="wallet-input" placeholder="Enter wallet address or worker name" style="color: black;">
    <button onclick="fetchMiningStats()">Go</button>
    <div id="mining-stats"></div>
    <div id="loading-spinner" class="spinner" style="display: none;"></div>

    <script>
        let isChartVisible = false;

        window.onload = function() {
            const savedInput = localStorage.getItem('walletInput');
            if (savedInput) {
                document.getElementById('wallet-input').value = savedInput;
            }
        };

        function fetchMiningStats() {
            const input = document.getElementById('wallet-input').value.trim();
            localStorage.setItem('walletInput', input);
            if (input.length > 45) {
                fetchMiningStatsForWallet(input);
            } else {
                fetchWorker(input);
            }
        }

        function fetchMiningStatsForWallet(walletAddress) {
            const url = `https://api.codetabs.com/v1/proxy/?quest=65.108.57.232:4000/api/pools/ErgoSigmanauts/miners/${walletAddress}`;
            fetchAndDisplayStats(url, walletAddress);
        }

        function fetchWorker(workerName) {
            document.getElementById('loading-spinner').style.display = 'block';
            const apiUrl = 'https://api.codetabs.com/v1/proxy/?quest=http://65.108.57.232:4000/api/pools/ErgoSigmanauts/miners?pageSize=5000';
            fetch(apiUrl)
                .then(response => response.json())
                .then(data => {
                    data.forEach((miner, index) => {
                        const minerAddress = miner.miner.replace('"', '').replace('"', '');
                        const url = `https://api.codetabs.com/v1/proxy/?quest=http://65.108.57.232:4000/api/pools/ErgoSigmanauts/miners/${minerAddress}`;
                        setTimeout(() => fetchAndCheckWorker(url, workerName, minerAddress), index * 250);
                    });
                })
                .catch(error => console.error('Error fetching worker list:', error));
        }

        function fetchAndCheckWorker(url, workerName, minerAddress) {
            fetch(url)
                .then(response => response.json())
                .then(stats => {
                    if (Object.keys(stats.performance.workers).includes(workerName)) {
                        displayStats(stats, url, minerAddress);
                    }
                })
                .catch(error => console.error('Error fetching mining stats:', error));
        }

        function fetchAndDisplayStats(url, walletAddress) {
            fetch(url)
                .then(response => response.json())
                .then(stats => displayStats(stats, url, walletAddress))
                .catch(error => console.error('Error fetching mining stats:', error));
        }

        function displayStats(stats, statsUrl, walletAddress) {
            const workers = stats.performance.workers;
            const totalWorkers = Object.keys(workers).length;
            const totalHashrate = Object.values(workers).reduce((sum, workerData) => sum + workerData.hashrate, 0);
            const formattedAddress = formatMinerAddress(walletAddress);

            fetchLastPayment(walletAddress).then(lastPaymentInfo => {
                const miningStatsElement = document.getElementById('mining-stats');
                miningStatsElement.innerHTML = `
                    <h2>Summary</h2>
                    Address: ${formattedAddress}<br>
                    Total Hashrate: ${formatHashrate(totalHashrate)}<br>
                    Total Workers: ${totalWorkers}<br>
                    Last Payment: ${lastPaymentInfo.date} - Amount: ${lastPaymentInfo.amount} ERG 
                    <a href="${lastPaymentInfo.transactionLink}" target="_blank">View Transaction</a><br>
                    Pending Balance: ${stats.pendingBalance.toFixed(3)} ERG<br>
                    <a href="#" id="toggle-performance" onclick="toggle24hrPerformance(event, '${statsUrl}')">Show 24hr Performance</a><br>
                    <div id="chart-container" class="chart-container"></div>
                    <h2>Worker Stats</h2>
                    ${Object.entries(workers).map(([workerName, workerData]) => `
                        <h3>${workerName}</h3>
                        Hashrate: ${formatHashrate(workerData.hashrate)}<br>
                        Shares Per Second: ${workerData.sharesPerSecond}<br>
                    `).join('')}
                `;
                document.getElementById('loading-spinner').style.display = 'none';
            });
        }

        function fetchLastPayment(walletAddress) {
            const url = `https://api.codetabs.com/v1/proxy/?quest=http://65.108.57.232:4000/api/pools/ErgoSigmanauts/miners/${walletAddress}/payments`;
            return fetch(url)
                .then(response => response.json())
                .then(data => {
                    const lastPayment = data[0];
                    return {
                        date: new Date(lastPayment.created).toLocaleString(),
                        amount: lastPayment.amount.toFixed(6),
                        transactionLink: lastPayment.transactionInfoLink
                    };
                })
                .catch(error => {
                    console.error('Error fetching last payment data:', error);
                    return {
                        date: 'N/A',
                        amount: '0.000000',
                        transactionLink: '#'
                    };
                });
        }

        function formatMinerAddress(address) {
            if (address && address.length > 8) {
                return `${address.slice(0, 4)}...${address.slice(-4)}`;
            }
            return address;
        }

        function formatHashrate(hashrate) {
            return hashrate >= 1000000000 ? `${(hashrate / 1000000000).toFixed(1)} GH/s` : `${(hashrate / 1000000).toFixed(1)} MH/s`;
        }

        function toggle24hrPerformance(event, statsUrl) {
            event.preventDefault();
            const chartContainer = document.getElementById('chart-container');
            const toggleLink = document.getElementById('toggle-performance');

            if (isChartVisible) {
                chartContainer.innerHTML = '';
                toggleLink.textContent = 'Show 24hr Performance';
            } else {
                show24hrPerformance(statsUrl);
                toggleLink.textContent = 'Hide 24hr Performance';
            }
            isChartVisible = !isChartVisible;
        }

        function show24hrPerformance(statsUrl) {
            const url = `${statsUrl}/performance`;
            fetch(url)
                .then(response => response.json())
                .then(data => {
                    const labels = data.map(item => new Date(item.created).toLocaleTimeString());
                    const datasets = [];
                    const workers = new Set();
                    const colors = [
                        '#1f77b4', '#ff7f0e', '#2ca02c', '#d62728', '#9467bd',
                        '#8c564b', '#e377c2', '#7f7f7f', '#bcbd22', '#17becf',
                        '#aec7e8', '#ffbb78', '#98df8a', '#ff9896', '#c5b0d5',
                        '#c49c94', '#f7b6d2', '#dbdb8d', '#9edae5', '#f5b041',
                        '#e67e22', '#e74c3c', '#3498db', '#2ecc71', '#1abc9c',
                        '#9b59b6', '#34495e', '#16a085', '#27ae60', '#2980b9'
                    ];

                    data.forEach(item => Object.keys(item.workers).forEach(worker => workers.add(worker)));
                    const workerArray = Array.from(workers);

                    workerArray.forEach((worker, index) => {
                        datasets.push({
                            x: data.map(item => new Date(item.created)),
                            y: data.map(item => item.workers[worker]?.hashrate || 0),
                            mode: 'lines+markers',
                            type: 'scatter',
                            name: worker,
                            line: { shape: 'linear' },
                            marker: { color: colors[index % colors.length] },
                            text: data.map(item => formatHashrate(item.workers[worker]?.hashrate || 0)),
                            textposition: 'top center',
                            hoverinfo: 'x+y+text'
                        });
                    });

                    const isDarkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
                    const layout = {
                        title: { text: '24hr Hashrate Performance', font: { color: isDarkMode ? '#fff' : '#000' } },
                        xaxis: { title: 'Time', type: 'date', titlefont: { color: isDarkMode ? '#fff' : '#000' }, tickfont: { color: isDarkMode ? '#fff' : '#000' } },
                        yaxis: { 
                            title: 'Hashrate', 
                            titlefont: { color: isDarkMode ? '#fff' : '#000' }, 
                            tickfont: { color: isDarkMode ? '#fff' : '#000' },
                            tickvals: [], 
                            ticktext: [] 
                        },
                        plot_bgcolor: isDarkMode ? '#000' : '#fff',
                        paper_bgcolor: isDarkMode ? '#000' : '#fff',
                        hovermode: 'x',
                        hoverlabel: { namelength: 0, bgcolor: isDarkMode ? '#333' : '#ccc' }
                    };

                    // Format y-axis ticks at 100 MH intervals
                    const maxHashrate = Math.max(...data.map(item => Math.max(...Object.values(item.workers).map(w => w.hashrate || 0))));
                    const yTickInterval = 100000000; // 100 MH
                    layout.yaxis.tickvals = Array.from({length: Math.floor(maxHashrate / yTickInterval) + 1}, (_, i) => i * yTickInterval);
                    layout.yaxis.ticktext = layout.yaxis.tickvals.map(val => formatHashrate(val));

                    Plotly.newPlot('chart-container', datasets, layout);
                })
                .catch(error => console.error('Error fetching 24hr performance data:', error));
        }
    </script>
</body>
</html>
