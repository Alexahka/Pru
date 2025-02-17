<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Тест на психологический портрет</title>
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
        .hidden {
            display: none;
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
        <h2>Тест на психологический портрет</h2>
        <p id="questionText">Нажмите "Начать тест", чтобы узнать свой результат.</p>

        <button class="btn" id="startBtn" onclick="startTest()">Начать тест</button>

        <div id="testContainer" class="hidden">
            <p id="question"></p>
            <button class="btn" onclick="answer(true)">Да</button>
            <button class="btn" onclick="answer(false)">Нет</button>
        </div>

        <p class="output" id="resultText"></p>
    </div>

    <script>
        const questions = [
            { text: "Вы часто переживаете по пустякам?", yesPoints: 2, noPoints: 0 },
            { text: "Вам легко знакомиться с новыми людьми?", yesPoints: 2, noPoints: 1 },
            { text: "Вы предпочитаете одиночество?", yesPoints: 2, noPoints: 0 },
            { text: "В стрессовых ситуациях вы сохраняете спокойствие?", yesPoints: 1, noPoints: 2 },
            { text: "Вы любите пробовать что-то новое?", yesPoints: 2, noPoints: 1 },
        ];

        let currentQuestion = 0;
        let score = 0;

        function startTest() {
            document.getElementById("startBtn").classList.add("hidden");
            document.getElementById("testContainer").classList.remove("hidden");
            loadQuestion();
        }

        function loadQuestion() {
            if (currentQuestion < questions.length) {
                document.getElementById("question").textContent = questions[currentQuestion].text;
            } else {
                endTest();
            }
        }

        function answer(isYes) {
            if (isYes) {
                score += questions[currentQuestion].yesPoints;
            } else {
                score += questions[currentQuestion].noPoints;
            }
            currentQuestion++;
            loadQuestion();
        }

        function endTest() {
            document.getElementById("testContainer").classList.add("hidden");

            let resultText = "";
            if (score <= 3) {
                resultText = "Вы спокойный и уравновешенный человек. Продолжайте в том же духе!";
            } else if (score <= 6) {
                resultText = "Вы эмоциональный, но умеете контролировать себя. Иногда стоит расслабиться.";
            } else {
                resultText = "Вы очень чувствительный человек. Попробуйте меньше переживать!";
            }

            document.getElementById("resultText").textContent = `Ваш результат: ${score} баллов. ${resultText}`;
        }
    </script>

</body>
</html>
