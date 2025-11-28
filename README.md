<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<title>Generator Angka Filter Histori</title>
<style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    .box { border: 1px solid #999; padding: 10px; margin: 10px 0; width: 300px; }
    .row { display: flex; align-items: center; gap: 10px; }
    textarea { width: 360px; height: 120px; }
    #sortBox { width: 360px; height: 120px; }
</style>
</head>
<body>

<h2>Generator Angka Dengan Filter Histori</h2>

<textarea id="historyBox" placeholder="Masukkan histori 4 digit (pisahkan dengan spasi / enter / tab)"></textarea><br><br>

<button onclick="generateNumbers()">Generate</button>
<button onclick="resetAll()">Reset</button>

<div id="result"></div>

<hr>

<h2>Penyortir</h2>
<textarea id="sortBox" placeholder="Masukkan angka format 00*01*..."></textarea><br>
<button onclick="processSort()">Proses</button>
<div class="row">
    <div class="box" id="sortResult"></div>
    <button onclick="copySort()">Salin</button>
</div>

<script>
let generatedCache = null; // hasil generate hanya sekali

// ==================== PARSE HISTORI =====================
function parseHistory() {
    const raw = document.getElementById("historyBox").value;
    if (!raw.trim()) return [];

    return raw
        .split(/\s+/)
        .map(x => x.trim())
        .filter(x => /^\d{4}$/.test(x));
}

// ================== ATURAN FILTER ========================
function forbidden2DfromHistory(history) {
    const last10 = history.slice(-10);
    const forbidden = new Set();

    last10.forEach(h => {
        forbidden.add(h.substring(0, 2)); // 2D depan
        forbidden.add(h.substring(2, 4)); // 2D belakang
    });

    return forbidden;
}

function forbiddenSamePositionDigits(history) {
    if (history.length < 2) return { tens: null, ones: null };

    const h1 = history[history.length - 1];
    const h2 = history[history.length - 2];

    return {
        tens: (h1[1] === h2[1]) ? h1[1] : null,
        ones: (h1[3] === h2[3]) ? h1[3] : null
    };
}

function lastHistoryHasDouble(history) {
    if (history.length === 0) return false;
    const h = history[history.length - 1];
    return h[0] === h[1] || h[2] === h[3];
}

// ================== GENERATE =============================
function generateNumbers() {
    if (generatedCache) {
        // tampilkan hasil lama
        renderResult(generatedCache);
        return;
    }

    const history = parseHistory();
    const forbid2D = forbidden2DfromHistory(history);
    const posRule = forbiddenSamePositionDigits(history);
    const forbidDouble = lastHistoryHasDouble(history);

    const resultSet = new Set();

    while (resultSet.size < 50) {
        const num = String(Math.floor(Math.random() * 100)).padStart(2, '0');

        // rule 1 → tidak boleh dari 10 histori terakhir
        if (forbid2D.has(num)) continue;

        // rule 2 → posisi sama pada 2 histori terakhir
        if (posRule.tens !== null && num[0] === posRule.tens) continue;
        if (posRule.ones !== null && num[1] === posRule.ones) continue;

        // rule 3 → histori terakhir angka kembar → hasil tidak boleh kembar
        if (forbidDouble && num[0] === num[1]) continue;

        resultSet.add(num);
    }

    generatedCache = Array.from(resultSet);
    renderResult(generatedCache);
}

function renderResult(numbers) {
    const container = document.getElementById('result');
    container.innerHTML = '';

    for (let i = 0; i < 5; i++) {
        const group = numbers.slice(i * 10, i * 10 + 10);
        const text = group.join('*');

        const div = document.createElement('div');
        div.className = 'row';
        div.innerHTML = `
            <div class='box'>${text}</div>
            <button onclick="copyText('${text}')">Salin</button>
        `;
        container.appendChild(div);
    }
}

function copyText(txt) {
    navigator.clipboard.writeText(txt);
}

// ================== RESET SYSTEM =========================
function resetAll() {
    document.getElementById("historyBox").value = "";
    document.getElementById("result").innerHTML = "";
    generatedCache = null;
    alert("Semua data telah di-reset.");
}

// ================== SORTER ===============================
function processSort() {
    const txt = document.getElementById('sortBox').value.trim();
    if (!txt) return alert('Textbox masih kosong');

    const raw = txt.split('*').map(x => x.trim()).filter(x => x !== '');
    const unique = Array.from(new Set(raw));

    if (unique.length < 10) {
        alert('Isi kurang dari 10 angka unik');
        return;
    }

    const result = [];
    while (result.length < 10) {
        const pick = unique[Math.floor(Math.random() * unique.length)];
        if (!result.includes(pick)) result.push(pick);
    }

    document.getElementById('sortResult').innerHTML = result.join('*');
}

function copySort() {
    const txt = document.getElementById('sortResult').innerText;
    navigator.clipboard.writeText(txt);
}
</script>

</body>
</html>
