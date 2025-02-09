<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>ドテラ目標達成シミュレーション</title>
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet" />
  <!-- Chart.js CDN -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- Annotation Plugin -->
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-annotation@3.0.1/dist/chartjs-plugin-annotation.min.js"></script>
  <style>
    /* リセット＆ベーススタイル */
    *, *::before, *::after {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    html {
      font-size: 16px;
    }
    body {
      font-family: 'Roboto', sans-serif;
      background: #f3ecf9; /* 全体をさらに薄いラベンダー系 */
      color: #333333;
      padding: 20px;
      line-height: 1.6;
    }
    h1, h2, h3 {
      margin-bottom: 10px;
      font-weight: 700;
    }
    p {
      margin-bottom: 15px;
    }
    .explanation {
      font-size: 0.8em;
      color: #666666;
    }
    .container {
      max-width: 1200px;
      margin: 0 auto;
      background: #ffffff;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      padding: 30px;
    }

    /* 入力フォームセクション */
    .input-section {
      display: flex;
      flex-wrap: wrap;
      gap: 20px;
      margin-bottom: 20px;
    }
    .input-group {
      display: flex;
      flex-direction: column;
      flex: 1 1 200px;
    }
    label {
      font-weight: 500;
      margin-bottom: 5px;
      font-size: 0.8rem;
    }
    input,
    select {
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 5px;
      font-size: 1rem;
    }
    button {
      cursor: pointer;
      padding: 10px 20px;
      background: rgba(209, 196, 233, 0.7);
      border: none;
      border-radius: 5px;
      color: #333333;
      font-size: 1rem;
      transition: background 0.3s ease;
    }
    button:hover {
      background: rgba(209, 196, 233, 0.9);
    }

    /* 目安表 */
    .reference-table {
      font-size: 0.7em;
      color: #666666;
      margin-bottom: 0;
    }
    .reference-table table {
      width: 100%;
      border-collapse: collapse;
    }
    .reference-table th,
    .reference-table td {
      border: 1px solid #ddd;
      padding: 3px;
      text-align: center;
    }

    /* 出力ブロック：目安表と出力結果 */
    .output-container {
      display: flex;
      gap: 20px;
      margin-bottom: 30px;
    }
    .reference-container {
      flex: 1;
      background: #faf9fc;
      padding: 10px;
      border-radius: 8px;
    }

    /* kpiUnified: single container for KPI & Rank achievement */
    .kpiUnified {
      flex: 1;
      padding: 15px;
      /* ベースカラーを意識しつつ色を変える（より濃いグラデーション） */
      background: linear-gradient(
        135deg,
        rgba(209, 196, 233, 0.5) 0%,
        rgba(180, 130, 200, 0.3) 100%
      );
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
      display: flex;
      flex-direction: column;
      gap: 20px;
      overflow: hidden;
    }

    /* タイトル追加 */
    .kpiUnified-title {
      text-align: center;
      font-size: 1rem;
      font-weight: 700;
      color: #4a2d68;
      margin-bottom: 10px;
    }

    .kpiUnified-row {
      display: flex;
      gap: 20px;
      align-items: flex-start; /* 修正：上揃えにする */
      justify-content: space-around;
      flex-wrap: wrap;
    }
    .kpiUnified-col {
      flex: 1;
      min-width: 120px;
      text-align: left; /* 左寄せ */
      display: flex;
      flex-direction: column;
      justify-content: flex-start; /* 上揃え */
    }
    /* 項目名は小さく */
    .kpiUnified-col h3 {
      margin-bottom: 2px; /* 隙間をより小さく */
      font-size: 0.8rem; /* 小さく */
      font-weight: 700;
      color: #4a2d68;
      text-transform: uppercase;
      letter-spacing: 1px;
    }
    /* 数値はさらに大きく＆色濃いめ */
    .kpiUnified-col p {
      margin: 0;
      line-height: 1.2;
      white-space: pre-line;
    }
    .kpiUnified-col p strong {
      font-size: 3.2em; /* 大きく */
      font-weight: 900;
      color: #3b1f57; /* より濃い紫系 */
    }

    /* 達成月の(～か月後)は小さく、太字ではない */
    .achieve-later {
      font-size: 0.7em; /* 目安表と同じぐらいの文字サイズ */
      font-weight: normal;
      color: #333;
      margin-top: -0.1em; /* 隙間を狭く */
      display: inline-block;
    }

    /* シミュレーションテーブル */
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
      background: #fff;
      border-radius: 8px;
      overflow: hidden;
    }
    thead {
      background: rgba(209, 196, 233, 0.4); /* ラベンダーをさらに薄く */
    }
    th,
    td {
      border: 1px solid #ddd;
      padding: 8px;
      text-align: right;
      font-size: 0.9rem;
    }
    th {
      font-weight: 700;
      text-align: center;
    }
    tbody tr.year-break {
      border-top: 3px solid #4a2d68;
    }
    tbody tr:nth-child(even) {
      background: #f4edf7;
    }

    /* チャート表示エリア */
    .charts-wrapper {
      display: flex;
      flex-wrap: wrap;
      gap: 30px;
      justify-content: space-between;
      margin-bottom: 30px;
    }
    .chart-container {
      flex: 1;
      min-width: 300px;
      max-width: 600px;
      background: #fff;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
    }
    canvas {
      width: 100% !important;
      height: auto !important;
    }

    /* レスポンシブ対応 */
    @media (max-width: 768px) {
      .input-section,
      .charts-wrapper,
      .output-container {
        flex-direction: column;
      }
      .kpiUnified-row {
        flex-direction: column;
      }
      .kpiUnified-col {
        text-align: center;
        align-items: center;
      }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>ドテラ目標達成シミュレーション</h1>
    <p class="explanation">
      下記フォームで【現在のPV】、【今年の目標Rank】、【目標PV】に加え、「新規登録者目標数(月平均)」
      （＝毎月の新規登録者数期待値）と【目標設定日】を入力すると、ファネル型のシミュレーション（各月24カ月分）・グラフ、
      さらにシミュレーションのPV予測（OV）が目標PVに初めて到達する月を自動判定して表示します。
    </p>

    <!-- 入力フォーム -->
    <div class="input-section">
      <div class="input-group">
        <label for="currentPV">現在の PV</label>
        <input type="number" id="currentPV" value="3000" />
      </div>
      <div class="input-group">
        <label for="yearGoal">今年の目標 Rank</label>
        <select id="yearGoal">
          <option value="EXECUTIVE">EXECUTIVE</option>
          <option value="ELITE">ELITE</option>
          <option value="PREMIERE">PREMIERE</option>
          <option value="SILVER" selected>SILVER</option>
          <option value="GOLD">GOLD</option>
          <option value="PLATINUM">PLATINUM</option>
          <option value="DIAMOND">DIAMOND</option>
          <option value="BLUE DIAMOND">BLUE DIAMOND</option>
          <option value="PRESIDENTIAL">PRESIDENTIAL</option>
        </select>
      </div>
      <div class="input-group">
        <label for="goalPV">目標 PV</label>
        <input type="number" id="goalPV" value="9000" />
      </div>
      <div class="input-group">
        <label for="monthlyTarget">新規登録者目標数(月平均)</label>
        <input type="number" id="monthlyTarget" value="7" step="0.1" />
      </div>
      <div class="input-group">
        <label for="targetDate">目標設定日</label>
        <input type="date" id="targetDate" value="2025-01-01" />
      </div>
      <div class="input-group" style="align-self: flex-end;">
        <button id="updateBtn">更新</button>
      </div>
    </div>

    <!-- 出力ブロック： 目安表 + KPI&目標Rank達成 -->
    <div class="output-container">
      <div class="reference-container">
        <strong>ランク別目標値 目安表</strong>
        <table class="reference-table">
          <thead>
            <tr>
              <th>Rank</th>
              <th>PV</th>
              <th>登録者</th>
              <th>目安月収$</th>
              <th>目安月収¥</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>EXECUTIVE</td>
              <td>2,000</td>
              <td>13</td>
              <td></td>
              <td></td>
            </tr>
            <tr>
              <td>ELITE</td>
              <td>3,000</td>
              <td>20</td>
              <td>$300</td>
              <td>¥39,000</td>
            </tr>
            <tr>
              <td>PREMIERE</td>
              <td>5,000</td>
              <td>27</td>
              <td>$600</td>
              <td>¥78,000</td>
            </tr>
            <tr>
              <td>SILVER</td>
              <td>9,000</td>
              <td>60</td>
              <td>$2,125</td>
              <td>¥276,250</td>
            </tr>
            <tr>
              <td>GOLD</td>
              <td>12,000</td>
              <td>80</td>
              <td>$4,667</td>
              <td>¥606,710</td>
            </tr>
            <tr>
              <td>PLATINUM</td>
              <td>27,000</td>
              <td>180</td>
              <td>$8,750</td>
              <td>¥1,137,500</td>
            </tr>
            <tr>
              <td>DIAMOND</td>
              <td>36,000</td>
              <td>240</td>
              <td>$10,683</td>
              <td>¥1,388,790</td>
            </tr>
            <tr>
              <td>BLUE DIAMOND</td>
              <td>60,000</td>
              <td>400</td>
              <td>$36,500</td>
              <td>¥4,745,000</td>
            </tr>
            <tr>
              <td>PRESIDENTIAL</td>
              <td>162,000</td>
              <td>1,080</td>
              <td>$106,833</td>
              <td>¥13,888,290</td>
            </tr>
          </tbody>
        </table>
      </div>
      <!-- 統合された1枠 -->
      <div class="kpiUnified" id="kpiUnified">
        <!-- タイトル -->
        <div class="kpiUnified-title">あなたの目標達成に向けた数値</div>

        <!-- 上段：KPI 出力 -->
        <div class="kpiUnified-row" id="kpiResults">
          <!-- Filled by JS -->
        </div>
        <hr style="border:none; border-top:1px solid #ccc;">
        <!-- 下段：目標Rank達成 -->
        <div class="kpiUnified-row" id="goalMonthResult">
          <!-- Filled by JS -->
        </div>
      </div>
    </div>

    <!-- チャート表示エリア -->
    <div class="charts-wrapper">
      <div class="chart-container">
        <h2>1年予測</h2>
        <canvas id="chartYear1"></canvas>
      </div>
      <div class="chart-container">
        <h2>2年予測</h2>
        <canvas id="chartYear2"></canvas>
      </div>
    </div>

    <!-- KPI シミュレーション テーブル（24カ月分） -->
    <h2>KPI シミュレーション (月次)</h2>
    <table id="kpiTable">
      <thead>
        <tr>
          <th>月</th>
          <th>サンプル配布数</th>
          <th>講座参加数</th>
          <th>新規登録者数</th>
          <th>LRP継続者数</th>
          <th>User数</th>
          <th>シェアラー</th>
          <th>ビルダー育成数</th>
          <th>OV (PV予測)</th>
        </tr>
      </thead>
      <tbody>
        <!-- シミュレーションデータは JavaScript で生成 -->
      </tbody>
    </table>
  </div>

  <script>
    /************************************************
     * シミュレーションロジック（KPI算出）
     *
     * 【ターゲット値】
     *  サンプル配布数: 30
     *  講座参加数: floor(newReg * (21/11)) ※目標率70%
     *  新規登録者数（メンバー登録数）: newReg （目標率50% 目標値11の場合）
     *  定期購入契約者数（LRP継続者数）: floor(newReg * (8/11)) ※目標率80%
     *  ユーザー: 同じ
     *  シェアラー: floor(newReg * (4/11)) ※目標率50%
     *  ビルダー育成数: floor(newReg * (1/11)) ※目標率20%
     *
     * 各月の数値は、入力された「新規登録者目標数(月平均)」（newReg）をもとに算出する。
     *
     * OV（PV予測）の算出：
     *  ・月0: OV = currentPV + (新規登録者数 * 150)
     *  ・月1以降: OV = currentPV + ((新規登録者数 + 累積LRP継続者数) * 150)
     *    ※累積LRP継続者数は、各月で floor(newReg * (8/11)) を累積
     *  OVの表示はtoLocaleString()でカンマ区切り
     *
     * ※各行のラベルは、入力された【目標設定日】を基準に「YYYY年M月」として表示。
     *    年切替時は、その行に "year-break" クラスを付与して視覚的に区切る。
     ************************************************/
    function recalcKpiData() {
      const newRegInput = parseFloat(document.getElementById("monthlyTarget").value) || 7;
      const newReg = Math.floor(newRegInput);
      let currentPV = parseFloat(document.getElementById("currentPV").value);
      if (isNaN(currentPV)) currentPV = 3000;

      // 入力された目標設定日を基準に
      const targetDateStr = document.getElementById("targetDate").value;
      let startDate = targetDateStr ? new Date(targetDateStr) : new Date();

      // KPI算出：
      const sampleMonthly = Math.floor(newReg * (30 / 11));
      const lectureMonthly = Math.floor(newReg * (21 / 11));
      // 新規登録者数 = newReg
      const lrpMonthly = Math.floor(newReg * (8 / 11));
      const sharer = Math.floor(newReg * (4 / 11));
      const builder = Math.floor(newReg * (1 / 11));

      // OVの単位は150PV
      const ovMultiplier = 150;
      let simData = [];
      let cumLRP = 0;
      let prevYear = startDate.getFullYear();

      for (let m = 0; m < 24; m++) {
        let simDate = new Date(
          startDate.getFullYear(),
          startDate.getMonth() + m,
          startDate.getDate()
        );
        let monthLabel =
          simDate.getFullYear() + "年" + (simDate.getMonth() + 1) + "月";
        let newYearBreak = false;
        if (m > 0 && simDate.getFullYear() !== prevYear) {
          newYearBreak = true;
          prevYear = simDate.getFullYear();
        }
        let OV;
        if (m === 0) {
          // 初月: OV = currentPV + (newReg * 150)
          OV = currentPV + newReg * ovMultiplier;
        } else {
          cumLRP += lrpMonthly;
          // 以降: OV = currentPV + ((newReg + cumLRP) * 150)
          OV = currentPV + (newReg + cumLRP) * ovMultiplier;
        }
        const OVFormatted = OV.toLocaleString();

        let row = {
          monthLabel: monthLabel,
          sample: sampleMonthly,
          lecture: lectureMonthly,
          newReg: newReg,
          lrp: m === 0 ? "" : Math.floor(cumLRP),
          user: m === 0 ? "" : Math.floor(cumLRP),
          sharer: sharer,
          builder: builder,
          OV: OVFormatted,
          newYearBreak: newYearBreak,
        };
        simData.push(row);
      }
      return simData;
    }

    /************************************************
     * KPI 出力ブロックの更新
     *
     * ・年間新規登録者数 = newReg × 12
     * ・年間 ビルダー育成数 = floor(newReg * (1/11) * 12)
     * ・LRP継続率は固定 80%
     * ・【目標Rank達成】は、シミュレーションのOVで初めて【goalPV】以上となる月を判定
     ************************************************/
    function updateKpiResults() {
      const simData = recalcKpiData();
      const newRegInput = parseFloat(document.getElementById("monthlyTarget").value) || 7;
      const newReg = Math.floor(newRegInput);
      const annualNew = Math.floor(newReg * 12);
      const annualBuilder = Math.floor(newReg * (1 / 11) * 12);

      const goalPV = parseFloat(document.getElementById("goalPV").value) || 9000;
      let achieveMonth = "未達";
      let achieveMonthLine1 = "";
      let achieveMonthLine2 = "";

      for (let i = 0; i < simData.length; i++) {
        const OV = parseInt(simData[i].OV.replace(/,/g, ""));
        if (OV >= goalPV) {
          // 改行させたい: 例) 2025年8月  \n  (8カ月後)
          achieveMonthLine1 = simData[i].monthLabel;
          achieveMonthLine2 = "(" + (i + 1) + "カ月後)";
          achieveMonth = achieveMonthLine1 + "\n" + achieveMonthLine2;
          break;
        }
      }

      // 上段: KPI 出力
      const kpiResults = document.getElementById("kpiResults");
      kpiResults.innerHTML = `
        <div class="kpiUnified-col">
          <h3>新規登録者</h3>
          <p><strong>${annualNew}</strong> 名/年</p>
        </div>
        <div class="kpiUnified-col">
          <h3>ビルダー育成数</h3>
          <p><strong>${annualBuilder}</strong> 名/年</p>
        </div>
        <div class="kpiUnified-col">
          <h3>LRP継続率</h3>
          <p><strong>80%</strong></p>
        </div>
      `;

      // 下段: 目標Rank達成
      const goalMonthResult = document.getElementById("goalMonthResult");
      const goalRank = document.getElementById("yearGoal").value;

      if (achieveMonth === "未達") {
        goalMonthResult.innerHTML = `
          <div class="kpiUnified-col">
            <h3>目標Rank</h3>
            <p><strong>${goalRank}</strong></p>
          </div>
          <div class="kpiUnified-col">
            <h3>達成月</h3>
            <p><strong>${achieveMonth}</strong></p>
          </div>
        `;
      } else {
        // line1 => bold, line2 => small normal text
        goalMonthResult.innerHTML = `
          <div class="kpiUnified-col">
            <h3>目標Rank</h3>
            <p><strong>${goalRank}</strong></p>
          </div>
          <div class="kpiUnified-col">
            <h3>達成月</h3>
            <p style="display: flex; flex-direction: column; line-height:1.1;">
              <strong style="margin-bottom:0.1em;">${achieveMonthLine1}</strong>
              <span class="achieve-later">${achieveMonthLine2}</span>
            </p>
          </div>
        `;
      }
    }

    /************************************************
     * KPI シミュレーションテーブルの描画
     ************************************************/
    function populateKpiTable(simData) {
      const tbody = document.getElementById("kpiTable").querySelector("tbody");
      tbody.innerHTML = "";
      simData.forEach((row) => {
        const tr = document.createElement("tr");
        if (row.newYearBreak) {
          tr.classList.add("year-break");
        }
        let cells = [
          row.monthLabel,
          row.sample,
          row.lecture,
          row.newReg,
          row.lrp,
          row.user,
          row.sharer,
          row.builder,
          row.OV,
        ];
        cells.forEach((cell) => {
          const td = document.createElement("td");
          td.textContent = cell;
          tr.appendChild(td);
        });
        tbody.appendChild(tr);
      });
    }

    /************************************************
     * Chart.js を使ったグラフ描画
     * （シミュレーション表から抜粋して、「新規登録者数」、「LRP継続者数」、「OV (PV予測)」を描画）
     *
     * ※x軸ラベルは、最初の行または年切替行はフル表示、それ以外は「M月」部分のみと表示
     ************************************************/
    let chart1, chart2;
    function renderCharts(simData) {
      Chart.register(window["chartjs-plugin-annotation"]);
      if (chart1) chart1.destroy();
      if (chart2) chart2.destroy();

      // 目標PVを取得
      const goalPV = parseFloat(document.getElementById("goalPV").value) || 9000;

      // 1年目：最初の12カ月
      const dataYear1 = simData.slice(0, 12);
      const labelsYear1 = dataYear1.map((row, i) => {
        if (i === 0 || row.newYearBreak) {
          return row.monthLabel;
        } else {
          return row.monthLabel.split("年")[1];
        }
      });
      const newRegYear1 = dataYear1.map((row) => parseInt(row.newReg || 0));
      const lrpYear1 = dataYear1.map((row) => parseInt(row.lrp || 0));
      const OVYear1 = dataYear1.map((row) => parseInt(row.OV.replace(/,/g, "")));

      // 目標達成月のindexを探す
      let achieveIndexYear1 = -1;
      for (let i = 0; i < OVYear1.length; i++) {
        if (OVYear1[i] >= goalPV) {
          achieveIndexYear1 = i;
          break;
        }
      }

      // 達成月の強調色をベースカラーの傾向（薄いラベンダー～ピンク）に合わせる
      const achieveBorderColor = "rgba(200,100,200,0.8)";
      const achieveBgColor = "rgba(200,100,200,0.2)";

      const ctx1 = document.getElementById("chartYear1").getContext("2d");
      chart1 = new Chart(ctx1, {
        type: "bar",
        data: {
          labels: labelsYear1,
          datasets: [
            {
              label: "新規登録者数",
              data: newRegYear1,
              backgroundColor: "rgba(209,196,233,0.7)",
              stack: "stack1",
            },
            {
              label: "LRP継続者数",
              data: lrpYear1,
              backgroundColor: "rgba(209,196,233,0.9)",
              stack: "stack1",
            },
            {
              label: "OV (PV予測)",
              data: OVYear1,
              type: "line",
              yAxisID: "yOv",
              borderColor: "rgba(150,100,180,0.8)",
              backgroundColor: "rgba(150,100,180,0.2)",
              fill: true,
              tension: 0.3,
            },
          ],
        },
        options: {
          responsive: true,
          interaction: { mode: "index", intersect: false },
          plugins: {
            annotation: {
              annotations: (() => {
                // 達成があった場合のみアノテーション
                if (achieveIndexYear1 === -1) return {};
                return {
                  line1: {
                    type: "line",
                    xMin: achieveIndexYear1,
                    xMax: achieveIndexYear1,
                    borderColor: achieveBorderColor,
                    borderWidth: 2,
                    label: {
                      enabled: true,
                      position: "top",
                      content: "達成",
                      backgroundColor: achieveBgColor,
                      color: achieveBorderColor,
                    },
                  },
                };
              })(),
            },
          },
          scales: {
            x: { stacked: true },
            y: {
              stacked: true,
              title: { display: true, text: "数値" },
            },
            yOv: {
              position: "right",
              title: { display: true, text: "PV (OV)" },
              grid: { drawOnChartArea: false },
            },
          },
        },
      });

      // 2年目：残り12カ月
      const dataYear2 = simData.slice(12, 24);
      const labelsYear2 = dataYear2.map((row, i) => {
        if (i === 0 || row.newYearBreak) {
          return row.monthLabel;
        } else {
          return row.monthLabel.split("年")[1];
        }
      });
      const newRegYear2 = dataYear2.map((row) => parseInt(row.newReg || 0));
      const lrpYear2 = dataYear2.map((row) => parseInt(row.lrp || 0));
      const OVYear2 = dataYear2.map((row) => parseInt(row.OV.replace(/,/g, "")));

      // 達成月のindexを探す（ただし、1年目で達成していたら表示しない）
      let achieveIndexYear2 = -1;
      // もし1年目で達成していなければ、2年目の中で探す
      if (achieveIndexYear1 === -1) {
        for (let i = 0; i < OVYear2.length; i++) {
          if (OVYear2[i] >= goalPV) {
            achieveIndexYear2 = i;
            break;
          }
        }
      }

      const ctx2 = document.getElementById("chartYear2").getContext("2d");
      chart2 = new Chart(ctx2, {
        type: "bar",
        data: {
          labels: labelsYear2,
          datasets: [
            {
              label: "新規登録者数",
              data: newRegYear2,
              backgroundColor: "rgba(209,196,233,0.7)",
              stack: "stack2",
            },
            {
              label: "LRP継続者数",
              data: lrpYear2,
              backgroundColor: "rgba(209,196,233,0.9)",
              stack: "stack2",
            },
            {
              label: "OV (PV予測)",
              data: OVYear2,
              type: "line",
              yAxisID: "yOv2",
              borderColor: "rgba(150,100,180,0.8)",
              backgroundColor: "rgba(150,100,180,0.2)",
              fill: true,
              tension: 0.3,
            },
          ],
        },
        options: {
          responsive: true,
          interaction: { mode: "index", intersect: false },
          plugins: {
            annotation: {
              annotations: (() => {
                if (achieveIndexYear2 === -1) return {};
                return {
                  line2: {
                    type: "line",
                    xMin: achieveIndexYear2,
                    xMax: achieveIndexYear2,
                    borderColor: achieveBorderColor,
                    borderWidth: 2,
                    label: {
                      enabled: true,
                      position: "top",
                      content: "達成",
                      backgroundColor: achieveBgColor,
                      color: achieveBorderColor,
                    },
                  },
                };
              })(),
            },
          },
          scales: {
            x: { stacked: true },
            y: {
              stacked: true,
              title: { display: true, text: "数値" },
            },
            yOv2: {
              position: "right",
              title: { display: true, text: "PV (OV)" },
              grid: { drawOnChartArea: false },
            },
          },
        },
      });
    }

    /************************************************
     * 更新ボタン押下時の処理
     ************************************************/
    function onUpdate() {
      updateKpiResults();
      const simData = recalcKpiData();
      populateKpiTable(simData);
      renderCharts(simData);
    }

    /************************************************
     * ページ読み込み時の初期処理
     ************************************************/
    window.addEventListener("DOMContentLoaded", () => {
      document.getElementById("updateBtn").addEventListener("click", onUpdate);
      onUpdate();
    });
  </script>
</body>
</html>
