
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>主軸回転数・切削速度 計算アプリ（NC旋盤対応）</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 480px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ccc;
            border-radius: 8px;
            background-color: #f9f9f9;
        }
        h2 { text-align: center; color: #333; margin-top: 0; }
        .tabs { display: flex; gap: 4px; margin-bottom: 18px; }
        .tab {
            flex: 1; width: auto; padding: 8px 2px; font-size: 12px;
            background-color: #dee2e6; color: #333; border-radius: 4px 4px 0 0;
        }
        .tab:hover { background-color: #ced4da; }
        .tab.active { background-color: #007bff; color: white; }
        .panel { display: none; }
        .panel.active { display: block; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input, select {
            width: 100%; padding: 8px; box-sizing: border-box;
            border: 1px solid #ccc; border-radius: 4px;
        }
        button {
            width: 100%; padding: 10px; background-color: #007bff; color: white;
            border: none; border-radius: 4px; font-size: 16px; cursor: pointer;
        }
        button:hover { background-color: #0056b3; }
        .result-box {
            margin-top: 20px; padding: 15px; background-color: #e9ecef;
            border-radius: 4px; text-align: center;
        }
        .result-val { font-size: 24px; font-weight: bold; color: #28a745; }
        .result-sub { margin-top: 8px; font-size: 13px; color: #555; }
        .warn { color: #dc3545; font-weight: bold; }
        .code {
            margin-top: 10px; padding: 8px; background: #212529; color: #f8f9fa;
            font-family: Consolas, monospace; font-size: 14px; text-align: left;
            border-radius: 4px; white-space: pre;
        }
        table { width: 100%; border-collapse: collapse; margin-top: 12px; font-size: 14px; }
        th, td { border: 1px solid #ccc; padding: 4px 6px; text-align: right; }
        th { background: #dee2e6; }
        td.lim { color: #dc3545; font-weight: bold; }
        .note { font-size: 12px; color: #666; margin-top: 6px; }
        .pi-tabs { display: flex; gap: 4px; }
        .pi-tab { flex: 1; width: auto; padding: 8px; font-size: 14px; background-color: #dee2e6; color: #333; }
        .pi-tab:hover { background-color: #ced4da; }
        .pi-tab.active { background-color: #007bff; color: white; }
        .mat-tabs { display: flex; flex-wrap: wrap; gap: 4px; }
        .mat-tab { position: static; flex: 1 1 30%; width: auto; padding: 8px 2px; font-size: 14px; background-color: #dee2e6; color: #333; }
        .mat-tab:hover { background-color: #ced4da; }
        .mat-tab.active { background-color: #007bff; color: white; }
        .mat-tabs { position: relative; }
        .mat-pop {
            display: none; position: absolute; bottom: calc(100% + 6px); left: 0; right: 0;
            background: #212529; color: #fff; padding: 8px 10px; border-radius: 6px;
            font-size: 12px; font-weight: normal; line-height: 1.5; text-align: left;
            white-space: normal; z-index: 10; pointer-events: none;
            box-shadow: 0 2px 8px rgba(0,0,0,.3);
        }
        .mat-pop .pt { display: block; margin-bottom: 4px; color: #ffc107; }
        .mat-pop .pr { display: block; margin-top: 4px; }
        .mat-pop small { color: #ced4da; font-size: 11px; }
        .mat-tab.show .mat-pop { display: block; }
        @media (hover: hover) { .mat-tab:hover .mat-pop { display: block; } }
        .vc-tabs { display: flex; gap: 4px; overflow-x: auto; margin-top: 8px; padding-bottom: 4px; }
        .vc-tab { flex: none; width: auto; padding: 5px 9px; font-size: 13px; background-color: #dee2e6; color: #333; }
        .vc-tab:hover { background-color: #ced4da; }
        .vc-tab.active { background-color: #007bff; color: white; }
        .pi-box { margin-top: 18px; padding-top: 12px; border-top: 1px dashed #bbb; }

        /* ===== スマホ向け調整 ===== */
        html { -webkit-text-size-adjust: 100%; scroll-padding-top: 130px; }
        * { -webkit-tap-highlight-color: transparent; }
        /* 拡大縮小（ピンチ・ダブルタップ）を無効化。スクロールは可能 */
        html, body, * { touch-action: pan-x pan-y; }
        body {
            font-size: 16px; max-width: 520px; margin: 0 auto; border: none; border-radius: 0;
            padding: 0 14px calc(28px + env(safe-area-inset-bottom));
        }
        @media (min-width: 600px) {
            body { margin: 20px auto; border: 1px solid #ccc; border-radius: 8px; padding: 0 20px 28px; }
        }
        h2 { font-size: 18px; margin: 0; padding: calc(10px + env(safe-area-inset-top)) 0 8px; }
        .sticky-head {
            position: sticky; top: 0; z-index: 30; background: #f9f9f9;
            margin: 0 -14px 14px; padding: 4px 14px 8px; border-bottom: 1px solid #dee2e6;
        }
        @media (min-width: 600px) { .sticky-head { margin: 0 -20px 14px; padding: 4px 20px 8px; } }
        .tabs { margin-bottom: 8px; }
        .tab { min-height: 48px; font-size: 13px; line-height: 1.25; white-space: normal; border-radius: 6px; }
        .pi-row { display: flex; align-items: center; gap: 10px; }
        .pi-row .lbl { font-weight: bold; font-size: 14px; white-space: nowrap; }
        .pi-row .pi-tabs { flex: 1; }
        .pi-tab { min-height: 40px; font-size: 15px; }
        label { font-size: 15px; }
        input, select { font-size: 16px; min-height: 46px; padding: 10px; }
        button { min-height: 46px; }
        .mat-tab { min-height: 46px; font-size: 15px; }
        .mat-pop { display: none !important; }
        .mat-info {
            background: #212529; color: #fff; border-radius: 8px; padding: 10px 12px;
            margin-top: 8px; font-size: 13px; line-height: 1.6;
        }
        .mat-info[hidden] { display: none; }
        .mat-info .pt { display: block; color: #ffc107; margin-bottom: 2px; }
        .mat-info .pr { display: block; margin-top: 6px; }
        .mat-info small { color: #ced4da; font-size: 12px; }
        .vc-tabs { scroll-snap-type: x proximity; -webkit-overflow-scrolling: touch; padding-bottom: 6px; }
        .vc-tab { min-height: 44px; min-width: 54px; font-size: 16px; scroll-snap-align: center; }
        .result-box { padding: 16px 12px; }
        .result-val { font-size: 28px; }
        .result-sub { font-size: 14px; }
        table { font-size: 15px; }
        th, td { padding: 6px 6px; }
        .note { font-size: 13px; }
        .secondary { background-color: #6c757d; }
        .secondary:hover { background-color: #565e64; }
        .reset-row { margin-top: 24px; }
    </style>
</head>
<body>

    <h2>回転数・切削速度 計算</h2>

    <div class="sticky-head">
    <div class="tabs">
        <button class="tab active" data-tab="rpm">回転数 N</button>
        <button class="tab" data-tab="vc">切削速度 Vc</button>
        <button class="tab" data-tab="lathe">NC旋盤 G96</button>
        <button class="tab" data-tab="cond">切削条件</button>
    </div>
        <div class="pi-row">
            <span class="lbl">円周率 π</span>
            <div class="pi-tabs">
                <button class="pi-tab active" data-pi="3">3</button>
                <button class="pi-tab" data-pi="3.14">3.14</button>
            </div>
        </div>
    </div>

    <!-- ========== タブ1: 主軸回転数 ========== -->
    <div class="panel active" id="panel-rpm">
        <div class="form-group">
            <label for="k">素材補正（被削材）:</label>
            <select id="k">
                <option value="2.0" data-range="×2.5〜×4.0" data-vc="250〜400" data-note="超高速加工が可能。溶着防止にアルミ用ブレーカーを使用">アルミ合金（A5052, A7075）</option>
                <option value="3.0" data-range="×2.0〜×3.0" data-vc="200〜300" data-note="切りくずが細かく割れる。すくい角浅めの工具を選択">黄銅・真鍮（C3604 など）</option>
                <option value="1.2" data-range="×1.0〜×1.2" data-vc="100〜120" data-note="切りくずは粉状。耐摩耗性の高い材種（K種・CVD）を使用">鋳鉄（FC250, FCD450）</option>
                <option value="1.0" data-range="×1.0" data-vc="100〜120" data-note="基準（超硬工具・中切削時）" selected>炭素鋼・基準（S45C, SS400）</option>
                <option value="0.9" data-range="×0.8〜×0.9" data-vc="80〜100" data-note="生材時の目安。少し下げる">合金鋼（SCM440, SNCM）</option>
                <option value="0.7" data-range="×0.6〜×0.7" data-vc="60〜80" data-note="加工硬化を防ぐため送りは下げすぎず一定に保つ">ステンレス鋼（SUS304, SUS316）</option>
                <option value="0.8" data-range="×0.6〜×0.8" data-vc="60〜80" data-note="粘り強いため刃先を鋭利（ポジ）にする">純銅（C1100 など）</option>
                <option value="0.5" data-range="×0.3〜×0.5" data-vc="30〜50" data-note="超硬は厳しく、基本は CBN工具 へ切り替え">高硬度材・焼入れ（HRC50以上）</option>
                <option value="0.4" data-range="×0.2〜×0.4" data-vc="20〜40" data-note="発熱が凄まじいため、低速＆高圧クーラント必須">耐熱合金・チタン（インコネル, Ti-6Al-4V）</option>
            </select>
            <label for="k-val" style="margin-top:8px;">補正係数 ×:</label>
            <input type="number" id="k-val" value="1" step="0.05" min="0.05">
            <div class="note" id="k-hint"></div>
        </div>
        <div class="form-group">
            <label>工具材質 / 被削材目安:</label>
            <div class="mat-tabs" data-select="material" data-target="vc">
                <button class="mat-tab" data-v="30" data-range="20〜40">ハイス<span class="mat-pop"><b class="pt">ハイス (HSS)　推奨切削速度 Vc</b><span class="pr"><b>軟鋼・炭素鋼</b>　20〜40 m/min<br><small>靭性が高く折れにくいが、熱に弱いため低速加工用</small></span><span class="pr"><b>ステンレス</b>　10〜20 m/min<br><small>加工硬化しやすいため低速＆しっかり送る</small></span><span class="pr"><b>アルミ合金</b>　50〜100 m/min<br><small>柔らかいためハイスでも比較的高速が可能</small></span></span></button>
                <button class="mat-tab active" data-v="130" data-range="100〜160">超硬<span class="mat-pop"><b class="pt">超硬 (Carbide)　推奨切削速度 Vc</b><span class="pr"><b>軟鋼・SS400</b>　120〜200 m/min<br><small>一般的なスチール加工の標準</small></span><span class="pr"><b>炭素鋼・S45C</b>　100〜160 m/min<br><small>汎用的な旋盤加工で最も多用される</small></span><span class="pr"><b>合金鋼・SCM440</b>　80〜140 m/min<br><small>生材（未焼入れ）の場合の目安</small></span><span class="pr"><b>ステンレス・SUS304</b>　80〜130 m/min<br><small>粘り強いため専用コーティング超硬を推奨</small></span><span class="pr"><b>鋳鉄・FC/FCD</b>　100〜180 m/min<br><small>切りくずが細かく粉状になるため高速可</small></span><span class="pr"><b>アルミ合金</b>　200〜500 m/min<br><small>切削抵抗が小さく、超高速加工が可能</small></span></span></button>
                <button class="mat-tab" data-v="200" data-range="150〜250">サーメット<span class="mat-pop"><b class="pt">サーメット　推奨切削速度 Vc</b><span class="pr"><b>炭素鋼・合金鋼</b>　150〜250 m/min<br><small>鋼の仕上げ加工に最適（キレイな鏡面仕上げ）</small></span></span></button>
                <button class="mat-tab" data-v="150" data-range="100〜200">セラミック<span class="mat-pop"><b class="pt">セラミック　推奨切削速度 Vc</b><span class="pr"><b>焼入れ鋼・高硬度材</b>　100〜200 m/min<br><small>焼き入れ後の硬い材料（HRC50以上）向け</small></span></span></button>
                <button class="mat-tab" data-v="185" data-range="120〜250">CBN<span class="mat-pop"><b class="pt">CBN　推奨切削速度 Vc</b><span class="pr"><b>焼入れ鋼・鋳鉄</b>　120〜250 m/min</span></span></button>
                <button class="mat-tab" data-v="custom">直接入力<span class="mat-pop">任意の切削速度を<br>入力できます</span></button>
            </div>
        </div>
        <div class="form-group">
            <label for="vc">切削速度 Vc (m/min):</label>
            <input type="number" id="vc" value="130" step="any">
            <div class="note" id="vc-hint">目安範囲: 100〜160 m/min（中央値を初期設定）</div>
            <div class="vc-tabs" data-target="vc"></div>
        </div>
        <div class="form-group">
            <label for="diameter">被削材の外径 D (mm):</label>
            <input type="number" id="diameter" value="100" step="any">
        </div>
        <button id="btn-rpm">計算する</button>
        <div class="result-box">
            <div>主軸回転数 N:</div>
            <div class="result-val" id="result">433 rpm</div>
            <div class="result-sub" id="result-vc-corr">補正後 Vc: 130 m/min（×1.0）</div>
            <div class="result-sub">N = 1000 × Vc × 補正係数 ÷ (π × D)</div>
        </div>
    </div>

    <!-- ========== タブ2: 切削速度（回転数から） ========== -->
    <div class="panel" id="panel-vc">
        <div class="form-group">
            <label for="vc-n">主軸回転数 N (rpm):</label>
            <input type="number" id="vc-n" value="1000" step="any">
            <div class="note">「回転数 N」タブの計算結果が自動で入ります（手入力もできます）</div>
        </div>
        <div class="form-group">
            <label for="vc-d">被削材の外径 D (mm):</label>
            <input type="number" id="vc-d" value="50" step="any">
            <div class="note">「回転数 N」タブの外径が自動で入ります（手入力もできます）</div>
        </div>
        <button id="btn-vc">計算する</button>
        <div class="result-box">
            <div>切削速度 Vc:</div>
            <div class="result-val" id="result-vc">157.1 m/min</div>
            <div class="result-sub">Vc = π × D × N ÷ 1000</div>
        </div>
    </div>

    <!-- ========== タブ3: NC旋盤（周速一定制御 G96） ========== -->
    <div class="panel" id="panel-lathe">
        <div class="form-group">
            <label>工具材質 / 被削材目安:</label>
            <div class="mat-tabs" data-select="l-material" data-target="l-vc">
                <button class="mat-tab" data-v="30" data-range="20〜40">ハイス<span class="mat-pop"><b class="pt">ハイス (HSS)　推奨切削速度 Vc</b><span class="pr"><b>軟鋼・炭素鋼</b>　20〜40 m/min<br><small>靭性が高く折れにくいが、熱に弱いため低速加工用</small></span><span class="pr"><b>ステンレス</b>　10〜20 m/min<br><small>加工硬化しやすいため低速＆しっかり送る</small></span><span class="pr"><b>アルミ合金</b>　50〜100 m/min<br><small>柔らかいためハイスでも比較的高速が可能</small></span></span></button>
                <button class="mat-tab active" data-v="130" data-range="100〜160">超硬<span class="mat-pop"><b class="pt">超硬 (Carbide)　推奨切削速度 Vc</b><span class="pr"><b>軟鋼・SS400</b>　120〜200 m/min<br><small>一般的なスチール加工の標準</small></span><span class="pr"><b>炭素鋼・S45C</b>　100〜160 m/min<br><small>汎用的な旋盤加工で最も多用される</small></span><span class="pr"><b>合金鋼・SCM440</b>　80〜140 m/min<br><small>生材（未焼入れ）の場合の目安</small></span><span class="pr"><b>ステンレス・SUS304</b>　80〜130 m/min<br><small>粘り強いため専用コーティング超硬を推奨</small></span><span class="pr"><b>鋳鉄・FC/FCD</b>　100〜180 m/min<br><small>切りくずが細かく粉状になるため高速可</small></span><span class="pr"><b>アルミ合金</b>　200〜500 m/min<br><small>切削抵抗が小さく、超高速加工が可能</small></span></span></button>
                <button class="mat-tab" data-v="200" data-range="150〜250">サーメット<span class="mat-pop"><b class="pt">サーメット　推奨切削速度 Vc</b><span class="pr"><b>炭素鋼・合金鋼</b>　150〜250 m/min<br><small>鋼の仕上げ加工に最適（キレイな鏡面仕上げ）</small></span></span></button>
                <button class="mat-tab" data-v="150" data-range="100〜200">セラミック<span class="mat-pop"><b class="pt">セラミック　推奨切削速度 Vc</b><span class="pr"><b>焼入れ鋼・高硬度材</b>　100〜200 m/min<br><small>焼き入れ後の硬い材料（HRC50以上）向け</small></span></span></button>
                <button class="mat-tab" data-v="185" data-range="120〜250">CBN<span class="mat-pop"><b class="pt">CBN　推奨切削速度 Vc</b><span class="pr"><b>焼入れ鋼・鋳鉄</b>　120〜250 m/min</span></span></button>
                <button class="mat-tab" data-v="custom">直接入力<span class="mat-pop">任意の切削速度を<br>入力できます</span></button>
            </div>
        </div>
        <div class="form-group">
            <label for="l-vc">周速 S (m/min) ※G96 Sxxx:</label>
            <input type="number" id="l-vc" value="130" step="any">
            <div class="note" id="l-vc-hint">目安範囲: 100〜160 m/min（中央値を初期設定）</div>
            <div class="vc-tabs" data-target="l-vc"></div>
        </div>
        <div class="form-group">
            <label for="l-dmax">加工開始径（最大径）Dmax (mm):</label>
            <input type="number" id="l-dmax" value="100" step="any">
        </div>
        <div class="form-group">
            <label for="l-dmin">加工終了径（最小径）Dmin (mm):</label>
            <input type="number" id="l-dmin" value="20" step="any">
        </div>
        <div class="form-group">
            <label for="l-nmax">主軸最高回転数 Nmax (rpm) ※G50 Sxxx:</label>
            <input type="number" id="l-nmax" value="3000" step="any">
        </div>
        <button id="btn-lathe">計算する</button>
        <div class="result-box">
            <div>最高回転数の制限が効く径:</div>
            <div class="result-val" id="l-result">—</div>
            <div class="result-sub" id="l-sub"></div>
            <div class="code" id="l-code"></div>
            <button id="btn-copy" class="secondary" style="margin-top:8px;">コードをコピー</button>
            <table id="l-table"></table>
            <div class="note">G96（周速一定制御）では、径が小さくなるほど回転数が上がります。G50 S で回転数の上限を必ず指定してください。</div>
        </div>
    </div>

    <!-- ========== タブ4: 切削条件（N・F・Rz 一括） ========== -->
    <div class="panel" id="panel-cond">
        <div class="form-group">
            <label for="c-d">被削材の外径 D (mm):</label>
            <input type="number" id="c-d" value="100" step="any">
        </div>
        <div class="form-group">
            <label>工具材質 / 被削材目安:</label>
            <div class="mat-tabs" data-select="c-material" data-target="c-vc">
                <button class="mat-tab" data-v="30" data-range="20〜40">ハイス<span class="mat-pop"><b class="pt">ハイス (HSS)　推奨切削速度 Vc</b><span class="pr"><b>軟鋼・炭素鋼</b>　20〜40 m/min<br><small>靭性が高く折れにくいが、熱に弱いため低速加工用</small></span><span class="pr"><b>ステンレス</b>　10〜20 m/min<br><small>加工硬化しやすいため低速＆しっかり送る</small></span><span class="pr"><b>アルミ合金</b>　50〜100 m/min<br><small>柔らかいためハイスでも比較的高速が可能</small></span></span></button>
                <button class="mat-tab active" data-v="130" data-range="100〜160">超硬<span class="mat-pop"><b class="pt">超硬 (Carbide)　推奨切削速度 Vc</b><span class="pr"><b>軟鋼・SS400</b>　120〜200 m/min<br><small>一般的なスチール加工の標準</small></span><span class="pr"><b>炭素鋼・S45C</b>　100〜160 m/min<br><small>汎用的な旋盤加工で最も多用される</small></span><span class="pr"><b>合金鋼・SCM440</b>　80〜140 m/min<br><small>生材（未焼入れ）の場合の目安</small></span><span class="pr"><b>ステンレス・SUS304</b>　80〜130 m/min<br><small>粘り強いため専用コーティング超硬を推奨</small></span><span class="pr"><b>鋳鉄・FC/FCD</b>　100〜180 m/min<br><small>切りくずが細かく粉状になるため高速可</small></span><span class="pr"><b>アルミ合金</b>　200〜500 m/min<br><small>切削抵抗が小さく、超高速加工が可能</small></span></span></button>
                <button class="mat-tab" data-v="200" data-range="150〜250">サーメット<span class="mat-pop"><b class="pt">サーメット　推奨切削速度 Vc</b><span class="pr"><b>炭素鋼・合金鋼</b>　150〜250 m/min<br><small>鋼の仕上げ加工に最適（キレイな鏡面仕上げ）</small></span></span></button>
                <button class="mat-tab" data-v="150" data-range="100〜200">セラミック<span class="mat-pop"><b class="pt">セラミック　推奨切削速度 Vc</b><span class="pr"><b>焼入れ鋼・高硬度材</b>　100〜200 m/min<br><small>焼き入れ後の硬い材料（HRC50以上）向け</small></span></span></button>
                <button class="mat-tab" data-v="185" data-range="120〜250">CBN<span class="mat-pop"><b class="pt">CBN　推奨切削速度 Vc</b><span class="pr"><b>焼入れ鋼・鋳鉄</b>　120〜250 m/min</span></span></button>
                <button class="mat-tab" data-v="custom">直接入力<span class="mat-pop">任意の切削速度を<br>入力できます</span></button>
            </div>
        </div>
        <div class="form-group">
            <label for="c-vc">切削速度 Vc (m/min):</label>
            <input type="number" id="c-vc" value="130" step="any">
            <div class="note" id="c-vc-hint">目安範囲: 100〜160 m/min（中央値を初期設定）</div>
            <div class="vc-tabs" data-target="c-vc"></div>
        </div>
        <div class="form-group">
            <label for="c-f">1回転あたりの送り量 f (mm/rev):</label>
            <input type="number" id="c-f" value="0.15" step="any">
        </div>
        <div class="form-group">
            <label for="c-r">チップ ノーズR (mm):</label>
            <select id="c-r">
                <option value="0.2">R0.2</option>
                <option value="0.4" selected>R0.4</option>
                <option value="0.8">R0.8</option>
                <option value="1.2">R1.2</option>
            </select>
        </div>
        <button id="btn-cond">一括計算する</button>
        <div class="result-box">
            <div>主軸回転数 N:</div>
            <div class="result-val" id="c-rpm">— rpm</div>
            <div class="result-sub">毎分送り速度 F (F = f × N):</div>
            <div class="result-val" id="c-rate">— mm/min</div>
            <div class="result-sub">理論表面粗さ Rz (Rz = f² ÷ 8R):</div>
            <div class="result-val" id="c-rz">— μm</div>
        </div>
    </div>

    <div class="reset-row">
        <button id="btn-reset" class="secondary">入力を初期値に戻す</button>
    </div>

    <script>
        const $ = id => document.getElementById(id);
        let piValue = 3;
        const getPi = () => piValue;
        document.querySelectorAll('.pi-tab').forEach(b => {
            b.addEventListener('click', () => {
                document.querySelectorAll('.pi-tab').forEach(x => x.classList.remove('active'));
                b.classList.add('active');
                piValue = parseFloat(b.dataset.pi);
            });
        });
        const bad = (...v) => v.some(x => isNaN(x) || x <= 0);

        // タブ切り替え
        document.querySelectorAll('.tab').forEach(t => {
            t.addEventListener('click', () => {
                document.querySelectorAll('.tab').forEach(x => x.classList.remove('active'));
                document.querySelectorAll('.panel').forEach(x => x.classList.remove('active'));
                t.classList.add('active');
                $('panel-' + t.dataset.tab).classList.add('active');
            });
        });

        // 切削速度 10単位タブ（10〜200）
        function syncVcTabs(inputId) {
            const val = parseFloat($(inputId).value);
            const box = document.querySelector('.vc-tabs[data-target="' + inputId + '"]');
            box.querySelectorAll('.vc-tab').forEach(b => {
                const on = parseFloat(b.dataset.v) === val;
                b.classList.toggle('active', on);
                if (on) box.scrollLeft = b.offsetLeft - box.clientWidth / 2 + b.offsetWidth / 2;
            });
        }
        document.querySelectorAll('.vc-tabs').forEach(box => {
            const id = box.dataset.target;
            for (let v = 10; v <= 250; v += 10) {
                const b = document.createElement('button');
                b.className = 'vc-tab';
                b.dataset.v = v;
                b.textContent = v;
                b.addEventListener('click', () => {
                    $(id).value = v;
                    syncVcTabs(id);
                });
                box.appendChild(b);
            }
            $(id).addEventListener('input', () => syncVcTabs(id));
            syncVcTabs(id);
        });

        // 材質選択 → Vc 反映 ＋ 数値の説明（もう一度押すと閉じる）
        document.querySelectorAll('.mat-tabs').forEach(box => {
            const inputId = box.dataset.target;
            const info = document.createElement('div');
            info.className = 'mat-info';
            info.hidden = true;
            box.after(info);
            box.querySelectorAll('.mat-tab').forEach(b => {
                b.addEventListener('click', () => {
                    const same = b.classList.contains('active');
                    box.querySelectorAll('.mat-tab').forEach(x => x.classList.remove('active'));
                    b.classList.add('active');
                    const pop = b.querySelector('.mat-pop');
                    if (pop && !(same && !info.hidden)) {
                        info.innerHTML = pop.innerHTML;
                        info.hidden = false;
                    } else {
                        info.hidden = true;
                    }
                    const hint = $(inputId + '-hint');
                    if (b.dataset.v !== 'custom') {
                        $(inputId).value = b.dataset.v;
                        syncVcTabs(inputId);
                        hint.innerText = '目安範囲: ' + b.dataset.range + ' m/min（中央値を初期設定）';
                    } else {
                        hint.innerText = '任意の値を入力してください';
                    }
                });
            });
        });

        // ---- タブ1: N = 1000Vc / (πD) 切り捨て ----
        function calcRpm(prop) {
            const vc = parseFloat($('vc').value);
            const d = parseFloat($('diameter').value);
            const pi = getPi();
            if (bad(vc, d, pi)) {
                $('result').innerText = "エラー: 正しい数値を入力してください";
                return;
            }
            const k = parseFloat($('k-val').value);
            if (bad(k)) { $('result').innerText = "エラー: 補正係数を確認してください"; return; }
            const vcCorr = vc * k;
            const rpm = Math.floor((1000 * vcCorr) / (d * pi));
            $('result').innerText = rpm + " rpm";
            if (prop) propagate(rpm, d, vcCorr); // 他タブへ自動反映
            $('result-vc-corr').innerText = "補正後 Vc: " + parseFloat(vcCorr.toFixed(1)) + " m/min（×" + parseFloat(k.toFixed(2)) + "）";
        }
        $('btn-rpm').addEventListener('click', () => calcRpm(true));

        // ---- タブ2: Vc = πDN / 1000 ----
        function calcVc() {
            const n = parseFloat($('vc-n').value);
            const d = parseFloat($('vc-d').value);
            const pi = getPi();
            if (bad(n, d, pi)) {
                $('result-vc').innerText = "エラー: 正しい数値を入力してください";
                return;
            }
            const vc = (pi * d * n) / 1000;
            $('result-vc').innerText = vc.toFixed(1) + " m/min";
        }
        $('btn-vc').addEventListener('click', calcVc);

        // ---- タブ3: G96 周速一定制御 ----
        function calcLathe() {
            const vc = parseFloat($('l-vc').value);
            const dmax = parseFloat($('l-dmax').value);
            const dmin = parseFloat($('l-dmin').value);
            const nmax = parseFloat($('l-nmax').value);
            const pi = getPi();
            const out = $('l-result'), sub = $('l-sub'), code = $('l-code'), table = $('l-table');

            if (bad(vc, dmax, dmin, nmax, pi) || dmin > dmax) {
                out.innerText = "エラー: 数値を確認してください（Dmin ≤ Dmax）";
                sub.innerText = ""; code.innerText = ""; table.innerHTML = "";
                return;
            }

            // 回転数が Nmax に達する径: D = 1000Vc / (π Nmax)
            const dLimit = (1000 * vc) / (pi * nmax);

            if (dLimit > dmin) {
                out.innerText = "φ" + dLimit.toFixed(1) + " mm 以下";
                sub.innerHTML = "この径より小さい部分は " + nmax + " rpm で頭打ちになり、実際の周速は設定値 " + vc + " m/min より低くなります。";
            } else {
                out.innerText = "制限なし";
                sub.innerText = "加工範囲内では " + nmax + " rpm に達しません。";
            }

            code.innerText = "G50 S" + Math.floor(nmax) + ";\nG96 S" + vc + " M03;";

            // 径ごとの回転数・実周速の表
            const steps = 6;
            let rows = "<tr><th>径 D (mm)</th><th>回転数 (rpm)</th><th>実周速 (m/min)</th></tr>";
            for (let i = 0; i <= steps; i++) {
                const d = dmax - (dmax - dmin) * i / steps;
                const nIdeal = (1000 * vc) / (d * pi);
                const limited = nIdeal > nmax;
                const n = Math.floor(limited ? nmax : nIdeal);
                const vReal = (pi * d * n) / 1000;
                rows += "<tr>"
                    + "<td>" + d.toFixed(1) + "</td>"
                    + "<td" + (limited ? ' class="lim"' : '') + ">" + n + (limited ? " ※" : "") + "</td>"
                    + "<td" + (limited ? ' class="lim"' : '') + ">" + vReal.toFixed(1) + "</td>"
                    + "</tr>";
            }
            table.innerHTML = rows;
        }
        $('btn-lathe').addEventListener('click', calcLathe);

        // ---- タブ4: 切削条件 一括計算 ----
        function calcCond() {
            const d = parseFloat($('c-d').value);
            const vc = parseFloat($('c-vc').value);
            const f = parseFloat($('c-f').value);
            const r = parseFloat($('c-r').value);
            const pi = getPi();
            if (bad(d, vc, f, r, pi)) {
                ['c-rpm', 'c-rate', 'c-rz'].forEach(id => $(id).innerText = "入力エラー");
                return;
            }
            const rpm = Math.floor((1000 * vc) / (d * pi));
            $('c-rpm').innerText = rpm + " rpm";
            $('c-rate').innerText = (f * rpm).toFixed(1) + " mm/min";
            $('c-rz').innerText = ((f * f) / (8 * r) * 1000).toFixed(2) + " μm";
        }
        $('btn-cond').addEventListener('click', calcCond);
        // 素材補正: 選択 → 係数・目安を反映
        $('k').addEventListener('change', () => {
            const o = $('k').options[$('k').selectedIndex];
            $('k-val').value = o.value;
            $('k-hint').innerText = '係数の範囲 ' + o.dataset.range + '（上限値を初期設定）／補正後の目安 ' + o.dataset.vc + ' m/min。' + o.dataset.note;
        });
        $('k').dispatchEvent(new Event('change'));
        // ---- スマホ操作の補助 ----
        // 拡大縮小を禁止（iOS Safari 対策）
        ['gesturestart', 'gesturechange', 'gestureend'].forEach(ev =>
            document.addEventListener(ev, e => e.preventDefault()));
        document.addEventListener('touchmove', e => {
            if (e.touches.length > 1) e.preventDefault();
        }, { passive: false });
        // 数値キーパッド表示・タップで全選択
        document.querySelectorAll('input[type=number]').forEach(i => {
            i.setAttribute('inputmode', 'decimal');
            i.addEventListener('focus', () => setTimeout(() => i.select(), 0));
        });
        // 計算後、結果が見える位置まで自動スクロール（固定ヘッダーの下に合わせる）
        [['btn-rpm', 'rpm'], ['btn-vc', 'vc'], ['btn-lathe', 'lathe'], ['btn-cond', 'cond']].forEach(([btn, panel]) => {
            $(btn).addEventListener('click', () => {
                if (document.activeElement) document.activeElement.blur(); // キーボードを閉じる
                requestAnimationFrame(() => {
                    const box = document.querySelector('#panel-' + panel + ' .result-box');
                    const head = document.querySelector('.sticky-head').offsetHeight;
                    const y = box.getBoundingClientRect().top + window.scrollY - head - 10;
                    window.scrollTo({ top: Math.max(0, y), behavior: 'smooth' });
                });
            });
        });
        // ---- 回転数タブの結果を他タブへ反映 ----
        function propagate(rpm, d, vcCorr) {
            $('vc-n').value = rpm;               // 切削速度タブ: 主軸回転数
            $('vc-d').value = d;                 // 切削速度タブ: 外径
            $('c-d').value = d;                  // 切削条件タブ: 外径
            $('l-dmax').value = d;               // NC旋盤タブ: 最大径
            const v = parseFloat(vcCorr.toFixed(1)); // 補正後の切削速度
            $('c-vc').value = v;                 // 切削条件タブ: Vc
            $('l-vc').value = v;                 // NC旋盤タブ: 周速 S
            syncVcTabs('c-vc');
            syncVcTabs('l-vc');
            calcVc();                            // 切削速度タブも自動計算
        }

        // ---- Enterキーで計算 ----
        document.querySelectorAll('.panel input').forEach(i => {
            i.setAttribute('enterkeyhint', 'go');
            i.addEventListener('keydown', e => {
                if (e.key === 'Enter') {
                    e.preventDefault();
                    i.closest('.panel').querySelector('button[id^="btn-"]').click();
                }
            });
        });

        // ---- G96コードのコピー ----
        $('btn-copy').addEventListener('click', async () => {
            const text = $('l-code').innerText;
            const btn = $('btn-copy');
            try {
                await navigator.clipboard.writeText(text);
                btn.innerText = 'コピーしました';
            } catch (e) {
                btn.innerText = 'コピーできませんでした';
            }
            setTimeout(() => btn.innerText = 'コードをコピー', 1800);
        });

        // ---- 入力値の自動保存・復元 ----
        const STORE_KEY = 'nc-calc-v1';
        let resetting = false;
        function saveState() {
            if (resetting) return;
            try {
                const st = { inputs: {}, selects: {}, mats: {}, pi: piValue,
                             tab: document.querySelector('.tab.active').dataset.tab };
                document.querySelectorAll('input[type=number]').forEach(i => st.inputs[i.id] = i.value);
                ['k', 'c-r'].forEach(id => st.selects[id] = $(id).value);
                document.querySelectorAll('.mat-tabs').forEach(box => {
                    st.mats[box.dataset.select] =
                        [...box.querySelectorAll('.mat-tab')].findIndex(b => b.classList.contains('active'));
                });
                localStorage.setItem(STORE_KEY, JSON.stringify(st));
            } catch (e) {}
        }
        function loadState() {
            try {
                const raw = localStorage.getItem(STORE_KEY);
                if (!raw) return false;
                const st = JSON.parse(raw);
                Object.entries(st.inputs || {}).forEach(([id, v]) => { if ($(id)) $(id).value = v; });
                ['k', 'c-r'].forEach(id => { if (st.selects && st.selects[id] !== undefined) $(id).value = st.selects[id]; });
                const o = $('k').options[$('k').selectedIndex];
                $('k-hint').innerText = '係数の範囲 ' + o.dataset.range + '（上限値を初期設定）／補正後の目安 ' + o.dataset.vc + ' m/min。' + o.dataset.note;
                document.querySelectorAll('.mat-tabs').forEach(box => {
                    const btns = box.querySelectorAll('.mat-tab');
                    const b = btns[st.mats && st.mats[box.dataset.select]];
                    if (!b) return;
                    btns.forEach(x => x.classList.remove('active'));
                    b.classList.add('active');
                    $(box.dataset.target + '-hint').innerText = b.dataset.v === 'custom'
                        ? '任意の値を入力してください'
                        : '目安範囲: ' + b.dataset.range + ' m/min（中央値を初期設定）';
                });
                if (st.pi) {
                    piValue = st.pi;
                    document.querySelectorAll('.pi-tab').forEach(x => x.classList.toggle('active', parseFloat(x.dataset.pi) === piValue));
                }
                if (st.tab) {
                    document.querySelectorAll('.tab').forEach(x => x.classList.toggle('active', x.dataset.tab === st.tab));
                    document.querySelectorAll('.panel').forEach(x => x.classList.toggle('active', x.id === 'panel-' + st.tab));
                }
                ['vc', 'l-vc', 'c-vc'].forEach(syncVcTabs);
                return true;
            } catch (e) { return false; }
        }
        ['input', 'change', 'click'].forEach(ev => document.addEventListener(ev, () => setTimeout(saveState, 0)));
        $('btn-reset').addEventListener('click', () => {
            resetting = true;
            try { localStorage.removeItem(STORE_KEY); } catch (e) {}
            location.reload();
        });

        // ---- 起動時: 保存値を復元し、結果を表示 ----
        const restored = loadState();
        calcRpm(!restored);   // 初回のみ、初期値を他タブへ反映
        calcVc(); calcLathe(); calcCond();
    </script>

</body>
</html>
