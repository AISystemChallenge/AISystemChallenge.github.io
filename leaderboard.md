---
title:
feature_text: |
  <!--# Samsung AI System Challenge 2026-->
feature_image: "/assets/logos/banner_AISC2026_20.png"
excerpt:
---

<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Leaderboard</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        .leaderboard-card {
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            margin-bottom: 1rem;
            padding: 1rem;
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }

        .table-header {
            border-bottom: 1px solid #e9ecef;
            padding-bottom: 8px;
            margin-top: 20px;
        }

        .table-header .row {
            margin: 0;
        }

        .table-header .col-2, 
        .table-header .col-10 {
            padding: 0 15px;
        }
        
        .leaderboard-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
        }
        
        .rank-number {
            font-weight: bold;
            font-size: 1.2rem;
            min-width: 3rem;
            text-align: center;
        }
        
        .team-name {
            font-weight: 600;
            font-size: 1.1rem;
            overflow-wrap: anywhere;
        }
        

        
        .preparing-card {
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            padding: 2rem;
            text-align: center;
            margin-bottom: 2rem;
        }
        
        .countdown-timer {
            font-family: 'Courier New', monospace;
            font-size: 2rem;
            font-weight: bold;
            color: #007bff;
            margin: 1rem 0;
        }
        
        .dday {
            font-size: 1.5rem;
            font-weight: bold;
            color: #dc3545;
            margin-left: 1rem;
        }
        
        .open-info {
            color: #6c757d;
            font-size: 1.1rem;
            margin: 1rem 0;
        }
        
        .notice {
            margin-top: 2rem;
            padding: 1rem;
            background-color: #f8f9fa;
            border-radius: 8px;
            border-left: 4px solid #007bff;
        }
        
        .notice h6 {
            margin-bottom: 0.5rem;
            color: #495057;
        }
        
        .notice p {
            margin-bottom: 0;
            color: #6c757d;
            font-size: 0.9rem;
        }
        
        .spinner-border-sm {
            width: 1rem;
            height: 1rem;
        }

        @media (max-width: 768px) {
            .countdown-timer {
                font-size: 1.5rem;
            }
            .dday {
                font-size: 1.2rem;
                display: block;
                margin-left: 0;
                margin-top: 0.5rem;
            }
        }

        header .container {
          max-width: 1400px;
          width: 90%;
          font-family: samsung, serif;
          font-size: 17px;
          line-height: 30px;
        }
        /* Header font color */
        header .container a {
          color: #37B9F8;
          text-decoration: none;
        }
        header .container a:hover {
          color: #206B8F;
        }
        /* Leaderboard는 마우스를 올려도 회색 유지 */
        header .container a[href="/leaderboard/"],
          header .container a[href="/leaderboard/"]:hover {
          color: #a8adac;
        }
    </style>
