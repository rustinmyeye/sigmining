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

    <script>
        // Function to fetch mining stats
        function fetchMiningStats() {
            const input = document.getElementById('wallet-input').value.trim();
            const spinner = document.getElementById('loading-spinner');
            const miningStatsElement = document.getElementById('mining-stats');

            if (input.length > 0) {
                spinner.style.display = 'block'; // Show loading spinner
                miningStatsElement.innerHTML = ''; // Clear previous stats

                if (input.length > 25) {
                    // Assume it's a wallet address
                    fetchMiningStatsForWallet(input);
                } else {
                    // Assume it's a worker name
                    fetchMiningStatsForWorker(input);
                }
            } else {
                alert('Please enter a wallet address or worker name.');
            }
        }

        // Function to fetch mining stats for a wallet address
        function fetchMiningStatsForWallet(walletAddress) {
            const statsUrl = `https://api.codetabs.com/v1/proxy/?quest=http://37.27.198.175:8000/sigscore/miners/${walletAddress}/workers`;
            const additionalInfoUrl = `https://api.codetabs.com/v1/proxy/?quest=http://37.27.198.175:8000/sigscore/miners/${walletAddress}`;

            Promise.all([
                fetch(statsUrl).then(response => response.json()),
                fetch(additionalInfoUrl).then(response => response.json())
            ])
            .then(([statsData, additionalData]) => {
                displayAdditionalInfo(additionalData, walletAddress);
                displayStats(statsData);
            })
            .catch(error => {
                console.error('Error fetching data:', error);
                alert('Error fetching mining stats. Please try again later.');
            })
            .finally(() => {
                document.getElementById('loading-spinner').style.display = 'none';
            });
        }

        // Function to fetch the list of all miners and their associated worker stats
        function fetchMiningStatsForWorker(workerName) {
            const apiUrl = 'https://api.codetabs.com/v1/proxy/?quest=http://37.27.198.175:8000/sigscore/miners';
            fetch(apiUrl)
                .then(response => response.json())
                .then(miners => {
                    let found = false;

                    // Iterate through all miners and fetch their worker stats
                    const fetchWorkerStatsForAddress = (address) => {
                        const url = `https://api.codetabs.com/v1/proxy/?quest=http://37.27.198.175:8000/sigscore/miners/${address}/workers`;
                        const additionalInfoUrl = `https://api.codetabs.com/v1/proxy/?quest=http://37.27.198.175:8000/sigscore/miners/${address}`;

                        return Promise.all([
                            fetch(url).then(response => response.json()),
                            fetch(additionalInfoUrl).then(response => response.json())
                        ])
                        .then(([workerData, additionalData]) => {
                            if (workerData[workerName]) {
                                found = true;
                                displayAdditionalInfo(additionalData, address);
                                displayStats(workerData);
                            }
                        })
                        .catch(error => {
                            console.error('Error fetching data:', error);
                        });
                    };

                    // Check all miners
                    const promises = miners.map(miner => fetchWorkerStatsForAddress(miner.address));
                    Promise.all(promises).then(() => {
                        if (!found) {
                            alert('Worker name not found. Please check the input.');
                        }
                        document.getElementById('loading-spinner').style.display = 'none'; // Hide loading spinner
                    });
                })
                .catch(error => {
                    console.error('Error fetching miners list:', error);
                    alert('Error fetching miners list. Please try again later.');
                    document.getElementById('loading-spinner').style.display = 'none'; // Hide loading spinner
                });
        }

        // Function to display additional information
        function displayAdditionalInfo(data, address) {
            const miningStatsElement = document.getElementById('mining-stats');
            let additionalInfoContent = '<h2>Overall Miner Info</h2>';

            additionalInfoContent += `
                Address: ${formatAddress(address)}<br>
                Current Hashrate: ${formatHashrate(data.current_hashrate)}<br>
                Worker Count: ${data.worker_count}<br>
                ${data.last_block_found.block_height && data.last_block_found.timestamp ? 
                    `Last Block Found: Block ${data.last_block_found.block_height} at ${formatDate(data.last_block_found.timestamp)}<br>` : 
                    'Last Block Found: No blocks found yet<br>'
                }
                Balance: ${data.balance.toFixed(8)} ERG<br>
                Last Payment: ${formatDate(data.last_payment.date)}<br>
                Payment Amount: ${data.last_payment.amount.toFixed(8)} ERG<br>
                <br>
            `;

            miningStatsElement.innerHTML += additionalInfoContent;
        }

        // Function to display mining stats
        function displayStats(data) {
            const miningStatsElement = document.getElementById('mining-stats');
            let htmlContent = '<h2>Worker Stats</h2>';

            Object.entries(data).forEach(([workerName, stats]) => {
                // Get the most recent stat
                const mostRecentStat = stats.reduce((latest, current) => {
                    return new Date(current.created) > new Date(latest.created) ? current : latest;
                });

                htmlContent += `<h3>${workerName}</h3>`;
                htmlContent += `
                    Updated: ${formatDate(mostRecentStat.created)}
                    <br>
                    Hashrate: ${formatHashrate(mostRecentStat.hashrate)}
                    <br>
                    Shares Per Second: ${mostRecentStat.sharesPerSecond.toFixed(3)}
                    <br><br>
                `;
            });

            miningStatsElement.innerHTML += htmlContent;
        }

        // Function to format address
        function formatAddress(address) {
            if (address.length > 8) {
                return `${address.slice(0, 4)}...${address.slice(-4)}`;
            }
            return address;
        }

        // Function to format date
        function formatDate(dateString) {
            const date = new Date(dateString);
            return `${date.toLocaleDateString()} ${date.toLocaleTimeString()}`;
        }

        // Function to format hashrate
        function formatHashrate(hashrate) {
            if (hashrate >= 1000000000) {
                return `${(hashrate / 1000000000).toFixed(1)} GH/s`;
            } else {
                return `${(hashrate / 1000000).toFixed(1)} MH/s`;
            }
        }

        // Load last input when the page loads
        window.onload = function() {
            const lastInput = localStorage.getItem("lastInput");
            if (lastInput) {
                document.getElementById("wallet-input").value = lastInput;
                fetchMiningStats(); // Automatically fetch stats based on the saved input
            }
        }

        // Save user input to localStorage on input change
        document.getElementById("wallet-input").addEventListener("input", function() {
            const userInput = document.getElementById("wallet-input").value;
            localStorage.setItem("lastInput", userInput);
        });
    </script>
</body>
</html>
