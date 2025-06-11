<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ViralCheck.ai - Analisador de Potencial Viral</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #0B0F19; color: #E0E0E0; display: flex; flex-direction: column; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; box-sizing: border-box; }
        .container { background: #1A2035; padding: 40px; border-radius: 16px; border: 1px solid #2A3352; text-align: center; width: 100%; max-width: 700px; box-shadow: 0 10px 30px rgba(0,0,0,0.2); }
        h1 { color: #FFFFFF; margin-top: 0; font-size: 2.5em; background: -webkit-linear-gradient(45deg, #3D8BFF, #AB63F2); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        p { color: #A0AEC0; }
        .input-wrapper { position: relative; }
        input[type="text"] { width: 100%; background-color: #0B0F19; color: #E0E0E0; padding: 16px 20px; border: 1px solid #2A3352; border-radius: 10px; margin-bottom: 20px; box-sizing: border-box; font-size: 16px; }
        input[type="text"]::placeholder { color: #718096; }
        button { background-image: linear-gradient(45deg, #3D8BFF, #AB63F2); color: white; padding: 16px 30px; border: none; border-radius: 10px; cursor: pointer; font-size: 18px; font-weight: bold; transition: transform 0.2s, box-shadow 0.2s; width: 100%; }
        button:hover { transform: translateY(-2px); box-shadow: 0 5px 15px rgba(171, 99, 242, 0.3); }
        button:disabled { background: #2A3352; cursor: not-allowed; transform: none; box-shadow: none; }
        .loader { border: 4px solid #2A3352; border-radius: 50%; border-top: 4px solid #AB63F2; width: 40px; height: 40px; animation: spin 1s linear infinite; margin: 20px auto; display: none; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        #results { margin-top: 30px; text-align: left; opacity: 0; transition: opacity 0.5s; }
        .score-container { margin-bottom: 25px; }
        .score-bar-bg { background-color: #0B0F19; border-radius: 10px; height: 30px; overflow: hidden; border: 1px solid #2A3352; }
        .score-bar { height: 100%; width: 0; border-radius: 10px; transition: width 1.5s cubic-bezier(0.25, 1, 0.5, 1), background-color 1s ease-out; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; box-sizing: border-box; }
        .analysis-list { list-style: none; padding: 0; }
        .analysis-list li { display: flex; align-items: flex-start; margin-bottom: 12px; padding: 12px; border-radius: 8px; border-left: 4px solid; }
        .analysis-list li.positive { background-color: rgba(61, 139, 255, 0.1); border-color: #3D8BFF; }
        .analysis-list li.warning { background-color: rgba(247, 185, 40, 0.1); border-color: #f7b928; }
        .analysis-list li::before { margin-right: 12px; font-size: 20px; line-height: 1.2; }
        .analysis-list li.positive::before { content: '🚀'; }
        .analysis-list li.warning::before { content: '💡'; }
        .keywords-container { background-color: rgba(42, 51, 82, 0.5); padding: 15px; border-radius: 8px; margin-top: 20px; }
        .keywords-container span { display: inline-block; background-color: #2A3352; color: #A0AEC0; padding: 5px 12px; margin: 5px; border-radius: 20px; font-size: 14px; }
        footer { margin-top: 40px; color: #718096; font-size: 14px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>ViralCheck.ai</h1>
        <p>A inteligência artificial que revela o potencial viral do seu vídeo do YouTube.</p>
        <div class="input-wrapper">
            <input type="text" id="videoUrl" placeholder="Cole o link do vídeo aqui...">
        </div>
        <button id="analyzeBtn" onclick="analyze()">Analisar Potencial</button>
        <div class="loader" id="loader"></div>
        <div id="results"></div>
    </div>
    <footer>Criado com IA - Projeto de Portfólio</footer>

    <script>
        // <<< INSIRA A URL DA SUA API DO PIPEDREAM AQUI >>>
        const API_ENDPOINT = 'https://eoXXXXXXXX.m.pipedream.net';

        const analyzeBtn = document.getElementById('analyzeBtn');
        const urlInput = document.getElementById('videoUrl');
        const resultsDiv = document.getElementById('results');
        const loader = document.getElementById('loader');

        async function analyze() {
            const url = urlInput.value;
            if (!url.includes('youtube.com') && !url.includes('youtu.be')) {
                alert('Por favor, insira uma URL válida do YouTube.');
                return;
            }

            analyzeBtn.disabled = true;
            analyzeBtn.textContent = 'Analisando...';
            resultsDiv.style.opacity = 0;
            loader.style.display = 'block';

            try {
                const response = await fetch(API_ENDPOINT, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ url: url }),
                });

                if (!response.ok) {
                    const errorData = await response.json();
                    throw new Error(errorData.error || `Erro ${response.status}: A análise falhou.`);
                }

                const data = await response.json();
                displayResults(data);

            } catch (error) {
                resultsDiv.innerHTML = `<p style="color: #f02849;"><strong>Erro:</strong> ${error.message}</p>`;
                resultsDiv.style.opacity = 1;
            } finally {
                loader.style.display = 'none';
                analyzeBtn.disabled = false;
                analyzeBtn.textContent = 'Analisar Potencial';
            }
        }

        function displayResults(data) {
            let scoreColor;
            if (data.score >= 75) scoreColor = 'linear-gradient(45deg, #1FDE75, #35A36A)'; // Verde
            else if (data.score >= 50) scoreColor = 'linear-gradient(45deg, #FFC837, #FF8008)'; // Amarelo
            else scoreColor = 'linear-gradient(45deg, #F05F57, #E12D39)'; // Vermelho

            let resultsHtml = `
                <div class="score-container">
                    <h3>Pontuação de Viralização</h3>
                    <div class="score-bar-bg">
                        <div class="score-bar" id="scoreBar" style="width: 0%; background: ${scoreColor};">${data.score}</div>
                    </div>
                </div>`;
            
            if (data.keywords && data.keywords.length > 0) {
                resultsHtml += `
                    <div class="keywords-container">
                        <strong>Tópicos Principais:</strong>
                        <div>${data.keywords.map(kw => `<span>${kw}</span>`).join('')}</div>
                    </div>`;
            }

            resultsHtml += '<h3>Diagnóstico Completo:</h3><ul class="analysis-list">';
            data.analysis.forEach(point => {
                resultsHtml += `<li class="${point.type}">${point.text}</li>`;
            });
            resultsHtml += '</ul>';
            
            resultsDiv.innerHTML = resultsHtml;
            resultsDiv.style.opacity = 1;

            setTimeout(() => {
                const scoreBar = document.getElementById('scoreBar');
                if(scoreBar) scoreBar.style.width = `${data.score}%`;
            }, 100);
        }
    </script>
</body>
</html>
