<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Interactive League Analyzer</title>
    <!-- Chosen Palette: Warm Neutrals & Slate Blue -->
    <!-- Application Structure Plan: The application is designed as an interactive dashboard. The layout uses a two-column grid on larger screens. The main, wider column on the left contains the primary user workflow: Matchday selection via a slider, a section for inputting that day's results, and the main league table. This flow is intuitive: select a day, enter scores, update table. The right column provides secondary, summarized information: key league-wide statistics and a bar chart visualizing team points. This structure was chosen to separate the core data entry/viewing task from the supplementary analysis, preventing information overload while keeping key insights readily available. The user can focus on the day-to-day results while still getting a high-level visual overview of the league standings. -->
    <!-- Visualization & Content Choices: 
        - Report Info: League standings (P, W, D, L, etc.). Goal: Inform, Organize, Compare. Viz/Method: Interactive HTML Table. Interaction: Column header sorting. Justification: Standard, clear format for ranked data. Sorting allows user-driven analysis.
        - Report Info: Daily match results. Goal: Data Entry. Viz/Method: HTML Form Inputs with Dropdowns. Interaction: User types in scores and selects teams. Justification: Direct and universal method for data input, now with added flexibility for team selection.
        - Report Info: Team performance comparison. Goal: Compare. Viz/Method: Bar Chart. Interaction: Updates on table refresh, tooltips on hover. Justification: Provides a quick visual comparison of team points, easier to scan than the table. Library: Chart.js (Canvas).
        - Report Info: Navigating through the season. Goal: Change context. Viz/Method: HTML Range Slider. Interaction: User drags slider to select a day. Justification: Intuitive for navigating a fixed timeline (38 days).
        - Report Info: Key league stats (total goals, etc.). Goal: Inform. Viz/Method: Styled text blocks (Key Value pairs). Interaction: Updates on table refresh. Justification: Provides high-level summary stats at a glance.
        - Report Info: Team last 10 match performance. Goal: Inform, Compare. Viz/Method: Colored boxes (HTML/CSS). Interaction: Visual representation of recent form. Justification: Quick, intuitive visual summary of recent wins, draws, losses.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        .table-cell {
            padding: 0.75rem;
            text-align: center;
            white-space: nowrap;
        }
        .table-header {
            padding: 0.75rem;
            text-align: center;
            font-weight: 600;
            cursor: pointer;
            user-select: none;
        }
        .table-header:hover {
            background-color: #f1f5f9; /* slate-100 */
        }
        .btn-primary {
            background-color: #2563eb; /* blue-600 */
            color: white;
            transition: background-color 0.3s;
        }
        .btn-primary:hover {
            background-color: #1d4ed8; /* blue-700 */
        }
        .team-select {
            @apply w-24 p-1 border rounded-md text-center;
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800">
    <main class="container mx-auto p-4 sm:p-6 lg:p-8">
        <header class="text-center mb-8">
            <h1 class="text-4xl font-bold text-slate-900">Interactive League Analyzer</h1>
            <p class="mt-2 text-lg text-slate-600">Track daily matches, view live standings, and visualize team performance.</p>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <div class="lg:col-span-2 space-y-8">
                <section id="controls" class="bg-white p-6 rounded-xl shadow-sm">
                    <h2 class="text-2xl font-bold mb-4 text-slate-900">Season Controls</h2>
                    <div class="space-y-4">
                        <div>
                             <label for="matchday-slider" class="block mb-2 font-medium text-slate-700">Select Matchday: <span id="matchday-label" class="font-bold text-blue-600">1</span>/38</label>
                             <input type="range" id="matchday-slider" min="1" max="38" value="1" class="w-full h-2 bg-slate-200 rounded-lg appearance-none cursor-pointer">
                        </div>
                        <button id="update-table-btn" class="w-full sm:w-auto px-6 py-3 font-semibold rounded-lg btn-primary">
                            Update League Table
                        </button>
                    </div>
                     <p class="mt-4 text-sm text-slate-500">
                        This tool simulates a full 38-matchday season where each of the 20 teams plays every other team both home and away. Use the slider to select a matchday, enter the scores for the generated fixtures below, and click "Update League Table" to see the live standings. You can now also change the home and away teams for each match.
                    </p>
                </section>
                
                <section id="fixtures-section" class="bg-white p-6 rounded-xl shadow-sm">
                    <h2 class="text-2xl font-bold mb-4 text-slate-900">Matchday <span id="fixtures-day-label">1</span> Fixtures & Results</h2>
                    <p class="text-slate-600 mb-4">Enter the scores and select teams for the matches below. Leave scores blank for unplayed matches.</p>
                    <div id="fixtures-container" class="space-y-3"></div>
                </section>

                <section id="league-table-section" class="bg-white p-6 rounded-xl shadow-sm">
                    <h2 class="text-2xl font-bold mb-4 text-slate-900">League Table</h2>
                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-slate-200">
                            <thead class="bg-slate-50">
                                <tr>
                                    <th class="table-header" data-sort="rank">#</th>
                                    <th class="table-header text-left" data-sort="name">Team</th>
                                    <th class="table-header" data-sort="p">P</th>
                                    <th class="table-header" data-sort="w">W</th>
                                    <th class="table-header" data-sort="d">D</th>
                                    <th class="table-header" data-sort="l">L</th>
                                    <th class="table-header" data-sort="gf">GF</th>
                                    <th class="table-header" data-sort="ga">GA</th>
                                    <th class="table-header" data-sort="gd">GD</th>
                                    <th class="table-header" data-sort="pts">Pts</th>
                                    <th class="table-header">Last 10</th>
                                </tr>
                            </thead>
                            <tbody id="league-table-body" class="bg-white divide-y divide-slate-200">
                                <!-- Table rows will be generated by JavaScript -->
                            </tbody>
                        </table>
                    </div>
                </section>
            </div>
            
            <div class="lg:col-span-1 space-y-8">
                <section id="stats-summary" class="bg-white p-6 rounded-xl shadow-sm">
                    <h2 class="text-2xl font-bold mb-4 text-slate-900">League Summary</h2>
                     <p class="text-slate-600 mb-4">
                        This section provides a high-level overview of the league's activity based on the results you've entered. It helps in understanding the overall offensive and defensive trends across all teams.
                    </p>
                    <div class="grid grid-cols-2 gap-4 text-center">
                        <div class="bg-slate-100 p-4 rounded-lg">
                            <p class="text-sm font-medium text-slate-500">Total Goals</p>
                            <p id="total-goals" class="text-3xl font-bold text-slate-900">0</p>
                        </div>
                        <div class="bg-slate-100 p-4 rounded-lg">
                            <p class="text-sm font-medium text-slate-500">Matches Played</p>
                            <p id="matches-played" class="text-3xl font-bold text-slate-900">0</p>
                        </div>
                        <div class="bg-slate-100 p-4 rounded-lg col-span-2">
                            <p class="text-sm font-medium text-slate-500">Goals per Match</p>
                            <p id="goals-per-match" class="text-3xl font-bold text-slate-900">0.00</p>
                        </div>
                    </div>
                </section>

                <section id="performance-chart-section" class="bg-white p-6 rounded-xl shadow-sm">
                    <h2 class="text-2xl font-bold mb-4 text-slate-900">Team Points Comparison</h2>
                    <p class="text-slate-600 mb-4">
                        This chart visualizes the total points for each team, providing a quick comparison of their performance in the league so far. It dynamically updates every time you recalculate the league table.
                    </p>
                    <div class="chart-container relative w-full h-96 max-h-[400px] sm:h-auto sm:max-h-[500px] md:max-h-[600px] mx-auto max-w-2xl">
                        <canvas id="performance-chart"></canvas>
                    </div>
                </section>
            </div>
        </div>
    </main>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const TEAMS = ['ARS', 'AST', 'BHA', 'BOU', 'BRE', 'CHE', 'CRY', 'EVE', 'FOR', 'FUL', 'IPS', 'LEI', 'LIV', 'MCI', 'MUN', 'NEW', 'SOU', 'TOT', 'WHU', 'WOL'];
            let teamStats = {};
            let fixtures = []; // Stores the initial generated schedule
            let results = {}; // Stores user-inputted results, including potentially modified teams and scores
            let performanceChart;

            let currentSort = {
                column: 'pts',
                direction: 'desc'
            };

            const slider = document.getElementById('matchday-slider');
            const matchdayLabel = document.getElementById('matchday-label');
            const fixturesDayLabel = document.getElementById('fixtures-day-label');
            const fixturesContainer = document.getElementById('fixtures-container');
            const updateBtn = document.getElementById('update-table-btn');
            const leagueTableBody = document.getElementById('league-table-body');
            
            const totalGoalsEl = document.getElementById('total-goals');
            const matchesPlayedEl = document.getElementById('matches-played');
            const goalsPerMatchEl = document.getElementById('goals-per-match');

            function initializeApp() {
                generateFixtures();
                resetTeamStats();
                displayFixtures(1);
                initializeChart(); 
                updateLeagueTable(); 
                addEventListeners();
            }

            function addEventListeners() {
                slider.addEventListener('input', (e) => {
                    const day = parseInt(e.target.value);
                    matchdayLabel.textContent = day;
                    fixturesDayLabel.textContent = day;
                    saveCurrentResults(); // Save results of the current day before switching
                    displayFixtures(day); // Display fixtures for the new day
                });
                updateBtn.addEventListener('click', () => {
                    saveCurrentResults(); // Save current day's results
                    updateLeagueTable(); // Recalculate and render table
                });
                document.querySelectorAll('.table-header').forEach(header => {
                    header.addEventListener('click', (e) => {
                        const column = e.target.dataset.sort;
                        if (currentSort.column === column) {
                            currentSort.direction = currentSort.direction === 'asc' ? 'desc' : 'asc';
                        } else {
                            currentSort.column = column;
                            currentSort.direction = ['name'].includes(column) ? 'asc' : 'desc';
                        }
                        renderTable();
                    });
                });
            }

            function generateFixtures() {
                const teams = [...TEAMS];
                if (teams.length % 2 !== 0) {
                    teams.push(null); 
                }
                const numTeams = teams.length;
                const rounds = numTeams - 1;
                const halfSeasonFixtures = [];

                for (let round = 0; round < rounds; round++) {
                    const dailyFixtures = [];
                    for (let match = 0; match < numTeams / 2; match++) {
                        const home = teams[match];
                        const away = teams[numTeams - 1 - match];
                        if (home && away) {
                             dailyFixtures.push({ home, away });
                        }
                    }
                    halfSeasonFixtures.push(dailyFixtures);
                    
                    const lastTeam = teams.pop();
                    teams.splice(1, 0, lastTeam);
                }

                fixtures = [...halfSeasonFixtures];
                halfSeasonFixtures.forEach(dayFixtures => {
                    const reverseDayFixtures = dayFixtures.map(({ home, away }) => ({ home: away, away: home }));
                    fixtures.push(reverseDayFixtures);
                });
            }
            
            function resetTeamStats() {
                teamStats = {};
                TEAMS.forEach(team => {
                    teamStats[team] = { name: team, p: 0, w: 0, d: 0, l: 0, gf: 0, ga: 0, gd: 0, pts: 0, last10Results: [] };
                });
            }

            function generateTeamOptions(selectedTeam) {
                return TEAMS.map(team => 
                    `<option value="${team}" ${team === selectedTeam ? 'selected' : ''}>${team}</option>`
                ).join('');
            }

            function displayFixtures(day) {
                fixturesContainer.innerHTML = '';
                const dayFixtures = fixtures[day - 1] || [];
                results[day] = results[day] || []; 

                dayFixtures.forEach((match, index) => {
                    const currentMatchResult = results[day][index] || { 
                        homeTeam: match.home, 
                        awayTeam: match.away, 
                        homeScore: '', 
                        awayScore: '' 
                    };

                    const fixtureEl = document.createElement('div');
                    fixtureEl.className = 'grid grid-cols-5 gap-2 items-center text-center';
                    fixtureEl.innerHTML = `
                        <select class="team-select text-right col-span-2" data-day="${day}" data-match-index="${index}" data-team-type="home">
                            ${generateTeamOptions(currentMatchResult.homeTeam)}
                        </select>
                        <div class="flex items-center justify-center gap-1">
                            <input type="number" min="0" class="w-12 text-center border rounded-md p-1 fixture-input" data-day="${day}" data-match-index="${index}" data-score-type="home" value="${currentMatchResult.homeScore}">
                            <span>-</span>
                            <input type="number" min="0" class="w-12 text-center border rounded-md p-1 fixture-input" data-day="${day}" data-match-index="${index}" data-score-type="away" value="${currentMatchResult.awayScore}">
                        </div>
                        <select class="team-select text-left col-span-2" data-day="${day}" data-match-index="${index}" data-team-type="away">
                            ${generateTeamOptions(currentMatchResult.awayTeam)}
                        </select>
                    `;
                    fixturesContainer.appendChild(fixtureEl);
                });
            }

            function saveCurrentResults() {
                const day = parseInt(slider.value);
                const scoreInputs = document.querySelectorAll('.fixture-input');
                const teamSelects = document.querySelectorAll('.team-select');
                
                results[day] = results[day] || [];

                scoreInputs.forEach(input => {
                    const matchIndex = parseInt(input.dataset.matchIndex);
                    const scoreType = input.dataset.scoreType;
                    
                    if (!results[day][matchIndex]) {
                         results[day][matchIndex] = { homeTeam: '', awayTeam: '', homeScore: '', awayScore: '' };
                    }
                    if(scoreType === 'home') {
                        results[day][matchIndex].homeScore = input.value;
                    } else {
                        results[day][matchIndex].awayScore = input.value;
                    }
                });

                teamSelects.forEach(select => {
                    const matchIndex = parseInt(select.dataset.matchIndex);
                    const teamType = select.dataset.teamType;

                    if (!results[day][matchIndex]) {
                         results[day][matchIndex] = { homeTeam: '', awayTeam: '', homeScore: '', awayScore: '' };
                    }
                    if(teamType === 'home') {
                        results[day][matchIndex].homeTeam = select.value;
                    } else {
                        results[day][matchIndex].awayTeam = select.value;
                    }
                });
            }

            function updateLeagueTable() {
                resetTeamStats(); // Reset all stats, including last10Results
                let totalGoals = 0;
                let matchesPlayed = 0;

                // Process results day by day to correctly build last10Results chronologically
                for (let day = 1; day <= 38; day++) {
                    const dayResults = results[day] || [];
                    dayResults.forEach((result, index) => {
                        const homeScore = parseInt(result.homeScore);
                        const awayScore = parseInt(result.awayScore);

                        if (!isNaN(homeScore) && !isNaN(awayScore) && result.homeTeam && result.awayTeam) {
                            const homeTeamStats = teamStats[result.homeTeam];
                            const awayTeamStats = teamStats[result.awayTeam];

                            if (homeTeamStats && awayTeamStats && result.homeTeam !== result.awayTeam) {
                                homeTeamStats.p += 1;
                                awayTeamStats.p += 1;
                                homeTeamStats.gf += homeScore;
                                awayTeamStats.gf += awayScore;
                                homeTeamStats.ga += awayScore;
                                awayTeamStats.ga += homeScore;

                                let homeOutcome, awayOutcome;

                                if (homeScore > awayScore) {
                                    homeTeamStats.w += 1;
                                    homeTeamStats.pts += 3;
                                    awayTeamStats.l += 1;
                                    homeOutcome = 'W';
                                    awayOutcome = 'L';
                                } else if (awayScore > homeScore) {
                                    awayTeamStats.w += 1;
                                    awayTeamStats.pts += 3;
                                    homeTeamStats.l += 1;
                                    homeOutcome = 'L';
                                    awayOutcome = 'W';
                                } else {
                                    homeTeamStats.d += 1;
                                    awayTeamStats.d += 1;
                                    homeTeamStats.pts += 1;
                                    awayTeamStats.pts += 1;
                                    homeOutcome = 'D';
                                    awayOutcome = 'D';
                                }
                                
                                // Add outcome to last10Results and maintain size
                                homeTeamStats.last10Results.push(homeOutcome);
                                if (homeTeamStats.last10Results.length > 10) {
                                    homeTeamStats.last10Results.shift();
                                }
                                awayTeamStats.last10Results.push(awayOutcome);
                                if (awayTeamStats.last10Results.length > 10) {
                                    awayTeamStats.last10Results.shift();
                                }

                                totalGoals += homeScore + awayScore;
                                matchesPlayed++;
                            }
                        }
                    });
                }
                
                Object.values(teamStats).forEach(team => {
                    team.gd = team.gf - team.ga;
                });
                
                totalGoalsEl.textContent = totalGoals;
                matchesPlayedEl.textContent = matchesPlayed;
                goalsPerMatchEl.textContent = matchesPlayed > 0 ? (totalGoals / matchesPlayed).toFixed(2) : '0.00';

                renderTable();
                updateChart();
            }
            
            function sortStats(stats) {
                return stats.sort((a, b) => {
                    if (currentSort.direction === 'asc') {
                        [a, b] = [b, a]; 
                    }

                    if (a[currentSort.column] < b[currentSort.column]) return 1;
                    if (a[currentSort.column] > b[currentSort.column]) return -1;
                    
                    if (currentSort.column !== 'pts') {
                        if (a.pts < b.pts) return 1;
                        if (a.pts > b.pts) return -1;
                    }
                    if (currentSort.column !== 'gd') {
                        if (a.gd < b.gd) return 1;
                        if (a.gd > b.gd) return -1;
                    }
                    if (currentSort.column !== 'gf') {
                         if (a.gf < b.gf) return 1;
                         if (a.gf > b.gf) return -1;
                    }

                    if (a.name > b.name) return 1;
                    if (a.name < b.name) return -1;

                    return 0;
                });
            }

            function renderTable() {
                leagueTableBody.innerHTML = '';
                const sortedStats = sortStats(Object.values(teamStats));
                
                sortedStats.forEach((team, index) => {
                    const rank = index + 1;
                    const row = document.createElement('tr');
                    row.className = 'hover:bg-slate-50';
                    row.innerHTML = `
                        <td class="table-cell font-medium">${rank}</td>
                        <td class="table-cell text-left font-semibold">${team.name}</td>
                        <td class="table-cell">${team.p}</td>
                        <td class="table-cell text-green-600">${team.w}</td>
                        <td class="table-cell text-yellow-600">${team.d}</td>
                        <td class="table-cell text-red-600">${team.l}</td>
                        <td class="table-cell">${team.gf}</td>
                        <td class="table-cell">${team.ga}</td>
                        <td class="table-cell font-medium">${team.gd}</td>
                        <td class="table-cell font-bold text-slate-900">${team.pts}</td>
                        <td class="table-cell">
                            <div class="flex justify-center gap-0.5">
                                ${team.last10Results.map(result => `
                                    <div class="w-4 h-4 rounded-sm
                                        ${result === 'W' ? 'bg-green-500' : ''}
                                        ${result === 'D' ? 'bg-yellow-400' : ''}
                                        ${result === 'L' ? 'bg-red-500' : ''}
                                    "></div>
                                `).join('')}
                            </div>
                        </td>
                    `;
                    leagueTableBody.appendChild(row);
                });
            }
            
            function initializeChart() {
                const ctx = document.getElementById('performance-chart').getContext('2d');
                performanceChart = new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: [],
                        datasets: [{
                            label: 'Points',
                            data: [],
                            backgroundColor: 'rgba(59, 130, 246, 0.7)', 
                            borderColor: 'rgba(37, 99, 235, 1)', 
                            borderWidth: 1
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        indexAxis: 'y',
                        scales: {
                            x: {
                                beginAtZero: true,
                                grid: {
                                    color: '#e2e8f0' 
                                }
                            },
                            y: {
                                grid: {
                                    display: false
                                }
                            }
                        },
                        plugins: {
                            legend: {
                                display: false
                            },
                            tooltip: {
                                mode: 'index',
                                intersect: false,
                            },
                        }
                    }
                });
            }

            function updateChart() {
                const sortedStats = sortStats(Object.values(teamStats)).reverse(); 
                performanceChart.data.labels = sortedStats.map(t => t.name);
                performanceChart.data.datasets[0].data = sortedStats.map(t => t.pts);
                performanceChart.update();
            }

            initializeApp();
        });
    </script>
</body>
</html>
