
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
    <script src="https://cdn.jsdelivr.net/npm/confetti-js/dist/index.min.js"></script>
</head>
<body>
    <input type="text" id="inputString" placeholder="Enter a wallet address">
    <button onclick="redirectToAPI()">Go</button>

    <script>
        function redirectToAPI() {
            var input = document.getElementById('inputString').value;
            var apiUrl = 'http://15.204.211.130:4000/api/pools/ErgoSigmanauts/miners/' + input;
            
            // Trigger confetti animation
            confetti({
                particleCount: 100,
                spread: 70,
                origin: { y: 0.6 }
            });

            // Redirect to API URL after a delay (2 seconds)
            setTimeout(function() {
                window.location.href = apiUrl;
            }, 2000);
        }
    </script>
</body>
</html>
