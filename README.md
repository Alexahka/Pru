<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Кнопки Приложение</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: #f4f4f4;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 400px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        .btn {
            display: block;
            width: 100%;
            margin: 10px 0;
            padding: 10px;
            font-size: 16px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            transition: 0.3s;
        }
        .btn:hover {
            background: #0056b3;
        }
        .output {
            margin-top: 20px;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>Кнопки Приложение</h2>
        <button class="btn" onclick="buttonClicked(1)">Кнопка 1</button>
        <button class="btn" onclick="buttonClicked(2)">Кнопка 2</button>
        <button class="btn" onclick="buttonClicked(3)">Кнопка 3</button>
        <p class="output" id="outputText">Нажмите кнопку</p>
    </div>

    <script>
        function buttonClicked(number) {
            document.getElementById("outputText").textContent = "Вы нажали кнопку " + number;
        }
    </script>

</body>
</html>
