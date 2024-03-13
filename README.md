
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

    <script>
        function getData() {
            var input = document.getElementById('inputString').value;
            var apiUrl = 'http://15.204.211.130:4000/api/pools/ErgoSigmanauts/miners/' + input;
            fetch(apiUrl)
                .then(response => response.json())
                .then(data => {
                    // Redirect to a new page to display the data
                    window.location.href = 'display.html?address=' + input + '&data=' + encodeURIComponent(JSON.stringify(data));
                })
                .catch(error => console.error('Error:', error));
        }
    </script>
</body>
</html>
