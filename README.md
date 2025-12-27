<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bolão Mega da Virada 2025</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">

    <style>
        .sena-bg { background-color: #dcfce7; border: 2px solid #16a34a; }
        .quina-bg { background-color: #fef9c3; border: 2px solid #ca8a04; }
        .quadra-bg { background-color: #dbeafe; border: 2px solid #2563eb; }
        .hit-number { background-color: #16a34a; color: white; font-weight: bold; border-color: #16a34a; }

        /* 🔴 Destaque para jogos duplicados */
        .duplicate-game {
            background-color: #fee2e2 !important;
            border: 2px solid #dc2626 !important;
        }
    </style>
</head>

<body class="bg-gray-100 min-h-screen pb-20">

<nav class="bg-green-700 text-white p-4 shadow-lg sticky top-0 z-50">
    <div class="container mx-auto flex flex-col md:flex-row justify-between items-center gap-4">
        <h1 class="text-2xl font-bold flex items-center gap-2">
            <i class="fas fa-clover"></i> Bolão
        </h1>
        <input type="text" id="searchInput" placeholder="Pesquisar por nome ou dezenas..."
               class="w-full md:w-1/2 p-2 rounded text-gray-800 focus:outline-none">
        <button onclick="toggleAdminPanel()"
                class="bg-green-800 px-3 py-1 rounded text-xs border border-green-600">
            Admin
        </button>
    </div>
</nav>

<div class="container mx-auto mt-6 px-4">
    <div class="bg-white p-4 rounded-lg shadow-md grid grid-cols-3 gap-2 text-center font-bold">
        <div class="bg-green-50 p-2 rounded border border-green-500 text-green-700">
            <div class="text-[10px] uppercase">Sena</div>
            <span id="countSena">0</span>
        </div>
        <div class="bg-yellow-50 p-2 rounded border border-yellow-500 text-yellow-700">
            <div class="text-[10px] uppercase">Quina</div>
            <span id="countQuina">0</span>
        </div>
        <div class="bg-blue-50 p-2 rounded border border-blue-500 text-blue-700">
            <div class="text-[10px] uppercase">Quadra</div>
            <span id="countQuadra">0</span>
        </div>
    </div>

    <!-- 👥 Total de participantes -->
    <div class="mt-3 bg-white p-3 rounded-lg shadow text-center border border-gray-300">
        <span class="text-sm font-bold text-gray-700">
            👥 Total de participantes:
            <span id="totalParticipants" class="text-green-700">0</span>
        </span>
    </div>
</div>

<div class="container mx-auto mt-6 px-4">
    <div id="loading" class="text-center py-10 text-gray-500">
        <i class="fas fa-spinner fa-spin fa-2x"></i>
        <p class="mt-2">Carregando bolão...</p>
    </div>
    <div id="betsList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"></div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getFirestore, collection, getDocs, doc, setDoc, onSnapshot, writeBatch }
from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

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

/* 🔎 Identifica cartelas duplicadas */
function findDuplicateBets(participants) {
    const map = {};
    const duplicates = new Set();

    participants.forEach(p => {
        p.bets.forEach(b => {
            const key = (b.numbers || b).join('-');
            if (map[key]) {
                duplicates.add(key);
            } else {
                map[key] = true;
            }
        });
    });
    return duplicates;
}

onSnapshot(collection(db, "participants"), (s) => {
    allParticipants = s.docs.map(d => d.data());
    document.getElementById('totalParticipants').innerText = allParticipants.length;
    document.getElementById('loading').classList.add('hidden');
    render();
});

function render() {
    const container = document.getElementById('betsList');
    container.innerHTML = '';

    const duplicateBets = findDuplicateBets(allParticipants);

    allParticipants.forEach(p => {
        const card = document.createElement('div');
        card.className = "bg-white p-4 rounded-lg shadow border";

        let betsHtml = p.bets.map((bet, idx) => {
            const nums = bet.numbers || bet;
            const key = nums.join('-');
            const isDuplicate = duplicateBets.has(key);

            return `
            <div class="p-2 rounded mb-2 ${isDuplicate ? 'duplicate-game' : 'bg-gray-50'}">
                <div class="text-xs font-bold mb-1">
                    Jogo ${idx + 1} ${isDuplicate ? '⚠️ DUPLICADO' : ''}
                </div>
                <div class="flex gap-1 flex-wrap justify-center">
                    ${nums.map(n => `
                        <span class="w-7 h-7 flex items-center justify-center rounded-full text-xs border bg-white">
                            ${n.toString().padStart(2,'0')}
                        </span>`).join('')}
                </div>
            </div>`;
        }).join('');

        card.innerHTML = `<h3 class="font-bold mb-2">${p.name}</h3>${betsHtml}`;
        container.appendChild(card);
    });
}
</script>

</body>
</html>
