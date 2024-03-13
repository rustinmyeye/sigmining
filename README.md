<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>API Example</title>
    <style>
        body {
            background-color: black;
            color: white;
        }
    </style>
</head>
<body>
    <h1>Wallet Address</h1>
    <input type="text" id="inputString" placeholder="Enter a wallet address">
    <button onclick="getData()">Go</button>
    <div id="output"></div>

    <script>
        function getData() {
            var input = document.getElementById('inputString').value;
            var apiUrl = 'http://15.204.211.130:4000/api/pools/ErgoSigmanauts/miners/' + input;
            fetch(apiUrl)
                .then(response => response.json())
                .then(data => {
                    // Extract required information
                    var address = data.performance.workers.rustinmyeye.hashrate;
                    var hashrate = data.performance.workers.rustinmyeye.hashrate;
                    var time = data.performance.created;

                    // Display extracted information on the webpage
                    document.getElementById('output').innerHTML = '<p>Address: ' + address + '</p>';
                    document.getElementById('output').innerHTML += '<p>Hashrate: ' + hashrate + '</p>';
                    document.getElementById('output').innerHTML += '<p>Time: ' + time + '</p>';
                })
                .catch(error => console.error('Error:', error));
        }
    </script>
</body>
</html>

