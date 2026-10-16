<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WithCat引継ぎアプリ（写真・名簿管理対応）</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Inter', 'Hiragino Kaku Gothic ProN', sans-serif; -webkit-tap-highlight-color: transparent; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 min-h-screen pb-24">

    <!-- ヘッダー -->
    <header class="bg-teal-700 text-white shadow-md sticky top-0 z-50">
        <div class="max-w-md mx-auto px-3 py-2.5 flex justify-between items-center">
            <div class="flex items-center gap-2">
                <div class="bg-white/20 p-1.5 rounded-lg">
                    <i class="fa-solid fa-cat text-teal-100 text-base"></i>
                </div>
                <div>
                    <h1 class="font-bold text-sm sm:text-base leading-tight">WithCat引継ぎアプリ</h1>
                    <p class="text-[10px] text-teal-100" id="current-date-display">写真・名簿管理対応版</p>
                </div>
            </div>
            <div class="flex gap-1.5">
                <button onclick="resetDailyChecks()" class="bg-teal-600 hover:bg-teal-500 border border-teal-500 text-[11px] px-2 py-1.5 rounded-lg font-medium transition flex items-center gap-1" title="日次チェックを朝の状態に戻す">
                    <i class="fa-solid fa-rotate-right"></i>日次更新
                </button>
                <button onclick="copyLineReport()" class="bg-emerald-600 hover:bg-emerald-500 text-[11px] px-2.5 py-1.5 rounded-lg font-bold transition flex items-center gap-1 shadow-xs" title="LINEにそのまま貼れる形式でコピー">
                    <i class="fa-solid fa-clipboard-user"></i>LINE用
                </button>
            </div>
        </div>
        <!-- リアルタイム集計バー -->
        <div class="bg-teal-800/90 text-white text-[11px] py-1.5 px-3 border-t border-teal-600/50">
            <div class="max-w-md mx-auto flex justify-around items-center">
                <span>在籍: <strong id="stat-total">0</strong>頭</span>
                <span class="text-amber-200"><i class="fa-solid fa-triangle-exclamation mr-0.5"></i>様子見: <strong id="stat-care">0</strong>頭</span>
                <span class="text-emerald-200"><i class="fa-solid fa-bowl-food mr-0.5"></i>ごはん確認: <strong id="stat-fed">0</strong>頭</span>
            </div>
        </div>
    </header>

    <!-- ナビゲーションタブ -->
    <nav class="bg-white shadow-sm border-b border-slate-200 sticky top-[76px] z-40">
        <div class="max-w-md mx-auto flex justify-around text-xs font-bold">
            <button onclick="switchTab('handover')" id="tab-handover" class="flex-1 py-3 text-center border-b-2 border-teal-600 text-teal-700 transition flex items-center justify-center gap-1">
                <i class="fa-solid fa-clipboard-user"></i> トップ
            </button>
            <button onclick="switchTab('cats')" id="tab-cats" class="flex-1 py-3 text-center border-b-2 border-transparent text-slate-500 hover:text-teal-700 transition flex items-center justify-center gap-1">
                <i class="fa-solid fa-paw"></i> お世話カルテ
            </button>
            <button onclick="switchTab('roster')" id="tab-roster" class="flex-1 py-3 text-center border-b-2 border-transparent text-slate-500 hover:text-teal-700 transition flex items-center justify-center gap-1">
                <i class="fa-solid fa-address-book"></i> 在籍名簿管理
            </button>
        </div>
    </nav>

    <!-- トースト通知 -->
    <div id="toast" class="fixed bottom-5 left-1/2 -translate-x-1/2 bg-slate-800 text-white text-xs px-4 py-2.5 rounded-full shadow-lg z-50 hidden transition-opacity">
        保存しました
    </div>

    <main class="max-w-md mx-auto p-3 space-y-3">
        
        <!-- ================= 1. トップ・引継ぎタブ ================= -->
        <section id="view-handover" class="space-y-3">
            
            <!-- 施設・ルーティン確認バー -->
            <div class="bg-white p-3 rounded-xl shadow-xs border border-slate-200 space-y-2">
                <div class="flex justify-between items-center">
                    <h2 class="font-bold text-slate-700 text-xs flex items-center gap-1.5">
                        <i class="fa-solid fa-building-shield text-teal-600"></i> 施設・物資クイックチェック
                    </h2>
                    <button onclick="openFacilityModal()" class="text-[11px] text-teal-700 hover:underline font-semibold flex items-center gap-0.5">
                        <i class="fa-solid fa-pen text-[10px]"></i>メモ編集
                    </button>
                </div>
                <div class="grid grid-cols-4 gap-1.5 text-[11px] font-bold" id="facility-toggles">
                    <!-- 動的挿入 -->
                </div>
                <p class="text-[11px] text-slate-600 bg-slate-50 px-2.5 py-1.5 rounded-lg border border-slate-100" id="facility-summary-text">
                    <!-- 施設メモ -->
                </p>
            </div>

            <!-- 要注意・ケア対象猫のピックアップ -->
            <div class="bg-amber-50/80 p-3 rounded-xl border border-amber-200 space-y-2">
                <div class="flex justify-between items-center">
                    <h2 class="font-bold text-amber-900 text-xs flex items-center gap-1.5">
                        <i class="fa-solid fa-bell text-amber-600 animate-pulse"></i> 本日の様子見・ケアが必要な猫
                    </h2>
                    <button onclick="switchTab('cats'); setFilter('care');" class="text-[11px] text-amber-800 font-bold hover:underline">
                        全て見る <i class="fa-solid fa-chevron-right text-[10px]"></i>
                    </button>
                </div>
                <div id="priority-cats-preview" class="flex gap-2 overflow-x-auto no-scrollbar pb-1">
                    <!-- 動的挿入 -->
                </div>
            </div>

            <!-- シフト申し送りフォーム -->
        <div class="bg-white p-3.5 rounded-xl shadow-xs border border-slate-200">
            <div class="flex justify-between items-center mb-2.5">
                <h2 class="font-bold text-slate-700 flex items-center gap-1.5 text-xs">
                    <i class="fa-solid fa-comments text-teal-600"></i> シフト申し送り・連絡ボード
                </h2>
                <!-- AIアシスタント起動ボタン -->
                <button type="button" onclick="openAiModal()" class="bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-[11px] font-bold px-2 py-1 rounded-lg border border-indigo-200 transition flex items-center gap-1">
                    <i class="fa-solid fa-wand-magic-sparkles"></i> AI文章サポート
                </button>
            </div>
            <form id="handover-form" onsubmit="addHandover(event)" class="space-y-2.5">
                    <div class="grid grid-cols-12 gap-2">
                        <select id="handover-shift" class="col-span-4 text-xs px-2 py-2 rounded-lg border border-slate-200 bg-slate-50 font-semibold focus:outline-none focus:ring-2 focus:ring-teal-500">
                            <option value="朝シフト">☀️ 朝番</option>
                            <option value="昼シフト">🌤️ 昼番</option>
                            <option value="夕シフト">🌙 夕番</option>
                            <option value="その他">🐾 その他</option>
                        </select>
                        <input type="text" id="poster-name" placeholder="担当者名 (例: 佐藤)" required
                            class="col-span-4 text-xs px-2.5 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-teal-500">
                        <select id="handover-category" class="col-span-4 text-xs px-2 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-teal-500">
                            <option value="猫の様子">🐱 猫の様子</option>
                            <option value="施設・物資">🔵 施設・物資</option>
                            <option value="譲渡・来客">🟢 譲渡・来客</option>
                            <option value="その他">⚪ その他</option>
                        </select>
                    </div>
                    <div class="flex gap-2">
                        <select id="handover-cat-link" class="w-1/3 text-xs px-2 py-1.5 rounded-lg border border-slate-200 text-slate-600 focus:outline-none focus:ring-2 focus:ring-teal-500">
                            <option value="">🐾 対象猫(任意)</option>
                        </select>
                        <input type="text" id="handover-text" placeholder="報告内容（例：ハナちゃん少し軟便、完食）" required
                            class="w-2/3 text-xs px-3 py-1.5 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-teal-500">
                    </div>
                    <button type="submit" class="w-full bg-teal-600 hover:bg-teal-700 text-white font-bold py-2 rounded-lg text-xs transition shadow-xs flex items-center justify-center gap-1">
                        <i class="fa-solid fa-paper-plane"></i> 申し送りを登録する
                    </button>
                </form>
            </div>

            <!-- タイムライン一覧 -->
            <div id="handover-list" class="space-y-2">
                <!-- 動的挿入 -->
            </div>
        </section>

        <!-- ================= 2. お世話カルテタブ ================= -->
        <section id="view-cats" class="hidden space-y-3">
            <div class="bg-white p-3 rounded-xl shadow-xs border border-slate-200 space-y-2.5 sticky top-[121px] z-30">
                <div class="relative">
                    <i class="fa-solid fa-magnifying-glass absolute left-3 top-2.5 text-slate-400 text-xs"></i>
                    <input type="text" id="cat-search" oninput="renderCats()" placeholder="名前で検索..."
                        class="w-full text-xs pl-8 pr-3 py-2 rounded-lg border border-slate-200 bg-slate-50 focus:bg-white focus:outline-none focus:ring-2 focus:ring-teal-500">
                </div>
                <div class="flex gap-1.5 overflow-x-auto no-scrollbar text-[11px] font-bold">
                    <button onclick="setFilter('all')" id="filter-all" class="px-3 py-1 rounded-full bg-teal-600 text-white shrink-0 transition">
                        全頭
                    </button>
                    <button onclick="setFilter('care')" id="filter-care" class="px-3 py-1 rounded-full bg-slate-100 text-slate-600 hover:bg-amber-50 shrink-0 transition">
                        🟡 様子見・ケア
                    </button>
                    <button onclick="setFilter('unfed')" id="filter-unfed" class="px-3 py-1 rounded-full bg-slate-100 text-slate-600 hover:bg-teal-50 shrink-0 transition">
                        🍽️ ごはん未確認
                    </button>
                </div>
            </div>

            <div class="text-[11px] text-slate-500 px-1 flex justify-between items-center">
                <span>※食欲・排便はタップで切替可能</span>
                <span id="filtered-count-label">0頭表示中</span>
            </div>

            <div id="cats-list" class="space-y-2.5">
                <!-- 動的挿入 -->
            </div>
        </section>

        <!-- ================= 3. 在籍名簿管理タブ ================= -->
        <section id="view-roster" class="hidden space-y-3">
            <div class="bg-white p-3.5 rounded-xl shadow-xs border border-slate-200 flex justify-between items-center">
                <div>
                    <h2 class="font-bold text-slate-800 text-xs"><i class="fa-solid fa-address-book text-teal-600 mr-1"></i>在籍猫名簿のマスタ管理</h2>
                    <p class="text-[10px] text-slate-500">猫ちゃんの追加・削除・写真や情報の登録を行います</p>
                </div>
                <button onclick="openCatModal()" class="bg-teal-600 hover:bg-teal-700 text-white text-xs px-3 py-2 rounded-lg font-bold shadow-xs transition flex items-center gap-1 shrink-0">
                    <i class="fa-solid fa-plus"></i> 新規猫を追加
                </button>
            </div>

            <div id="roster-list" class="space-y-2">
                <!-- 動的挿入 -->
            </div>
        </section>

    </main>

    <!-- 猫カルテ・名簿詳細編集モーダル -->
    <div id="cat-modal" class="fixed inset-0 bg-black/50 backdrop-blur-xs z-50 hidden flex items-center justify-center p-3">
        <div class="bg-white w-full max-w-md rounded-2xl p-4 shadow-xl max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-3 pb-2 border-b border-slate-100">
                <h3 id="modal-title" class="font-bold text-slate-800 text-sm">猫カルテ・名簿編集</h3>
                <button onclick="closeCatModal()" class="text-slate-400 hover:text-slate-600 w-7 h-7 flex items-center justify-center rounded-full bg-slate-100"><i class="fa-solid fa-xmark"></i></button>
            </div>
            <form id="cat-form" onsubmit="saveCat(event)" class="space-y-3">
                <input type="hidden" id="cat-id">
                
                <!-- 写真アップロード -->
                <div class="flex items-center gap-3 bg-slate-50 p-2.5 rounded-xl border border-slate-200">
                    <div id="photo-preview-box" class="w-14 h-14 rounded-full bg-slate-200 overflow-hidden flex items-center justify-center shrink-0 border border-slate-300">
                        <i class="fa-solid fa-cat text-slate-400 text-xl"></i>
                    </div>
                    <div class="flex-1">
                        <label class="block text-[11px] font-bold text-slate-700 mb-1">猫ちゃんの写真</label>
                        <input type="file" id="input-photo" accept="image/*" onchange="previewPhoto(event)" class="w-xs text-[10px] text-slate-500 file:mr-2 file:py-1 file:px-2.5 file:rounded-lg file:border-0 file:text-[10px] file:font-semibold file:bg-teal-50 file:text-teal-700 hover:file:bg-teal-100">
                    </div>
                </div>

                <div class="grid grid-cols-3 gap-2">
                    <div class="col-span-2">
                        <label class="block text-[11px] font-bold text-slate-600 mb-1">名前 *</label>
                        <input type="text" id="input-name" required placeholder="例: みけ美" class="w-full text-xs px-2.5 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-teal-500">
                    </div>
                    <div>
                        <label class="block text-[11px] font-bold text-slate-600 mb-1">現在の様子</label>
                        <select id="input-status" class="w-full text-xs px-2 py-2 rounded-lg border border-slate-200 font-bold focus:outline-none focus:ring-2 focus:ring-teal-500">
                            <option value="normal">🟢 元気</option>
                            <option value="care">🟡 様子見</option>
                        </select>
                    </div>
                </div>
                <div>
                    <label class="block text-[11px] font-bold text-slate-600 mb-1">性別・特徴</label>
                    <input type="text" id="input-gender" placeholder="例: メス / 三毛猫" class="w-full text-xs px-2.5 py-2 rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-teal-500">
                </div>
                <div class="bg-slate-50 p-2.5 rounded-xl border border-slate-200 space-y-1.5">
                    <label class="block text-[11px] font-bold text-teal-800"><i class="fa-solid fa-paw mr-1"></i>お世話・引き継ぎメモ</label>
                    <textarea id="input-health" rows="2" placeholder="ごはんの好みや性格など" class="w-full text-xs px-2.5 py-1.5 rounded-lg border border-slate-200 bg-white focus:outline-none focus:ring-2 focus:ring-teal-500"></textarea>
                </div>
                <div class="flex gap-2 pt-2">
                    <button type="button" onclick="closeCatModal()" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-600 font-bold py-2.5 rounded-xl text-xs transition">キャンセル</button>
                    <button type="submit" class="flex-1 bg-teal-600 hover:bg-teal-700 text-white font-bold py-2.5 rounded-xl text-xs transition shadow-sm">保存する</button>
                </div>
            </form>
        </div>
    </div>

    <!-- 施設メモ編集モーダル -->
    <div id="facility-modal" class="fixed inset-0 bg-black/50 backdrop-blur-xs z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-sm rounded-2xl p-4 shadow-xl space-y-3">
            <h3 class="font-bold text-slate-800 text-sm flex items-center gap-1.5">
                <i class="fa-solid fa-building-shield text-teal-600"></i> 施設・物資の補足メモ更新
            </h3>
            <textarea id="facility-input-text" rows="3" class="w-full text-xs px-3 py-2 rounded-xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-teal-500" placeholder="例：猫砂ストック残り2袋です。"></textarea>
            <div class="flex gap-2">
                <button onclick="closeFacilityModal()" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-600 font-bold py-2 rounded-xl text-xs">キャンセル</button>
                <button onclick="saveFacilityStatus()" class="flex-1 bg-teal-600 hover:bg-teal-700 text-white font-bold py-2 rounded-xl text-xs">保存する</button>
            </div>
        </div>
    </div>

    <!-- AI文章作成サポートモーダル -->
    <div id="ai-modal" class="fixed inset-0 bg-black/50 backdrop-blur-xs z-50 hidden flex items-center justify-center p-4">
        <div class="bg-white w-full max-w-md rounded-2xl p-4 shadow-xl space-y-3">
            <h3 class="font-bold text-slate-800 text-sm flex items-center gap-1.5">
                <i class="fa-solid fa-wand-magic-sparkles text-indigo-600"></i> Gemini AI サポート（申し送り作成支援）
            </h3>
            <p class="text-[11px] text-slate-500">
                メモ書きや今日のトピックを入力すると、スタッフの皆さんに伝わりやすい丁寧な引き継ぎ文章をAIが自動作成します。
            </p>
            <div>
                <label class="block text-[11px] font-bold text-slate-600 mb-1">伝えたい要点（箇条書きOK）</label>
                <textarea id="ai-prompt-input" rows="3" class="w-full text-xs px-3 py-2 rounded-xl border border-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500" placeholder="例：みけ美ちゃんがご飯完食した、お薬も飲めた。トイレの砂補充した。"></textarea>
            </div>
            <div id="ai-loading" class="hidden text-center py-2 text-indigo-600 text-xs font-bold animate-pulse">
                <i class="fa-solid fa-spinner fa-spin mr-1"></i> AIが文章を考えています...
            </div>
            <div class="flex gap-2 pt-1">
                <button onclick="closeAiModal()" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-600 font-bold py-2 rounded-xl text-xs">キャンセル</button>
                <button onclick="generateAiHandover()" class="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 rounded-xl text-xs shadow-sm flex items-center justify-center gap-1">
                    <i class="fa-solid fa-sparkles"></i> 文章を生成する
                </button>
            </div>
        </div>
    </div>

    <script>
        const today = new Date();
        const days = ['日', '月', '火', '水', '木', '金', '土'];
        document.getElementById('current-date-display').innerText = 
            `${today.getFullYear()}年${today.getMonth()+1}月${today.getDate()}日(${days[today.getDay()]}) シフト共有`;

        // 初期30匹データ
        const defaultCats = [
            { id: '1', name: 'みけ美', gender: 'メス / 三毛', status: 'care', appetite: '◎', stool: '良', health: '右目少し涙目なのでコットンで拭いてあげてください。', photo: '' },
            { id: '2', name: 'チャチャ', gender: 'オス / 茶トラ', status: 'care', appetite: '△', stool: '良', health: '少し警戒心あり。ウェットトッピングで完食。', photo: '' },
            { id: '3', name: 'レオ', gender: 'オス / 黒猫', status: 'normal', appetite: '◎', stool: '良', health: '元気いっぱい。', photo: '' },
            { id: '4', name: 'ハナ', gender: 'メス / 白黒', status: 'care', appetite: '△', stool: '軟', health: '少し軟便ぎみなので便の状態を確認お願いします。', photo: '' },
            { id: '5', name: 'コタロー', gender: 'オス / キジ', status: 'normal', appetite: '◎', stool: '良', health: 'ウェットフード大好き。', photo: '' },
            { id: '6', name: 'サクラ', gender: 'メス / サビ', status: 'normal', appetite: '◎', stool: '良', health: '落ち着いています。', photo: '' },
            { id: '7', name: 'モモ', gender: 'メス / 白茶', status: 'normal', appetite: '◎', stool: '良', health: '問題なし。', photo: '' },
            { id: '8', name: 'ソラ', gender: 'オス / グレー', status: 'normal', appetite: '◎', stool: '良', health: '食欲旺盛。', photo: '' },
            { id: '9', name: 'ナッツ', gender: 'オス / 茶白', status: 'normal', appetite: '◎', stool: '良', health: '仲良く遊んでいます。', photo: '' },
            { id: '10', name: 'ラテ', gender: 'メス / クリーム', status: 'care', appetite: '◎', stool: '良', health: '体調管理中。', photo: '' },
            { id: '11', name: 'クッキー', gender: 'オス / サバ', status: 'normal', appetite: '◎', stool: '良', health: 'ドライフード良好。', photo: '' },
            { id: '12', name: 'シロ', gender: 'オス / 白', status: 'care', appetite: '◎', stool: '軟', health: '便チェック継続。', photo: '' },
            { id: '13', name: 'ビー', gender: 'メス / キジ白', status: 'care', appetite: '△', stool: '未', health: '保護直後。', photo: '' },
            { id: '14', name: 'フク', gender: 'オス / 黒白', status: 'normal', appetite: '◎', stool: '良', health: '元気。', photo: '' },
            { id: '15', name: 'メイ', gender: 'メス / 三毛', status: 'normal', appetite: '◎', stool: '良', health: 'おだやか。', photo: '' },
            { id: '16', name: 'キナコ', gender: 'メス / 茶トラ', status: 'normal', appetite: '◎', stool: '良', health: 'おだやか。', photo: '' },
            { id: '17', name: 'アズキ', gender: 'メス / 茶トラ', status: 'normal', appetite: 'ー', stool: '未', health: '少し臆病。', photo: '' },
            { id: '18', name: 'ジン', gender: 'オス / 黒', status: 'normal', appetite: '◎', stool: '良', health: '元気。', photo: '' },
            { id: '19', name: 'リン', gender: 'メス / グレー', status: 'normal', appetite: '◎', stool: '良', health: '問題なし。', photo: '' },
            { id: '20', name: 'ロン', gender: 'オス / キジ', status: 'normal', appetite: 'ー', stool: '良', health: 'シニア。', photo: '' },
            { id: '21', name: 'ティアラ', gender: 'メス / 白', status: 'normal', appetite: '◎', stool: '良', health: 'ブラッシング好き。', photo: '' },
            { id: '22', name: 'マロン', gender: 'オス / 茶白', status: 'normal', appetite: '◎', stool: '良', health: '食欲良好。', photo: '' },
            { id: '23', name: 'ショコラ', gender: 'メス / サビ', status: 'normal', appetite: 'ー', stool: '未', health: '元気。', photo: '' },
            { id: '24', name: 'プリン', gender: 'メス / クリーム', status: 'care', appetite: '◎', stool: '良', health: '優しく接してください。', photo: '' },
            { id: '25', name: 'エース', gender: 'オス / 黒', status: 'normal', appetite: '◎', stool: '良', health: '元気。', photo: '' },
            { id: '26', name: 'ルル', gender: 'メス / 三毛', status: 'normal', appetite: '◎', stool: '良', health: '良好。', photo: '' },
            { id: '27', name: 'キラ', gender: 'オス / 白', status: 'care', appetite: '△', stool: '良', health: '環境変化で緊張気味。', photo: '' },
            { id: '28', name: 'ユキ', gender: 'メス / 白', status: 'normal', appetite: '◎', stool: '良', health: '良好。', photo: '' },
            { id: '29', name: 'ゴロウ', gender: 'オス / キジ', status: 'normal', appetite: '◎', stool: '良', health: 'おやつ完食。', photo: '' },
            { id: '30', name: 'ハチ', gender: 'オス / 白黒', status: 'normal', appetite: '◎', stool: '良', health: '元気。', photo: '' }
        ];

        let cats = JSON.parse(localStorage.getItem('rescue_photo_cats_30')) || defaultCats;
        let handovers = JSON.parse(localStorage.getItem('rescue_photo_handovers')) || [
            { id: '1', shift: '朝シフト', poster: '鈴木', category: '猫の様子', catName: 'ハナ', text: '朝のごはん完食しました！少し軟便だったので夕方トイレ確認お願いします。', time: '今日 09:30', checks: 2 }
        ];
        let facilityChecks = JSON.parse(localStorage.getItem('rescue_photo_fac_checks')) || [
            { id: 'clean', label: '掃除・消毒', done: true },
            { id: 'water', label: '全室給水', done: true },
            { id: 'trash', label: 'ゴミ回収', done: false },
            { id: 'lock', label: '空調・戸締', done: false }
        ];
        let facilityStatus = localStorage.getItem('rescue_photo_fac_memo') || '【物資連絡】猫砂ストック残り3袋 / エアコン25度稼働中';
        let currentFilter = 'all';
        let tempPhotoBase64 = '';

        function showToast(msg) {
            const t = document.getElementById('toast');
            t.innerText = msg;
            t.classList.remove('hidden');
            setTimeout(() => t.classList.add('hidden'), 2000);
        }

        function saveData() {
            localStorage.setItem('rescue_photo_cats_30', JSON.stringify(cats));
            localStorage.setItem('rescue_photo_handovers', JSON.stringify(handovers));
            localStorage.setItem('rescue_photo_fac_checks', JSON.stringify(facilityChecks));
            localStorage.setItem('rescue_photo_fac_memo', facilityStatus);
            updateDashboardStats();
            renderPriorityPreview();
            populateCatSelect();
        }

        function updateDashboardStats() {
            document.getElementById('stat-total').innerText = cats.length;
            document.getElementById('stat-care').innerText = cats.filter(c => c.status === 'care').length;
            document.getElementById('stat-fed').innerText = cats.filter(c => c.appetite && c.appetite !== 'ー').length;
        }

        function renderFacilityToggles() {
            const container = document.getElementById('facility-toggles');
            let html = '';
            facilityChecks.forEach(item => {
                const activeClass = item.done ? 'bg-teal-50 border-teal-500 text-teal-800' : 'bg-slate-50 border-slate-200 text-slate-400';
                const icon = item.done ? 'fa-circle-check text-teal-600' : 'fa-circle text-slate-300';
                html += `
                    <button onclick="toggleFacilityCheck('${item.id}')" class="border rounded-lg p-1.5 flex flex-col items-center justify-center gap-0.5 transition ${activeClass}">
                        <i class="fa-solid ${icon} text-xs"></i>
                        <span>${item.label}</span>
                    </button>
                `;
            });
            container.innerHTML = html;
            document.getElementById('facility-summary-text').innerText = facilityStatus;
        }

        function toggleFacilityCheck(id) {
            facilityChecks = facilityChecks.map(f => f.id === id ? { ...f, done: !f.done } : f);
            saveData();
            renderFacilityToggles();
        }

        function renderPriorityPreview() {
            const container = document.getElementById('priority-cats-preview');
            const priorityCats = cats.filter(c => c.status === 'care');
            if (priorityCats.length === 0) {
                container.innerHTML = `<div class="text-xs text-slate-500 py-1">現在、様子見・ケア対象の猫はいません</div>`;
                return;
            }
            let html = '';
            priorityCats.forEach(c => {
                const imgTag = c.photo 
                    ? `<img src="${c.photo}" class="w-9 h-9 rounded-full object-cover border border-amber-300 shrink-0">` 
                    : `<div class="w-9 h-9 rounded-full bg-amber-100 flex items-center justify-center text-amber-700 text-xs shrink-0"><i class="fa-solid cat">🐱</i></div>`;
                html += `
                    <div onclick="openCatModal('${c.id}')" class="bg-white p-2.5 rounded-lg border border-amber-200 shadow-2xs shrink-0 w-44 cursor-pointer hover:border-amber-400 transition">
                        <div class="flex items-center gap-2 mb-1">
                            ${imgTag}
                            <div class="overflow-hidden">
                                <span class="font-bold text-xs text-slate-800 block truncate">${escapeHtml(c.name)}</span>
                                <span class="bg-amber-500 text-white text-[9px] px-1 py-0.5 rounded font-bold">様子見</span>
                            </div>
                        </div>
                        <p class="text-[10px] text-slate-500 mb-1">🍽️${c.appetite || 'ー'} 🚽${c.stool || '未'}</p>
                        <p class="text-[11px] text-slate-700 line-clamp-1 bg-amber-50/50 p-1 rounded">${escapeHtml(c.health)}</p>
                    </div>
                `;
            });
            container.innerHTML = html;
        }

        function populateCatSelect() {
            const sel = document.getElementById('handover-cat-link');
            let html = '<option value="">🐾 対象猫(任意)</option>';
            cats.forEach(c => {
                html += `<option value="${escapeHtml(c.name)}">${escapeHtml(c.name)}</option>`;
            });
            sel.innerHTML = html;
        }

        function switchTab(tabName) {
            ['handover', 'cats', 'roster'].forEach(t => {
                document.getElementById('view-' + t).classList.add('hidden');
                document.getElementById('tab-' + t).className = 'flex-1 py-3 text-center border-b-2 border-transparent text-slate-500 hover:text-teal-700 transition flex items-center justify-center gap-1';
            });
            document.getElementById('view-' + tabName).classList.remove('hidden');
            document.getElementById('tab-' + tabName).className = 'flex-1 py-3 text-center border-b-2 border-teal-600 text-teal-700 transition font-bold flex items-center justify-center gap-1';

            if (tabName === 'cats') renderCats();
            if (tabName === 'roster') renderRoster();
        }

        function openFacilityModal() {
            document.getElementById('facility-input-text').value = facilityStatus;
            document.getElementById('facility-modal').classList.remove('hidden');
        }
        function closeFacilityModal() {
            document.getElementById('facility-modal').classList.add('hidden');
        }

        // --- Gemini API連携機能 ---
        function openAiModal() {
            document.getElementById('ai-prompt-input').value = '';
            document.getElementById('ai-loading').classList.add('hidden');
            document.getElementById('ai-modal').classList.remove('hidden');
        }

        function closeAiModal() {
            document.getElementById('ai-modal').classList.add('hidden');
        }

        async function generateAiHandover() {
            const promptText = document.getElementById('ai-prompt-input').value.trim();
            if (!promptText) {
                showToast('伝えたい要点を入力してください');
                return;
            }

            const loadingEl = document.getElementById('ai-loading');
            loadingEl.classList.remove('hidden');

            const systemPrompt = "あなたは保護猫シェルターのベテランボランティアスタッフです。入力された箇条書きやメモから、次のシフトのスタッフにわかりやすく丁寧で温かみのある引き継ぎ・申し送り文章を1〜2文で簡潔に作成してください。余計な前置きは不要です。";
            const userQuery = `以下のメモを元に、引き継ぎ用の文章を作成してください:\n${promptText}`;
            
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: userQuery }] }],
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                }
            };

            // 指数バックオフ付きフェッチ関数
            async function fetchWithRetry(url, options, retries = 3, delay = 1000) {
                try {
                    const response = await fetch(url, options);
                    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                    return await response.json();
                } catch (error) {
                    if (retries > 0) {
                        await new Promise(res => setTimeout(res, delay));
                        return fetchWithRetry(url, options, retries - 1, delay * 2);
                    }
                    throw error;
                }
            }

            try {
                const result = await fetchWithRetry(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                const candidate = result.candidates?.[0];
                if (candidate && candidate.content?.parts?.[0]?.text) {
                    const generatedText = candidate.content.parts[0].text.trim();
                    document.getElementById('handover-text').value = generatedText;
                    closeAiModal();
                    showToast('AIが申し送り文面を作成しました！');
                } else {
                    throw new Error('予期せぬレスポンス構造です');
                }
            } catch (err) {
                console.error(err);
                showToast('AI文章の生成に失敗しました。時間を置いて再度お試しください。');
            } finally {
                loadingEl.classList.add('hidden');
            }
        }

        function saveFacilityStatus() {
            facilityStatus = document.getElementById('facility-input-text').value;
            saveData();
            renderFacilityToggles();
            closeFacilityModal();
            showToast('施設メモを更新しました');
        }

        function addHandover(e) {
            e.preventDefault();
            const shift = document.getElementById('handover-shift').value;
            const poster = document.getElementById('poster-name').value;
            const category = document.getElementById('handover-category').value;
            const catName = document.getElementById('handover-cat-link').value;
            const text = document.getElementById('handover-text').value;
            const now = new Date();
            const timeStr = `${now.getMonth() + 1}/${now.getDate()} ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}`;

            const newPost = { id: Date.now().toString(), shift, poster, category, catName, text, time: timeStr, checks: 0 };
            handovers.unshift(newPost);
            saveData();

            document.getElementById('handover-text').value = '';
            document.getElementById('handover-cat-link').value = '';
            renderHandovers();
            showToast('申し送りを登録しました');
        }

        function checkHandover(id) {
            handovers = handovers.map(h => h.id === id ? { ...h, checks: (h.checks || 0) + 1 } : h);
            saveData();
            renderHandovers();
        }

        function deleteHandover(id) {
            handovers = handovers.filter(h => h.id !== id);
            saveData();
            renderHandovers();
            showToast('投稿を削除しました');
        }

        function renderHandovers() {
            const list = document.getElementById('handover-list');
            if (handovers.length === 0) {
                list.innerHTML = `<div class="text-center py-6 text-slate-400 text-xs bg-white rounded-xl border border-slate-200">まだ申し送り投稿はありません</div>`;
                return;
            }
            let html = '';
            handovers.forEach(h => {
                let badgeColor = 'bg-slate-100 text-slate-700';
                if (h.category.includes('猫') || h.category.includes('体調')) badgeColor = 'bg-amber-100 text-amber-800';
                if (h.category.includes('譲渡')) badgeColor = 'bg-emerald-100 text-emerald-800';
                if (h.category.includes('施設')) badgeColor = 'bg-sky-100 text-sky-800';

                html += `
                    <div class="bg-white p-3 rounded-xl shadow-2xs border border-slate-200 space-y-1.5">
                        <div class="flex justify-between items-center text-xs">
                            <div class="flex items-center gap-1.5">
                                <span class="bg-teal-50 text-teal-800 border border-teal-200 px-1.5 py-0.5 rounded text-[10px] font-bold">${escapeHtml(h.shift)}</span>
                                <span class="font-bold text-slate-800">${escapeHtml(h.poster)}</span>
                                <span class="px-1.5 py-0.5 rounded text-[10px] font-bold ${badgeColor}">${escapeHtml(h.category)}</span>
                            </div>
                            <span class="text-slate-400 text-[10px]">${h.time}</span>
                        </div>
                        <div class="text-xs text-slate-700 leading-relaxed">
                            ${h.catName ? `<span class="inline-block bg-amber-100 text-amber-900 font-bold text-[10px] px-1.5 py-0.5 rounded mr-1"><i class="fa-solid fa-paw mr-0.5"></i>${escapeHtml(h.catName)}</span>` : ''}
                            <span>${escapeHtml(h.text)}</span>
                        </div>
                        <div class="flex justify-between items-center pt-1 border-t border-slate-100">
                            <button onclick="checkHandover('${h.id}')" class="text-[11px] bg-slate-100 hover:bg-teal-50 text-slate-600 hover:text-teal-700 font-semibold px-2.5 py-1 rounded-full transition flex items-center gap-1">
                                <i class="fa-solid fa-thumbs-up text-teal-600"></i> 確認済 (${h.checks || 0})
                            </button>
                            <button onclick="deleteHandover('${h.id}')" class="text-[10px] text-slate-400 hover:text-rose-600 px-1"><i class="fa-solid fa-trash-can"></i></button>
                        </div>
                    </div>
                `;
            });
            list.innerHTML = html;
        }

        const appetiteOrder = ['◎', '△', '×', 'ー'];
        function cycleAppetite(catId) {
            cats = cats.map(c => {
                if (c.id === catId) {
                    const idx = appetiteOrder.indexOf(c.appetite || 'ー');
                    return { ...c, appetite: appetiteOrder[(idx + 1) % appetiteOrder.length] };
                }
                return c;
            });
            saveData();
            renderCats();
        }

        const stoolOrder = ['良', '軟', '下', '未'];
        function cycleStool(catId) {
            cats = cats.map(c => {
                if (c.id === catId) {
                    const idx = stoolOrder.indexOf(c.stool || '未');
                    return { ...c, stool: stoolOrder[(idx + 1) % stoolOrder.length] };
                }
                return c;
            });
            saveData();
            renderCats();
        }

        function resetDailyChecks() {
            cats = cats.map(c => ({ ...c, appetite: 'ー', stool: '未' }));
            facilityChecks = facilityChecks.map(f => ({ ...f, done: false }));
            saveData();
            renderFacilityToggles();
            renderCats();
            showToast('本日の食欲・排便チェックをリセットしました');
        }

        function setFilter(filterType) {
            currentFilter = filterType;
            ['all', 'care', 'unfed'].forEach(f => {
                const btn = document.getElementById('filter-' + f);
                if (btn) {
                    btn.className = (f === filterType)
                        ? 'px-3 py-1 rounded-full bg-teal-600 text-white shrink-0 transition font-bold'
                        : 'px-3 py-1 rounded-full bg-slate-100 text-slate-600 hover:bg-slate-200 shrink-0 transition font-bold';
                }
            });
            renderCats();
        }

        function renderCats() {
            const query = (document.getElementById('cat-search').value || '').toLowerCase();
            const list = document.getElementById('cats-list');

            const filtered = cats.filter(c => {
                const matchesQuery = c.name.toLowerCase().includes(query) || (c.gender && c.gender.toLowerCase().includes(query));
                if (!matchesQuery) return false;
                if (currentFilter === 'care') return c.status === 'care';
                if (currentFilter === 'unfed') return c.appetite === 'ー';
                return true;
            });

            document.getElementById('filtered-count-label').innerText = `${filtered.length}頭表示中`;

            if (filtered.length === 0) {
                list.innerHTML = `<div class="text-center py-8 text-slate-400 text-xs bg-white rounded-xl border border-slate-200">猫ちゃんが見つかりません</div>`;
                return;
            }

            let html = '';
            filtered.forEach(c => {
                let cardBorder = 'border-slate-200';
                let statusBadge = `<span class="bg-emerald-100 text-emerald-800 text-[10px] font-bold px-2 py-0.5 rounded">元気</span>`;
                if (c.status === 'care') {
                    cardBorder = 'border-amber-300 bg-amber-50/20';
                    statusBadge = `<span class="bg-amber-500 text-white text-[10px] font-bold px-2 py-0.5 rounded">様子見</span>`;
                }

                let appColor = 'bg-slate-100 text-slate-600 border-slate-200';
                if (c.appetite === '◎') appColor = 'bg-emerald-50 text-emerald-700 border-emerald-300';
                if (c.appetite === '△') appColor = 'bg-amber-50 text-amber-700 border-amber-300';
                if (c.appetite === '×') appColor = 'bg-rose-50 text-rose-700 border-rose-300';

                let stoolColor = 'bg-slate-100 text-slate-600 border-slate-200';
                if (c.stool === '良') stoolColor = 'bg-emerald-50 text-emerald-700 border-emerald-300';
                if (c.stool === '軟') stoolColor = 'bg-amber-50 text-amber-700 border-amber-300';
                if (c.stool === '下') stoolColor = 'bg-rose-50 text-rose-700 border-rose-300';

                const avatarHtml = c.photo 
                    ? `<img src="${c.photo}" class="w-11 h-11 rounded-full object-cover border border-slate-300 shrink-0">`
                    : `<div class="w-11 h-11 rounded-full bg-slate-200 flex items-center justify-center text-slate-400 text-base shrink-0"><i class="fa-solid fa-cat"></i></div>`;

                html += `
                    <div class="bg-white p-3 rounded-xl shadow-2xs border ${cardBorder} space-y-2">
                        <div class="flex justify-between items-center gap-2">
                            <div class="flex items-center gap-2.5">
                                ${avatarHtml}
                                <div>
                                    <div class="flex items-center gap-1.5">
                                        <span class="font-bold text-slate-800 text-sm">${escapeHtml(c.name)}</span>
                                        ${statusBadge}
                                    </div>
                                    <p class="text-[11px] text-slate-400">${escapeHtml(c.gender || '特徴未設定')}</p>
                                </div>
                            </div>
                            <button onclick="openCatModal('${c.id}')" class="text-slate-400 hover:text-teal-600 px-2 py-1 text-xs border border-slate-200 rounded-lg bg-slate-50">
                                <i class="fa-solid fa-pen"></i> 編集
                            </button>
                        </div>

                        <div class="flex items-center gap-2 pt-0.5">
                            <button onclick="cycleAppetite('${c.id}')" class="border rounded-lg px-3 py-1 text-[11px] font-bold transition ${appColor}">
                                🍽️ 食欲: ${c.appetite || 'ー'}
                            </button>
                            <button onclick="cycleStool('${c.id}')" class="border rounded-lg px-3 py-1 text-[11px] font-bold transition ${stoolColor}">
                                🚽 便: ${c.stool || '未'}
                            </button>
                        </div>

                        <div class="text-xs text-slate-700 bg-slate-50 px-2.5 py-1.5 rounded-lg border border-slate-100">
                            <strong class="text-slate-500">メモ:</strong> ${escapeHtml(c.health || '特記事項なし')}
                        </div>
                    </div>
                `;
            });
            list.innerHTML = html;
        }

        function renderRoster() {
            const list = document.getElementById('roster-list');
            let html = '';
            cats.forEach((c, idx) => {
                const avatarHtml = c.photo 
                    ? `<img src="${c.photo}" class="w-10 h-10 rounded-full object-cover border border-slate-300 shrink-0">`
                    : `<div class="w-10 h-10 rounded-full bg-slate-200 flex items-center justify-center text-slate-400 text-sm shrink-0"><i class="fa-solid fa-cat"></i></div>`;

                html += `
                    <div class="bg-white p-2.5 rounded-xl border border-slate-200 flex justify-between items-center">
                        <div class="flex items-center gap-2.5">
                            ${avatarHtml}
                            <div>
                                <div class="flex items-center gap-1.5">
                                    <span class="text-xs text-slate-400 font-bold">#${idx+1}</span>
                                    <span class="font-bold text-slate-800 text-xs">${escapeHtml(c.name)}</span>
                                    <span class="text-[10px] text-slate-500">(${escapeHtml(c.gender || '-')})</span>
                                </div>
                                <p class="text-[10px] text-slate-500 truncate max-w-[200px]">${escapeHtml(c.health || 'メモなし')}</p>
                            </div>
                        </div>
                        <div class="flex gap-1">
                            <button onclick="openCatModal('${c.id}')" class="bg-slate-100 hover:bg-slate-200 text-slate-600 text-[11px] px-2.5 py-1.5 rounded-lg font-bold">編集</button>
                            <button onclick="deleteCat('${c.id}')" class="bg-rose-50 hover:bg-rose-100 text-rose-600 text-[11px] px-2 py-1.5 rounded-lg"><i class="fa-solid fa-trash"></i></button>
                        </div>
                    </div>
                `;
            });
            list.innerHTML = html;
        }

        function previewPhoto(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    tempPhotoBase64 = e.target.result;
                    document.getElementById('photo-preview-box').innerHTML = `<img src="${tempPhotoBase64}" class="w-full h-full object-cover">`;
                };
                reader.readAsDataURL(file);
            }
        }

        function openCatModal(catId = null) {
            document.getElementById('cat-modal').classList.remove('hidden');
            tempPhotoBase64 = '';
            if (catId) {
                const cat = cats.find(c => c.id === catId);
                if (cat) {
                    document.getElementById('modal-title').innerText = '猫情報・カルテ編集';
                    document.getElementById('cat-id').value = cat.id;
                    document.getElementById('input-name').value = cat.name;
                    document.getElementById('input-status').value = cat.status === 'care' ? 'care' : 'normal';
                    document.getElementById('input-gender').value = cat.gender || '';
                    document.getElementById('input-health').value = cat.health || '';
                    tempPhotoBase64 = cat.photo || '';
                    if (cat.photo) {
                        document.getElementById('photo-preview-box').innerHTML = `<img src="${cat.photo}" class="w-full h-full object-cover">`;
                    } else {
                        document.getElementById('photo-preview-box').innerHTML = `<i class="fa-solid fa-cat text-slate-400 text-xl"></i>`;
                    }
                }
            } else {
                document.getElementById('modal-title').innerText = '新規猫ちゃんの登録';
                document.getElementById('cat-form').reset();
                document.getElementById('cat-id').value = '';
                document.getElementById('photo-preview-box').innerHTML = `<i class="fa-solid fa-cat text-slate-400 text-xl"></i>`;
            }
        }

        function closeCatModal() {
            document.getElementById('cat-modal').classList.add('hidden');
        }

        function saveCat(e) {
            e.preventDefault();
            const id = document.getElementById('cat-id').value;
            const name = document.getElementById('input-name').value;
            const status = document.getElementById('input-status').value;
            const gender = document.getElementById('input-gender').value;
            const health = document.getElementById('input-health').value;

            if (id) {
                cats = cats.map(c => c.id === id ? { ...c, name, status, gender, health, photo: tempPhotoBase64 || c.photo } : c);
            } else {
                const newCat = { 
                    id: Date.now().toString(), name, status, gender, 
                    appetite: 'ー', stool: '未', health, photo: tempPhotoBase64 
                };
                cats.push(newCat);
            }
            saveData();
            closeCatModal();
            renderCats();
            renderRoster();
            showToast('猫情報を保存しました');
        }

        function deleteCat(id) {
            if (confirm('この猫ちゃんの名簿データを削除しますか？')) {
                cats = cats.filter(c => c.id !== id);
                saveData();
                renderCats();
                renderRoster();
                showToast('削除しました');
            }
        }

        function copyLineReport() {
            const d = new Date();
            const dateStr = `${d.getFullYear()}年${d.getMonth()+1}月${d.getDate()}日`;
            let text = `【WithCat引継ぎレポート (${dateStr})】\n\n`;

            text += `■ 施設・物資状況\n`;
            facilityChecks.forEach(f => {
                text += `${f.done ? '☑' : '□'} ${f.label}\n`;
            });
            text += `メモ: ${facilityStatus}\n\n`;

            const careCats = cats.filter(c => c.status === 'care');
            text += `■ 本日の様子見・ケア対象 (${careCats.length}頭)\n`;
            if (careCats.length === 0) {
                text += `特になし\n`;
            } else {
                careCats.forEach(c => {
                    text += `・${c.name} (食欲:${c.appetite || 'ー'} / 便:${c.stool || '未'})\n`;
                    text += `  メモ:${c.health || 'なし'}\n`;
                });
            }
            text += `\n`;

            text += `■ 最近の申し送り事項\n`;
            if (handovers.length === 0) {
                text += `特になし\n`;
            } else {
                handovers.slice(0, 3).forEach(h => {
                    text += `[${h.shift}] ${h.poster} (${h.time})\n`;
                    if(h.catName) text += `対象: ${h.catName}\n`;
                    text += `${h.text}\n---\n`;
                });
            }
            text += `\n（WithCat引継ぎアプリ）`;

            if (navigator.clipboard && navigator.clipboard.writeText) {
                navigator.clipboard.writeText(text).then(() => {
                    showToast('LINE用レポートをコピーしました！');
                }).catch(() => {
                    fallbackCopyText(text);
                });
            } else {
                fallbackCopyText(text);
            }
        }

        function fallbackCopyText(text) {
            const textarea = document.createElement("textarea");
            textarea.value = text;
            textarea.style.position = "fixed";
            document.body.appendChild(textarea);
            textarea.focus();
            textarea.select();
            try {
                document.execCommand('copy');
                showToast('LINE用レポートをコピーしました！');
            } catch (err) {
                showToast('コピーに失敗しました');
            }
            document.body.removeChild(textarea);
        }

        function escapeHtml(str) {
            if (!str) return '';
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }

        updateDashboardStats();
        renderFacilityToggles();
        renderPriorityPreview();
        populateCatSelect();
        renderHandovers();
        renderRoster();
    </script>
</body>
</html>
