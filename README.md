
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Puzzle Matematika Inklusif</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            background-color: #f0f9ff;
            touch-action: manipulation;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .card {
            transition: all 0.3s ease;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        .card:active {
            transform: scale(0.95);
        }
        .matched {
            background-color: #4ade80 !important;
            color: white !important;
            pointer-events: none;
            border: 2px solid #166534;
        }
        .selected {
            background-color: #fbbf24 !important;
            border: 4px solid #b45309;
        }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen p-4 font-sans">

    <div class="w-full max-w-md text-center">
        <h1 class="text-3xl font-extrabold text-blue-800 mb-2">Puzzle Matematika</h1>
        <p class="text-lg text-gray-700 mb-6 bg-white p-3 rounded-xl shadow-sm border border-blue-200">
            Temukan pasangan <b>Soal</b> dan <b>Jawaban</b> yang tepat!
        </p>

        <!-- Area Pesan (Benar/Salah) -->
        <div id="messageBoard" class="h-12 mb-4 flex items-center justify-center text-xl font-bold text-blue-600 rounded-lg">
            Ayo mulai bermain!
        </div>

        <!-- Papan Puzzle (Grid) -->
        <div id="gameBoard" class="grid grid-cols-2 gap-4 mb-8">
            <!-- Kartu akan dimunculkan di sini oleh JavaScript -->
        </div>

        <!-- Tombol Ulang -->
        <button onclick="initGame()" class="w-full bg-blue-600 text-white font-bold text-xl py-4 rounded-2xl shadow-lg active:bg-blue-800 transition-colors">
            🔄 Main Ulang
        </button>
    </div>

    <script>
        // Data Soal dan Jawaban (Bisa Anda ganti/tambah sesuai materi kelas)
        const puzzleData = [
            { id: 1, text: "5 + 3", type: "soal", pair: 1 },
            { id: 2, text: "8", type: "jawaban", pair: 1 },
            { id: 3, text: "10 - 4", type: "soal", pair: 2 },
            { id: 4, text: "6", type: "jawaban", pair: 2 },
            { id: 5, text: "2 x 4", type: "soal", pair: 3 },
            { id: 6, text: "8", type: "jawaban", pair: 3 }, // Menggunakan angka yang sama untuk melatih fokus
            { id: 7, text: "12 ÷ 2", type: "soal", pair: 4 },
            { id: 8, text: "6", type: "jawaban", pair: 4 }
        ];

        let cards = [];
        let firstSelection = null;
        let secondSelection = null;
        let lockBoard = false;
        let matchesFound = 0;

        function shuffle(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        function initGame() {
            const gameBoard = document.getElementById('gameBoard');
            const messageBoard = document.getElementById('messageBoard');
            
            gameBoard.innerHTML = '';
            messageBoard.textContent = 'Ayo mulai bermain!';
            messageBoard.className = 'h-12 mb-4 flex items-center justify-center text-xl font-bold text-blue-600 rounded-lg';
            
            firstSelection = null;
            secondSelection = null;
            lockBoard = false;
            matchesFound = 0;
            
            // Gandakan array agar tidak mengubah data asli, lalu acak
            cards = shuffle([...puzzleData]);

            cards.forEach((card, index) => {
                const btn = document.createElement('button');
                btn.className = 'card bg-white text-blue-900 border-2 border-blue-300 rounded-2xl text-2xl font-bold h-24 flex items-center justify-center cursor-pointer select-none';
                btn.textContent = card.text;
                btn.dataset.index = index;
                btn.dataset.pair = card.pair;
                btn.onclick = () => handleCardClick(btn, card);
                
                gameBoard.appendChild(btn);
            });
        }

        function handleCardClick(btn, card) {
            // Cegah klik ganda atau saat papan terkunci
            if (lockBoard || btn === firstSelection?.btn || btn.classList.contains('matched')) return;

            btn.classList.add('selected');

            if (!firstSelection) {
                // Pilihan pertama
                firstSelection = { btn, card };
                document.getElementById('messageBoard').textContent = 'Pilih pasangannya...';
                document.getElementById('messageBoard').classList.remove('bg-red-100', 'text-red-600');
            } else {
                // Pilihan kedua
                secondSelection = { btn, card };
                lockBoard = true;
                
                checkMatch();
            }
        }

        function checkMatch() {
            const isMatch = firstSelection.card.pair === secondSelection.card.pair;
            const msgBoard = document.getElementById('messageBoard');

            if (isMatch) {
                // Jika Benar
                msgBoard.textContent = '✅ Benar sekali!';
                msgBoard.className = 'h-12 mb-4 flex items-center justify-center text-xl font-bold text-green-700 bg-green-100 rounded-lg';
                
                firstSelection.btn.classList.remove('selected');
                secondSelection.btn.classList.remove('selected');
                firstSelection.btn.classList.add('matched');
                secondSelection.btn.classList.add('matched');
                
                matchesFound++;
                resetSelection();

                // Cek apakah game selesai
                if (matchesFound === (puzzleData.length / 2)) {
                    setTimeout(() => {
                        msgBoard.textContent = '🎉 HORE! SEMUA SELESAI! 🎉';
                        msgBoard.className = 'h-12 mb-4 flex items-center justify-center text-2xl font-extrabold text-white bg-blue-600 rounded-lg animate-pulse';
                    }, 500);
                }
            } else {
                // Jika Salah
                msgBoard.textContent = '❌ Kurang tepat, coba lagi!';
                msgBoard.className = 'h-12 mb-4 flex items-center justify-center text-xl font-bold text-red-600 bg-red-100 rounded-lg';
                
                setTimeout(() => {
                    firstSelection.btn.classList.remove('selected');
                    secondSelection.btn.classList.remove('selected');
                    resetSelection();
                    msgBoard.textContent = 'Ayo cari lagi!';
                    msgBoard.className = 'h-12 mb-4 flex items-center justify-center text-xl font-bold text-gray-500 rounded-lg';
                }, 1200); // Waktu tunda sebelum kartu tertutup kembali
            }
        }

        function resetSelection() {
            firstSelection = null;
            secondSelection = null;
            lockBoard = false;
        }

        // Mulai game saat halaman dimuat
        window.onload = initGame;
    </script>
</body>
</html>

```
