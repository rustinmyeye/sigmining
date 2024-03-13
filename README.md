<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>API Data</title>
    <style>
        body {
            background-color: black;
            color: white;
        }
    </style>
</head>
<body>
    <h1>API Data</h1>
    <div id="output"></div>

    <script>
        // Parse URL parameters to retrieve address
        var urlParams = new URLSearchParams(window.location.search);
        var address = urlParams.get('address');

        // Fetch data from the API
        fetch('http://15.204.211.130:4000/api/pools/ErgoSigmanauts/miners/' + address)
            .then(response => response.json())
            .then(data => {
                // Display the data on the webpage
                document.getElementById('output').innerHTML = '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => console.error('Error:', error));
    </script>
</body>
</html>
