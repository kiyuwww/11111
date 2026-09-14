<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Тестирование с контролем фокуса</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f9;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            user-select: none; /* Запрещаем выделение текста (защита от копирования) */
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            max-width: 500px;
            width: 100%;
            text-align: center;
        }
        .hidden { display: none !important; }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 16px;
            cursor: pointer;
            border-radius: 4px;
            margin-top: 15px;
        }
        button:hover { background-color: #45a049; }
        .quiz-option {
            display: block;
            margin: 10px 0;
            text-align: left;
            padding: 10px;
            background: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 4px;
            cursor: pointer;
        }
        .quiz-option:hover { background: #f0f0f0; }
        #warning-msg { color: red; font-weight: bold; margin-top: 15px; }
    </style>
</head>
<body>

<div class="container" id="start-block">
    <h2>Внимание: Контрольное тестирование</h2>
    <p>Для начала теста необходимо перейти в полноэкранный режим. Выход из полноэкранного режима или переключение вкладок будет зафиксировано как нарушение!</p>
    <button onclick="startTest()">Начать тест</button>
</div>

<div class="container hidden" id="quiz-block">
    <h3 id="question-text">Вопрос 1: Сколько будет 2 + 2 * 2?</h3>
    <div id="options-container">
        <!-- Варианты ответов сгенерируются кодом -->
    </div>
    <div id="warning-msg"></div>
</div>

<div class="container hidden" id="result-block">
    <h2>Тест завершен</h2>
    <p id="score-text"></p>
    <p id="violations-text" style="color: red;"></p>
</div>

<script>
    let violationsCount = 0;
    let currentQuestion = 0;
    let score = 0;

    const questions = [
        { q: "Сколько будет 2 + 2 * 2?", answers: ["4", "6", "8"], correct: 1 },
        { q: "Какой язык программирования «оживляет» страницы в браузере?", answers: ["HTML", "CSS", "JavaScript"], correct: 2 },
        { q: "Что из этого не является операционной системой?", answers: ["Linux", "Python", "Windows"], correct: 1 }
    ];

    // Блокируем клавишу F11 и контекстное меню (правую кнопку мыши)
    window.addEventListener('keydown', function(e) {
        if (e.key === 'F11') {
            e.preventDefault(); // Запрещаем стандартный F11
            alert("Использование F11 запрещено правилами тестирования!");
        }
    });

    document.addEventListener('contextmenu', e => e.preventDefault()); // Блок правой кнопки мыши

    // Функция запуска теста и перехода в Fullscreen
    function startTest() {
        let elem = document.documentElement;
        if (elem.requestFullscreen) {
            elem.requestFullscreen();
        } else if (elem.mozRequestFullScreen) { /* Firefox */
            elem.mozRequestFullScreen();
        } else if (elem.webkitRequestFullscreen) { /* Chrome, Safari & Opera */
            elem.webkitRequestFullscreen();
        } else if (elem.msRequestFullscreen) { /* IE/Edge */
            elem.msRequestFullscreen();
        }

        document.getElementById('start-block').classList.add('hidden');
        document.getElementById('quiz-block').classList.remove('hidden');
        loadQuestion();
    }

    // Загрузка вопроса
    function loadQuestion() {
        if (currentQuestion >= questions.length) {
            showResults();
            return;
        }
        let qData = questions[currentQuestion];
        document.getElementById('question-text').innerText = `Вопрос ${currentQuestion + 1}: ${qData.q}`;
        
        let optionsHtml = '';
        qData.answers.forEach((ans, index) => {
            optionsHtml += `<div class="quiz-option" onclick="checkAnswer(${index})">${ans}</div>`;
        });
        document.getElementById('options-container').innerHTML = optionsHtml;
    }

    // Проверка ответа
    function checkAnswer(selectedIndex) {
        if (selectedIndex === questions[currentQuestion].correct) {
            score++;
        }
        currentQuestion++;
        loadQuestion();
    }

    // Вывод результатов
    function showResults() {
        document.getElementById('quiz-block').classList.add('hidden');
        document.getElementById('result-block').classList.remove('hidden');
        document.getElementById('score-text').innerText = `Вы правильно ответили на ${score} из ${questions.length} вопросов.`;
        document.getElementById('violations-text').innerText = `Зафиксировано нарушений (выходов из фокуса): ${violationsCount}`;
        
        // Попытка выйти из полноэкранного режима в конце теста
        if (document.exitFullscreen) document.exitFullscreen().catch(() => {});
    }

    // Слежка за переключением вкладок (Page Visibility API)
    document.addEventListener("visibilitychange", function() {
        if (document.hidden && currentQuestion < questions.length) {
            registerViolation("Вы свернули браузер или открыли другую вкладку!");
        }
    });

    // Слежка за выходом из полноэкранного режима
    document.addEventListener('fullscreenchange', handleFullscreenExit);
    document.addEventListener('webkitfullscreenchange', handleFullscreenExit);
    document.addEventListener('mozfullscreenchange', handleFullscreenExit);
    document.addEventListener('MSFullscreenChange', handleFullscreenExit);

    function handleFullscreenExit() {
        if (!document.fullscreenElement && !document.webkitIsFullScreen && !document.mozFullScreen && !document.msFullscreenElement) {
            if (currentQuestion < questions.length) {
                registerViolation("Вы вышли из полноэкранного режима!");
            }
        }
    }

    // Регистрация нарушения
    function registerViolation(reason) {
        violationsCount++;
        let warnDiv = document.getElementById('warning-msg');
        warnDiv.innerText = `Внимание! Нарушение #${violationsCount}: ${reason}`;
        
        // Тут можно сразу завершать тест, если нарушений больше N
        if(violationsCount >= 3) {
            alert("Тест аннулирован из-за многочисленных нарушений!");
            showResults();
        }
    }
</script>

</body>
</html>
