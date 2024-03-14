
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
    <input type="text" id="inputString" placeholder="Enter a wallet address">
    <button onclick="redirectToAPI()">Go</button>

    <script>
        function redirectToAPI() {
            var input = document.getElementById('inputString').value;
            var apiUrl = 'http://15.204.211.130:4000/api/pools/ErgoSigmanauts/miners/' + input;
            window.location.href = apiUrl;
        }
    </script>
</body>
</html>