</head>
<body class="bg-light">
    <div class="container mt-4">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <h1 class="text-center mb-4">🏆 Leaderboard</h1>
                
                <div id="loading" class="text-center">
                    <div class="spinner-border text-primary" role="status">
                        <span class="visually-hidden">로딩 중...</span>
                    </div>
                </div>
                
                <div id="preparing" class="d-none">
                    <div class="preparing-card">
                        <h2>🚧 준비 중입니다</h2>
                        <div class="countdown-timer" id="countdown">
                            <span id="time-display">00:00:00.000</span>
                            <span class="dday" id="dday-display">D-0</span>
                        </div>
                        <div class="open-info">11/21 00시 오픈</div>
                        <p class="text-muted">Coming soon!</p>
                    </div>
                </div>

                <div id="leaderboard-header" class="table-header mb-2 d-none">
                    <div class="row text-muted small">
                        <div class="col-2 text-center"><strong>Rank</strong></div>
                        <div class="col-10"><strong>Team</strong></div>
                    </div>
                </div>

                <div id="leaderboard" class="d-none">
                    <!-- 리더보드 내용이 여기에 표시됩니다 -->
                </div>

                <div id="error" class="alert alert-danger d-none" role="alert">
                    <strong>오류!</strong> 데이터를 불러오는 중 문제가 발생했습니다.
                </div>
                
                <div class="notice">
                    <h6>주의 사항</h6>
                    <p>※ Leaderboard에 표시된 결과는 참고용이며, 최종 순위는 내부 평가 기준을 통해 결정됩니다.</p>
                </div>
            </div>
        </div>
    </div>

    <script>
        const RANK_CSV = '/rank/rank2026.csv';
        const OPEN_TIME = new Date('2026-11-21T00:00:00+09:00');
        let countdownInterval;
        let refreshInterval;

        function updateCountdown() {
            const remaining = OPEN_TIME.getTime() - Date.now();
            if (remaining <= 0) {
                clearInterval(countdownInterval);
                openLeaderboard();
                return;
            }

            const hours = Math.floor(remaining / 3600000);
            const minutes = Math.floor(remaining / 60000) % 60;
            const seconds = Math.floor(remaining / 1000) % 60;
            const milliseconds = remaining % 1000;
            const time = [hours, minutes, seconds].map((part, index) =>
                String(part).padStart(index === 0 ? 3 : 2, '0')
            ).join(':');
            document.getElementById('time-display').textContent =
                time + '.' + String(milliseconds).padStart(3, '0');
            document.getElementById('dday-display').textContent = 'D-' + Math.ceil(remaining / 86400000);
        }

        function parseRankCsv(csv) {
            const [header, ...rows] = csv.replace(/\r?\n$/, '').split(/\r?\n/);
            if (header !== 'rank,team' || rows.length > 10) {
                throw new Error('Invalid leaderboard CSV');
            }
            const seen = new Set();
            return rows.map((row, index) => {
                const match = /^([1-9]|10),"((?:[^"]|"")*)"$/.exec(row);
                const rank = index + 1;
                const team = match?.[2].replace(/""/g, '"');
                if (Number(match?.[1]) !== rank || !team?.trim() || seen.has(team)) {
                    throw new Error('Invalid leaderboard row');
                }
                seen.add(team);
                return { rank, team };
            });
        }

        function renderLeaderboard(rows) {
            const leaderboard = document.getElementById('leaderboard');
            const cards = rows.map(({ rank, team }) => {
                const card = document.createElement('div');
                card.className = 'leaderboard-card row align-items-center mx-0';
                const rankLabel = document.createElement('span');
                rankLabel.className = 'rank-number col-2';
                rankLabel.textContent = rank;
                const teamLabel = document.createElement('span');
                teamLabel.className = 'team-name col-10';
                teamLabel.textContent = team;
                card.append(rankLabel, teamLabel);
                return card;
            });
            if (!cards.length) {
                const empty = document.createElement('p');
                empty.className = 'text-muted text-center mt-4';
                empty.textContent = '공개할 순위가 아직 없습니다.';
                cards.push(empty);
            }
            leaderboard.replaceChildren(...cards);
            leaderboard.classList.remove('d-none');
            document.getElementById('leaderboard-header').classList.toggle('d-none', !rows.length);
            document.getElementById('error').classList.add('d-none');
        }

        async function fetchLeaderboard() {
            try {
                const response = await fetch(RANK_CSV + '?v=' + Date.now(), { cache: 'no-store' });
                if (!response.ok) throw new Error('Leaderboard request failed');
                renderLeaderboard(parseRankCsv(await response.text()));
            } catch (error) {
                console.error('Error fetching leaderboard:', error);
                document.getElementById('error').classList.remove('d-none');
            } finally {
                document.getElementById('loading').classList.add('d-none');
            }
        }

        function openLeaderboard() {
            document.getElementById('preparing').classList.add('d-none');
            fetchLeaderboard();
            refreshInterval = setInterval(fetchLeaderboard, 60000);
        }

        document.addEventListener('DOMContentLoaded', () => {
            if (Date.now() >= OPEN_TIME.getTime()) {
                openLeaderboard();
                return;
            }
            document.getElementById('loading').classList.add('d-none');
            document.getElementById('preparing').classList.remove('d-none');
            countdownInterval = setInterval(updateCountdown, 100);
            updateCountdown();
        });

        window.addEventListener('beforeunload', () => {
            clearInterval(countdownInterval);
            clearInterval(refreshInterval);
        });
    </script>
</body>
</html>
