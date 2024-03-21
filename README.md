<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <style>
        body {
            background-color: black;
            color: white;
            font-family: Arial, sans-serif;
        }
        #mining-stats {
            margin: 20px;
        }
        #wallet-input {
            margin-bottom: 10px;
            padding: 5px;
        }
        #input-title {
            font-size: 14px;
            margin-bottom: 5px;
        }
    </style>
</head>
<body>
    <div>
        <div id="input-title">Enter your wallet address or worker name:</div>
        <input type="text" id="wallet-input" placeholder="Enter wallet address or worker name" onchange="saveInput()">
        <button onclick="fetchMiningStats()">Go</button>
    </div>
    <div id="mining-stats"></div>

    <script>
        // Function to fetch mining stats
        function fetchMiningStats() {
            const input = document.getElementById('wallet-input').value.trim();
            if (input.length > 25) {
                fetchMiningStatsForWallet(input);
            } else {
                fetchWorker(input);
            }
        }

        // Function to fetch mining stats for a wallet address
        function fetchMiningStatsForWallet(walletAddress) {
            const url = `https://api.codetabs.com/v1/proxy/?quest=http://pool.ergo-sig-mining.net:4000/api/pools/ErgoSigmanauts/miners/${walletAddress}`;
            fetchAndDisplayStats(url);
        }

        // Function to fetch the list of workers and their associated wallet addresses
        function fetchWorker(workerName) {
            const apiUrl = 'https://api.codetabs.com/v1/proxy/?quest=http://pool.ergo-sig-mining.net:4000/api/pools/ErgoSigmanauts/miners/';
            fetch(apiUrl)
                .then(response => response.json())
                .then(data => {
                    let found = false;
                    data.forEach(miner => {
                        const minerAddress = miner.miner.replace('"', '').replace('"', ''); // Extract the miner address
                        const url = `https://api.codetabs.com/v1/proxy/?quest=http://pool.ergo-sig-mining.net:4000/api/pools/ErgoSigmanauts/miners/${minerAddress}`;
                        fetchAndCheckWorker(url, workerName);
                    });
                })
                .catch(error => {
                    console.error('Error fetching worker list:', error);
                    alert('Error fetching worker list. Please try again later.');
                });
        }

        // Function to fetch mining stats and check for the worker name
        function fetchAndCheckWorker(url, workerName) {
            fetch(url)
                .then(response => response.json())
                .then(stats => {
                    const workers = Object.keys(stats.performance.workers);
                    if (workers.includes(workerName)) {
                        displayStats(stats);
                    }
                })
                .catch(error => {
                    console.error('Error fetching mining stats:', error);
                });
        }

        // Function to fetch mining stats and display them
        function fetchAndDisplayStats(url) {
            fetch(url)
                .then(response => response.json())
                .then(stats => {
                    displayStats(stats);
                })
                .catch(error => {
                    console.error('Error fetching mining stats:', error);
                    alert('Error fetching mining stats. Please try again later.');
                });
        }

        // Function to display mining stats
        function displayStats(stats) {
            const miningStatsElement = document.getElementById('mining-stats');
            miningStatsElement.innerHTML = `
                <h2>Worker Stats</h2>
                ${Object.entries(stats.performance.workers).map(([workerName, workerData]) => `
                    <h3>${workerName}</h3>
                    Hashrate: ${formatHashrate(workerData.hashrate)}<br>
                    Shares Per Second: ${workerData.sharesPerSecond}<br>
                `).join('')}
                <h2>Summary</h2>
                Last Payment: ${stats.lastPayment}<br>
                Pending Balance: ${stats.pendingBalance}<br>
                Pending Shares: ${stats.pendingShares}<br>
            `;
        }

        // Function to format hashrate
        function formatHashrate(hashrate) {
            if (hashrate >= 1000000000) {
                return `${(hashrate / 1000000000).toFixed(1)} GH/s`;
            } else {
                return `${(hashrate / 1000000).toFixed(1)} MH/s`;
            }
        }

        // Function to save user input to localStorage
        function saveInput() {
            const userInput = document.getElementById("wallet-input").value;
            localStorage.setItem("lastInput", userInput);
        }

        // Function to load last input from localStorage
        function loadLastInput() {
            const lastInput = localStorage.getItem("lastInput");
            if (lastInput) {
                document.getElementById("wallet-input").value = lastInput;
                fetchMiningStats(); // Automatically fetch stats based on the saved input
            }
        }

        // Load last input when the page loads
        window.onload = function() {
            loadLastInput();
        }

        // Refresh the page every 2 minutes
        setInterval(function() {
            fetchMiningStats(); // Automatically fetch stats every 2 minutes
        }, 120000); // 2 minutes in milliseconds
    </script>
</body>
</html>
