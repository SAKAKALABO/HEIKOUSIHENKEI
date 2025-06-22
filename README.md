<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>平行四辺形の寸法計算ツール（PDF出力対応）</title>
  <style>
    body { font-family: sans-serif; padding: 1em; max-width: 600px; margin: auto; }
    label { display: block; margin-top: 1em; }
    input { width: 100%; padding: 0.5em; font-size: 1em; }
    button { margin-top: 1em; padding: 0.7em; font-size: 1em; background-color: #4CAF50; color: white; border: none; width: 100%; }
    #result { margin-top: 2em; background: #f9f9f9; padding: 1em; border-radius: 8px; }
    canvas { width: 100%; margin-top: 2em; border: 1px solid #ccc; }
    @media (max-width: 600px) {
      body { padding: 0.5em; }
      button { font-size: 1.2em; }
    }
  </style>
</head>
<body>
  <h1>平行四辺形の寸法計算ツール</h1>

  <label for="ac">対角線 AC（mm）:</label>
  <input type="number" id="ac" placeholder="例: 1200" min="0">

  <label for="bd">対角線 BD（mm）:</label>
  <input type="number" id="bd" placeholder="例: 1000" min="0">

  <label for="ad">高さ AD（mm）:</label>
  <input type="number" id="ad" placeholder="例: 400" min="0">

  <button onclick="calculate()">計算する</button>
  <button onclick="downloadPDF()">PDF出力</button>

  <div id="result"></div>
  <canvas id="diagramCanvas" width="600" height="700"></canvas>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script>
    function calculate() {
      const ac = parseFloat(document.getElementById('ac').value);
      const bd = parseFloat(document.getElementById('bd').value);
      const ad = parseFloat(document.getElementById('ad').value);

      if (isNaN(ac) || isNaN(bd) || isNaN(ad) || ac <= 0 || bd <= 0 || ad <= 0) {
        return showWarning("すべての寸法を正しく入力してください。");
      }

      if (ad >= ac || ad >= bd) {
        return showWarning("AD が対角線より長いのは物理的に不可能です。");
      }

      const sinA = ad / ac;
      if (sinA > 1 || sinA <= 0) {
        return showWarning("物理的に成立しない三角関係です。数値を確認してください。");
      }

      const angleRad = Math.asin(sinA);
      const w = ac * Math.cos(angleRad);
      const x = (bd - w) / 2;
      const ab = Math.sqrt(ad ** 2 + x ** 2);
      const h = ad + x;

      if (x < 0 || w <= 0) {
        return showWarning("入力値から平行四辺形は構成できません。長方形または不正な形状です。");
      }

      let html = `📐 寸法計算結果：<br>
        ・対角線 AC（測定）　　　：${ac.toFixed(2)} mm<br>
        ・対角線 BD（測定）　　　：${bd.toFixed(2)} mm<br>
        ・垂直な高さ AD（測定）　：${ad.toFixed(2)} mm<br><br>
        ・①→②の幅 W（求める）　：${w.toFixed(2)} mm<br>
        ・D→④のずれ寸法 X　　　 ：${x.toFixed(2)} mm<br>
        ・①→④の元板の必要高さ H：${h.toFixed(2)} mm<br>
        ・傾いた辺 AB（切断長）　：${ab.toFixed(2)} mm`;

      if (Math.abs(x) < 1e-2) {
        html += `<br><span style='color: red; font-weight: bold;'>※ X ≒ 0 → これは長方形の可能性が高いため、平行四辺形とは認められません。</span>`;
        return showWarning("X ≒ 0 → 長方形と判断しました。平行四辺形ではありません。数値を見直してください。");
      }

      document.getElementById('result').innerHTML = html;
      drawDiagram(ac, bd, ad, w, x, ab, h);
    }

    function showWarning(message) {
      document.getElementById('result').innerHTML = `<span style='color: red; font-weight: bold;'>⚠ ${message}</span>`;
      const canvas = document.getElementById('diagramCanvas');
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    function drawDiagram(ac, bd, ad, w, x, ab, h) {
      const canvas = document.getElementById('diagramCanvas');
      const ctx = canvas.getContext('2d');
      const image = new Image();
      image.src = 'diagram.png';

      image.onload = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.drawImage(image, 0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "red";
        ctx.font = "16px sans-serif";
        ctx.fillText(`AC = ${ac.toFixed(0)} mm`, 220, 370);
        ctx.fillText(`BD = ${bd.toFixed(0)} mm`, 100, 240);
        ctx.fillText(`AD = ${ad.toFixed(0)} mm`, 510, 210);
        ctx.fillText(`W = ${w.toFixed(1)} mm`, 220, 90);
        ctx.fillText(`X = ${x.toFixed(1)} mm`, 520, 420);
        ctx.fillText(`H = ${h.toFixed(1)} mm`, 50, 330);
        ctx.fillText(`AB = ${ab.toFixed(1)} mm`, 320, 260);
      };
    }

    function downloadPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      doc.setFontSize(12);
      doc.text(document.getElementById('result').innerText, 10, 10);
      const canvas = document.getElementById('diagramCanvas');
      const imgData = canvas.toDataURL("image/png");
      doc.addImage(imgData, 'PNG', 10, 60, 180, 120);
      doc.save("parallelogram_result.pdf");
    }
  </script>
</body>
</html>
