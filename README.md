<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Sigmanaut Mining Pool Stats</title>
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
    </style>
</head>
<body>
    <h1>Simple Sigmanaut Mining Pool Stats</h1>
    <div>
        <input type="text" id="wallet-input" placeholder="Enter wallet address">
        <button onclick="fetchMiningStats()">Go</button>
    </div>
    <div id="mining-stats"></div>

    <script>
        // Function to retrieve the stored wallet address from local storage
        function getStoredWalletAddress() {
            return localStorage.getItem('walletAddress') || '';
        }

        // Function to store the entered wallet address in local storage
        function storeWalletAddress(walletAddress) {
            localStorage.setItem('walletAddress', walletAddress);
        }

        // Function to fetch mining stats
        function fetchMiningStats() {
            const walletAddress = document.getElementById('wallet-input').value;
            const apiUrl = 'https://api.codetabs.com/v1/proxy/?quest=http://pool.ergo-sig-mining.net:4000/api/pools/ErgoSigmanauts/miners/';

            if (walletAddress.trim() !== '') {
                const updatedUrl = `${apiUrl}${walletAddress}`;
                
                fetch(updatedUrl)
                    .then(response => response.json())
                    .then(data => {
                        const miningStatsElement = document.getElementById('mining-stats');
                        miningStatsElement.innerHTML = `
                            <h2>Workers</h2>
                            ${Object.entries(data.performance.workers).map(([workerName, workerData]) => `
                                <h3>${workerName}</h3>
                                Hashrate: ${formatHashrate(workerData.hashrate)}<br>
                                Shares Per Second: ${workerData.sharesPerSecond}<br>
                            `).join('')}
                            <h2>Summary</h2>
                            Last Payment: ${data.lastPayment}<br>
                            Pending Balance: ${data.pendingBalance}<br>
                            Pending Shares: ${data.pendingShares}<br>
                        `;
                        // Store the entered wallet address in local storage
                        storeWalletAddress(walletAddress);
                    })
                    .catch(error => {
                        console.error('Error fetching mining stats:', error);
                    });
            } else {
                alert('Please enter a valid wallet address.');
            }
        }

        // Function to format hashrate
        function formatHashrate(hashrate) {
            const mhPerSec = hashrate / 1000000;
            return `${mhPerSec.toFixed(1)} MH/s`;
        }

        // Set the value of the wallet input field to the stored wallet address when the page is loaded
        document.getElementById('wallet-input').value = getStoredWalletAddress();
    </script>
</body>
</html>
