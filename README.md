<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bolão Mega da Virada 2025</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <style>
        .sena-bg { background-color: #dcfce7; border: 1px solid #16a34a; }
        .quina-bg { background-color: #fef9c3; border: 1px solid #ca8a04; }
        .quadra-bg { background-color: #dbeafe; border: 1px solid #2563eb; }
        .duplicate-bg { background-color: #fee2e2; border: 1px solid #ef4444; }
        .hit-number { background-color: #16a34a; color: white; font-weight: bold; border-color: #16a34a; }
        .draw-ball { 
            background: radial-gradient(circle at 30% 30%, #22c55e, #15803d);
            box-shadow: inset -1px -1px 2px rgba(0,0,0,0.3);
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen pb-10">

    <nav class="bg-green-700 text-white p-2 shadow-md sticky top-0 z-50">
        <div class="container mx-auto flex items-center gap-3">
            <h1 class="text-lg font-bold flex items-center gap-1 shrink-0"><i class="fas fa-clover text-sm"></i> Bolão</h1>
            <input type="text" id="searchInput" placeholder="Pesquisar..." class="flex-1 p-1.5 text-sm rounded text-gray-800 focus:outline-none h-8">
            <button onclick="toggleAdminPanel()" class="bg-green-800 px-2 py-1 rounded text-[10px] border border-green-600 uppercase">Admin</button>
        </div>
    </nav>

    <div id="passwordModal" class="hidden fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
        <div class="bg-white rounded-lg p-6 max-w-sm w-full mx-4 shadow-xl">
            <h3 class="text-lg font-bold mb-4 text-gray-800">Acesso Administrativo</h3>
            <input type="password" id="adminPassword" placeholder="Digite a senha" class="w-full p-3 border border-gray-300 rounded mb-4 focus:outline-none focus:border-green-500">
            <div class="flex gap-2">
                <button onclick="closePasswordModal()" class="flex-1 bg-gray-300 text-gray-700 px-4 py-2 rounded hover:bg-gray-400 transition">Cancelar</button>
                <button onclick="checkPassword()" class="flex-1 bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700 transition">Entrar</button>
            </div>
            <p id="passwordError" class="text-red-500 text-sm mt-2 hidden">Senha incorreta!</p>
        </div>
    </div>

    <div class="container mx-auto mt-2 px-2">
        <div class="bg-white p-2 rounded-lg shadow-sm grid grid-cols-4 gap-1 text-center font-bold text-xs">
            <div class="bg-gray-100 p-1 rounded border border-gray-300 text-gray-600"><div class="text-[8px] uppercase">Part.</div><span id="countTotal">0</span></div>
            <div class="bg-green-50 p-1 rounded border border-green-500 text-green-700"><div class="text-[8px] uppercase">Sena</div><span id="countSena">0</span></div>
            <div class="bg-yellow-50 p-1 rounded border border-yellow-500 text-yellow-700"><div class="text-[8px] uppercase">Quina</div><span id="countQuina">0</span></div>
            <div class="bg-blue-50 p-1 rounded border border-blue-500 text-blue-700"><div class="text-[8px] uppercase">Quadra</div><span id="countQuadra">0</span></div>
        </div>
    </div>

    <div id="mainDrawDisplay" class="container mx-auto mt-2 px-2 hidden">
        <div class="bg-white p-2 rounded-lg shadow-sm border-l-4 border-green-600 flex items-center justify-between">
            <span class="text-green-800 text-[9px] font-black uppercase tracking-tighter w-12 leading-none">Sorteio Oficial:</span>
            <div id="displayBalls" class="flex gap-1.5 flex-1 justify-center">
                </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden container mx-auto mt-2 px-2">
        <div class="bg-gray-200 p-3 rounded-lg border border-gray-300 shadow-inner text-sm">
            <div class="mb-2 bg-white p-2 rounded shadow-sm">
                <label class="block font-bold mb-1 text-[10px] text-gray-600 uppercase">Resultado do Sorteio:</label>
                <div class="flex gap-2">
                    <input type="text" id="drawInput" placeholder="Ex: 01 02..." class="flex-1 p-1 rounded border border-gray-300 text-sm">
                    <button onclick="saveDrawResult()" class="bg-blue-600 text-white px-3 py-1 rounded text-xs">Salvar</button>
                </div>
            </div>
            <div class="bg-white p-2 rounded shadow-sm">
                <label class="block font-bold mb-1 text-[10px] text-gray-600 uppercase">Importar Excel:</label>
                <input type="file" id="fileInput" accept=".xlsx, .csv" class="block w-full text-[10px] text-gray-500 file:mr-2 file:py-1 file:px-2 file:rounded file:border-0 file:bg-green-50 file:text-green-700"/>
            </div>
        </div>
    </div>

    <div class="container mx-auto mt-3 px-2">
        <div id="loading" class="text-center py-5 text-gray-500"><i class="fas fa-spinner fa-spin"></i></div>
        <div id="betsList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-2"></div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getFirestore, collection, getDocs, doc, setDoc, onSnapshot, writeBatch } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyBo8G3ZcWk4EepN0cHdVBtXc7tGOfcw-yg",
            authDomain: "inscricaosinuca.firebaseapp.com",
            projectId: "inscricaosinuca",
            storageBucket: "inscricaosinuca.firebasestorage.app",
            messagingSenderId: "338241576305",
            appId: "1:338241576305:web:288b6124384c6be4f76ad0"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        let allParticipants = [];
        let currentDraw = [];
        let isAdminAuthenticated = false;

        window.toggleAdminPanel = () => {
            if (isAdminAuthenticated) {
                document.getElementById('adminPanel').classList.toggle('hidden');
            } else {
                document.getElementById('passwordModal').classList.remove('hidden');
                document.getElementById('adminPassword').focus();
            }
        };

        window.closePasswordModal = () => {
            document.getElementById('passwordModal').classList.add('hidden');
            document.getElementById('adminPassword').value = '';
        };

        window.checkPassword = () => {
            const password = document.getElementById('adminPassword').value;
            if (password === 'bolao2025') {
                isAdminAuthenticated = true;
                closePasswordModal();
                document.getElementById('adminPanel').classList.remove('hidden');
            } else {
                document.getElementById('passwordError').classList.remove('hidden');
            }
        };

        function parseNumbersFromLine(row) {
            let allNumbers = [];
            for (let i = 1; i < row.length; i++) {
                if (row[i] !== null && row[i] !== undefined && row[i] !== '') {
                    const nums = row[i].toString().match(/\d+/g);
                    if (nums) allNumbers.push(...nums.map(n => parseInt(n, 10)));
                }
            }
            return allNumbers.filter(n => n > 0 && n <= 60);
        }

        document.getElementById('fileInput').addEventListener('change', (e) => {
            const file = e.target.files[0];
            const reader = new FileReader();
            reader.onload = async (evt) => {
                try {
                    const data = new Uint8Array(evt.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const rows = XLSX.utils.sheet_to_json(workbook.Sheets[workbook.SheetNames[0]], { header: 1 });
                    const snap = await getDocs(collection(db, "participants"));
                    const batch = writeBatch(db);
                    snap.forEach(d => batch.delete(d.ref));
                    let count = 0;
                    rows.forEach((row) => {
                        if (!row || !row[0]) return;
                        const nome = row[0].toString().trim();
                        if (["nome", "participante"].includes(nome.toLowerCase())) return;
                        const todosNumeros = parseNumbersFromLine(row);
                        let validBets = [];
                        for (let i = 0; i < todosNumeros.length; i += 6) {
                            const grupo = todosNumeros.slice(i, i + 6);
                            if (grupo.length === 6) validBets.push(grupo.sort((a, b) => a - b));
                        }
                        if (validBets.length > 0) {
                            const docRef = doc(collection(db, "participants"));
                            batch.set(docRef, { name: nome, bets: validBets.map(bet => ({ numbers: bet })) });
                            count++;
                        }
                    });
                    if (count > 0) { await batch.commit(); alert("Sucesso!"); location.reload(); }
                } catch (err) { alert("Erro: " + err.message); }
            };
            reader.readAsArrayBuffer(file);
        });

        window.saveDrawResult = async () => {
            const input = document.getElementById('drawInput').value;
            const nums = input.match(/\d+/g);
            if (!nums || nums.length < 6) return alert("Insira 6 números!");
            const sorted = nums.map(n => parseInt(n)).slice(0, 6).sort((a,b)=>a-b);
            await setDoc(doc(db, "settings", "drawResult"), { numbers: sorted });
        };

        onSnapshot(doc(db, "settings", "drawResult"), (s) => {
            currentDraw = s.exists() ? s.data().numbers : [];
            document.getElementById('drawInput').value = currentDraw.join(' ');
            render();
        });

        onSnapshot(collection(db, "participants"), (s) => {
            allParticipants = s.docs.map(d => d.data());
            document.getElementById('loading').classList.add('hidden');
            render();
        });

        function render() {
            const container = document.getElementById('betsList');
            const mainDrawDisplay = document.getElementById('mainDrawDisplay');
            const displayBalls = document.getElementById('displayBalls');
            container.innerHTML = '';
            
            if (currentDraw.length > 0) {
                mainDrawDisplay.classList.remove('hidden');
                displayBalls.innerHTML = currentDraw.map(n => `
                    <span class="draw-ball w-7 h-7 flex items-center justify-center rounded-full text-white text-xs font-bold border border-green-400">
                        ${n.toString().padStart(2, '0')}
                    </span>
                `).join('');
            } else {
                mainDrawDisplay.classList.add('hidden');
            }

            document.getElementById('countTotal').innerText = allParticipants.length;
            const rawSearch = document.getElementById('searchInput').value.toLowerCase();
            let stats = { sena: 0, quina: 0, quadra: 0 };
            const betCounts = {};
            allParticipants.forEach(p => {
                p.bets.forEach(b => {
                    const key = (b.numbers || b).slice().sort((a,b)=>a-b).join(',');
                    betCounts[key] = (betCounts[key] || 0) + 1;
                });
            });

            const searchNums = rawSearch.match(/\d{1,2}/g) || [];
            allParticipants.sort((a, b) => {
                const maxA = Math.max(...a.bets.map(bet => (bet.numbers || bet).filter(n => currentDraw.includes(n)).length));
                const maxB = Math.max(...b.bets.map(bet => (bet.numbers || bet).filter(n => currentDraw.includes(n)).length));
                return maxB - maxA;
            }).forEach(p => {
                const matchesName = p.name.toLowerCase().includes(rawSearch);
                const matchesBets = searchNums.length > 0 && p.bets.some(b => searchNums.every(sn => (b.numbers || b).includes(parseInt(sn))));
                if (rawSearch && !matchesName && !matchesBets) return;

                const card = document.createElement('div');
                card.className = "bg-white p-2 rounded shadow-sm border border-gray-200";
                let betsHtml = p.bets.map((bet, idx) => {
                    const nums = bet.numbers || bet;
                    const hits = nums.filter(n => currentDraw.includes(n)).length;
                    const isDuplicate = betCounts[nums.slice().sort((a,b)=>a-b).join(',')] > 1;
                    if (hits === 6) stats.sena++; else if (hits === 5) stats.quina++; else if (hits === 4) stats.quadra++;
                    let bg = hits === 6 ? 'sena-bg' : hits === 5 ? 'quina-bg' : hits === 4 ? 'quadra-bg' : isDuplicate ? 'duplicate-bg' : 'bg-gray-50';
                    return `
                        <div class="p-1.5 rounded mb-1 ${bg} text-center">
                            <div class="text-[8px] text-gray-400 font-bold uppercase flex justify-between px-1">
                                <span>Jogo ${idx+1} • ${hits} acertos</span>
                                ${isDuplicate ? '<span class="text-red-500 text-[7px] uppercase">Duplicado</span>' : ''}
                            </div>
                            <div class="flex flex-wrap gap-1 justify-center mt-0.5">
                                ${nums.map(n => `<span class="w-6 h-6 flex items-center justify-center rounded-full text-[10px] border ${currentDraw.includes(n) ? 'hit-number' : 'bg-white border-gray-200'}">${n.toString().padStart(2,'0')}</span>`).join('')}
                            </div>
                        </div>`;
                }).join('');
                card.innerHTML = `<h3 class="font-bold text-gray-700 text-xs border-b mb-1.5 pb-0.5 truncate">${p.name}</h3>${betsHtml}`;
                container.appendChild(card);
            });
            document.getElementById('countSena').innerText = stats.sena;
            document.getElementById('countQuina').innerText = stats.quina;
            document.getElementById('countQuadra').innerText = stats.quadra;
        }
        document.getElementById('searchInput').addEventListener('input', render);
    </script>
</body>
</html>
