<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vanij ROI Calculator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.7.0/chart.min.js"></script>
    <style>
        :root {
            --primary-color: #2563eb;
            --secondary-color: #4f46e5;
            --accent-color: #3b82f6;
            --background-color: #f8fafc;
            --card-background: #ffffff;
            --text-color: #1e293b;
            --border-color: #e2e8f0;
        }

        body {
            font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
            background-color: var(--background-color);
            margin: 0;
            padding: 20px;
            color: var(--text-color);
            line-height: 1.6;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }

        .header {
            text-align: center;
            margin-bottom: 40px;
        }

        .header h1 {
            color: var(--primary-color);
            font-size: 2.5rem;
            margin-bottom: 10px;
        }

        .card {
            background: var(--card-background);
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
            padding: 24px;
            margin-bottom: 24px;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 24px;
            margin-bottom: 24px;
        }

        .input-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            color: var(--text-color);
            font-weight: 500;
            margin-bottom: 8px;
        }

        input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 8px;
            font-size: 1rem;
            transition: border-color 0.3s;
        }

        input:focus, select:focus {
            outline: none;
            border-color: var(--primary-color);
            box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.1);
        }

        button {
            background-color: var(--primary-color);
            color: white;
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            font-size: 1rem;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.3s;
            width: 100%;
            max-width: 200px;
            margin: 20px auto;
            display: block;
        }

        button:hover {
            background-color: var(--secondary-color);
        }

        .results {
            display: none;
            margin-top: 40px;
        }

        .chart-container {
            margin-top: 24px;
            padding: 20px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
        }

        .summary {
            background-color: var(--primary-color);
            color: white;
            padding: 20px;
            border-radius: 8px;
            margin-top: 24px;
        }

        .cost-item {
            display: flex;
            justify-content: space-between;
            padding: 10px 0;
            border-bottom: 1px solid var(--border-color);
        }

        .cost-item:last-child {
            border-bottom: none;
        }

        .cost-label {
            font-weight: 500;
        }

        .cost-value {
            font-weight: 600;
            color: var(--primary-color);
        }

        .chart-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 24px;
            margin-top: 24px;
        }

        @media (max-width: 768px) {
            .chart-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Vanij ROI Calculator</h1>
            <p>Calculate the return on investment for your Vanij implementation</p>
        </div>

        <div class="grid">
            <div class="card">
                <h2>Current Annual Costs</h2>
                <div class="input-group">
                    <label>People Cost (Salaries, Benefits)</label>
                    <input type="number" id="currentPeopleCost" placeholder="Enter amount">
                </div>
                <div class="input-group">
                    <label>Infrastructure Cost (Servers, Hardware)</label>
                    <input type="number" id="currentInfraCost" placeholder="Enter amount">
                </div>
                <div class="input-group">
                    <label>Software Licenses</label>
                    <input type="number" id="currentLicensesCost" placeholder="Enter amount">
                </div>
                <div class="input-group">
                    <label>Other Operational Costs</label>
                    <input type="number" id="currentOtherCost" placeholder="Enter amount">
                </div>
            </div>

            <div class="card">
                <h2>Vanij Implementation Details</h2>
                <div class="input-group">
                    <label>Usage Type</label>
                    <select id="usageType">
                        <option value="individual">Individual Usage</option>
                        <option value="agency">Agency Usage</option>
                        <option value="company">Company Usage</option>
                        <option value="custom">Custom - Agent Only</option>
                        <option value="customRAG">Custom - Agent + RAG</option>
                        <option value="customComponent">Custom - Agent + Component</option>
                    </select>
                </div>
                <div class="input-group">
                    <label>Number of Users</label>
                    <input type="number" id="numUsers" placeholder="Enter number of users">
                </div>
                <div class="input-group">
                    <label>Number of Agents to Deploy</label>
                    <input type="number" id="numAgents" placeholder="Enter number of agents">
                </div>
                <div class="input-group">
                    <label>Number of Flows per Agent</label>
                    <input type="number" id="numFlows" placeholder="Enter number of flows">
                </div>
            </div>
        </div>

        <button onclick="calculateROI()">Calculate ROI</button>

        <div id="results" class="results">
            <div class="card">
                <h2>Cost Analysis Results</h2>
                <div class="chart-grid">
                    <div class="chart-container">
                        <canvas id="costComparisonChart"></canvas>
                    </div>
                    <div class="chart-container">
                        <canvas id="costBreakdownChart"></canvas>
                    </div>
                </div>
                <div id="costBreakdown" class="cost-breakdown"></div>
                <div id="summary" class="summary"></div>
            </div>
        </div>
    </div>

    <script>
        let costComparisonChart = null;
        let costBreakdownChart = null;

        function calculateSubscriptionCost(usageType, numUsers) {
            switch(usageType) {
                case 'individual':
                    return 8000 * numUsers;
                case 'agency':
                    if (numUsers <= 3) return 10000;
                    if (numUsers <= 10) return numUsers * 7000;
                    return numUsers * 6000;
                case 'company':
                    if (numUsers <= 3) return 8000 * numUsers;
                    if (numUsers <= 10) return 6000 * numUsers;
                    return 5000 * numUsers;
                case 'custom':
                case 'customRAG':
                case 'customComponent':
                    return 0;
                default:
                    return 0;
            }
        }

        function calculateImplementationFee(usageType, numAgents) {
            switch(usageType) {
                case 'custom':
                    return numAgents * 4000;
                case 'customRAG':
                    return numAgents * 10000;
                case 'customComponent':
                    return numAgents * 4000;
                default:
                    return numAgents * 1000;
            }
        }

        function calculateUsageFee(usageType, numAgents, numFlows) {
            if (usageType === 'individual' || usageType === 'agency' || usageType === 'company') {
                return numAgents * numFlows * 1000;
            }
            return 0;
        }

        function calculateInfraCost(numAgents) {
            return numAgents * 5000;
        }

        function updateCharts(currentCosts, vanijCosts) {
            if (costComparisonChart) {
                costComparisonChart.destroy();
            }
            if (costBreakdownChart) {
                costBreakdownChart.destroy();
            }

            // Cost Comparison Chart
            const ctxComparison = document.getElementById('costComparisonChart').getContext('2d');
            costComparisonChart = new Chart(ctxComparison, {
                type: 'bar',
                data: {
                    labels: ['Current Costs', 'Vanij Implementation'],
                    datasets: [{
                        label: 'Total Cost Comparison',
                        data: [currentCosts.total, vanijCosts.total],
                        backgroundColor: ['#60a5fa', '#4f46e5'],
                        borderColor: ['#3b82f6', '#4338ca'],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        title: {
                            display: true,
                            text: 'Cost Comparison'
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                callback: value => '$' + value.toLocaleString()
                            }
                        }
                    }
                }
            });

            // Cost Breakdown Chart
            const ctxBreakdown = document.getElementById('costBreakdownChart').getContext('2d');
            costBreakdownChart = new Chart(ctxBreakdown, {
                type: 'doughnut',
                data: {
                    labels: ['Subscription', 'Implementation', 'Usage Fee', 'Infrastructure'],
                    datasets: [{
                        data: [
                            vanijCosts.subscription,
                            vanijCosts.implementation,
                            vanijCosts.usage,
                            vanijCosts.infrastructure
                        ],
                        backgroundColor: [
                            '#60a5fa',
                            '#4f46e5',
                            '#34d399',
                            '#f472b6'
                        ]
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        title: {
                            display: true,
                            text: 'Vanij Cost Breakdown'
                        }
                    }
                }
            });
        }

        function calculateROI() {
            // Get current costs
            const currentCosts = {
                people: Number(document.getElementById('currentPeopleCost').value),
                infrastructure: Number(document.getElementById('currentInfraCost').value),
                licenses: Number(document.getElementById('currentLicensesCost').value),
                other: Number(document.getElementById('currentOtherCost').value)
            };
            currentCosts.total = currentCosts.people + currentCosts.infrastructure + 
                               currentCosts.licenses + currentCosts.other;

            // Get implementation details
            const usageType = document.getElementById('usageType').value;
            const numUsers = Number(document.getElementById('numUsers').value);
            const numAgents = Number(document.getElementById('numAgents').value);
            const numFlows = Number(document.getElementById('numFlows').value);

            // Calculate Vanij costs
            const vanijCosts = {
                subscription: calculateSubscriptionCost(usageType, numUsers),
                implementation: calculateImplementationFee(usageType, numAgents),
                usage: calculateUsageFee(usageType, numAgents, numFlows),
                infrastructure: calculateInfraCost(numAgents)
            };
            vanijCosts.total = vanijCosts.subscription + vanijCosts.implementation + 
                              vanijCosts.usage + vanijCosts.infrastructure;

            const costDifference = currentCosts.total - vanijCosts.total;
            const roi = ((costDifference) / vanijCosts.total) * 100;

            // Update charts
            updateCharts(currentCosts, vanijCosts);

            // Display detailed breakdown
            const costBreakdown = document.getElementById('costBreakdown');
            costBreakdown.innerHTML = `
                <div class="cost-item">
                    <span class="cost-label">Current Total Cost:</span>
                    <span class="cost-value">$${currentCosts.total.toLocaleString()}</span>
                </div>
                <div class="cost-item">
                    <span class="cost-label">Vanij Total Cost:</span>
                    <span class="cost-value">$${vanijCosts.total.toLocaleString()}</span>
                </div>
                <div class="cost-item">
                    <span class="cost-label">Cost Difference:</span>
                    <span class="cost-value">$${costDifference.toLocaleString()}</span>
                </div>
                <div class="cost-item">
                    <span class="cost-label">ROI:</span>
                    <span class="cost-value">${roi.toFixed(2)}%</span>
                </div>
            `;

            // Update summary
            const summary = document.getElementById('summary');
            summary.innerHTML = `
                <h3>Implementation Summary</h3>
                <p>Type: ${usageType.charAt(0).toUpperCase() + usageType.slice(1)}</p>
                <p>Users: ${numUsers} | Agents: ${numAgents} | Flows per Agent: ${numFlows}</p>
                <p>Annual Cost Savings: $${costDifference.toLocaleString()}</p>
                <p>Return on Investment: ${roi.toFixed(2)}%</p>
            `;

            document.getElementById('results').style.display = 'block';
        }
    </script>
</body>
</html>
