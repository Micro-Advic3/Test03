<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tableau Insights - POC Demo</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%);
            min-height: 100vh;
            padding: 40px 20px;
            color: #333;
        }

        .container {
            max-width: 1800px;
            margin: 0 auto;
        }

        /* Header */
        header {
            text-align: center;
            margin-bottom: 50px;
            color: white;
        }

        h1 {
            font-size: 52px;
            margin-bottom: 15px;
            text-shadow: 3px 3px 6px rgba(0,0,0,0.5);
            font-weight: 700;
        }

        .subtitle {
            font-size: 20px;
            opacity: 0.95;
            font-weight: 300;
        }

        /* Status Banner */
        .status-banner {
            background: white;
            padding: 25px;
            border-radius: 15px;
            text-align: center;
            margin-bottom: 40px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
            font-size: 18px;
        }

        .status-loading {
            color: #f59e0b;
            font-weight: 600;
        }

        .status-success {
            color: #10b981;
            font-weight: 700;
        }

        .status-error {
            color: #ef4444;
            font-weight: 700;
        }

        /* Summary Cards */
        .summary-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 25px;
            margin-bottom: 50px;
        }

        .summary-card {
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
            text-align: center;
        }

        .summary-card .icon {
            font-size: 40px;
            margin-bottom: 15px;
        }

        .summary-card .label {
            font-size: 13px;
            text-transform: uppercase;
            letter-spacing: 1.5px;
            color: #64748b;
            font-weight: 600;
            margin-bottom: 10px;
        }

        .summary-card .value {
            font-size: 48px;
            font-weight: 800;
            color: #008043;
        }

        /* Dashboard Cards Section */
        .section-title {
            color: white;
            font-size: 32px;
            margin-bottom: 30px;
            padding-bottom: 15px;
            border-bottom: 3px solid #008043;
            font-weight: 700;
        }

        /* Horizontal Scroll Container (Netflix Style) */
        .scroll-container {
            display: flex;
            gap: 25px;
            overflow-x: auto;
            padding: 20px 10px 30px 10px;
            margin-bottom: 50px;
            scroll-behavior: smooth;
        }

        .scroll-container::-webkit-scrollbar {
            height: 10px;
        }

        .scroll-container::-webkit-scrollbar-track {
            background: rgba(255,255,255,0.1);
            border-radius: 10px;
        }

        .scroll-container::-webkit-scrollbar-thumb {
            background: #008043;
            border-radius: 10px;
        }

        /* Dashboard Card */
        .dashboard-card {
            background: white;
            min-width: 350px;
            max-width: 350px;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 15px 50px rgba(0,0,0,0.3);
            transition: all 0.4s ease;
            position: relative;
            overflow: hidden;
            flex-shrink: 0;
        }

        .dashboard-card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 5px;
            background: linear-gradient(90deg, #008043, #00b359);
        }

        .dashboard-card:hover {
            transform: translateY(-10px);
            box-shadow: 0 25px 60px rgba(0,0,0,0.4);
        }

        .card-tag {
            display: inline-block;
            background: linear-gradient(135deg, #008043, #00b359);
            color: white;
            padding: 6px 16px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: 700;
            margin-bottom: 15px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .card-title {
            font-size: 22px;
            font-weight: 700;
            color: #1e293b;
            margin-bottom: 12px;
            line-height: 1.3;
        }

        .card-subtitle {
            font-size: 13px;
            color: #64748b;
            margin-bottom: 15px;
            font-style: italic;
        }

        .card-meta {
            font-size: 14px;
            color: #475569;
            margin-top: 15px;
            padding-top: 15px;
            border-top: 1px solid #e2e8f0;
        }

        .view-count {
            font-weight: 700;
            color: #008043;
            font-size: 16px;
        }

        /* API Info Section */
        .api-info {
            background: white;
            padding: 40px;
            border-radius: 20px;
            margin-bottom: 40px;
            box-shadow: 0 15px 50px rgba(0,0,0,0.3);
        }

        .api-info h2 {
            color: #1e293b;
            font-size: 28px;
            margin-bottom: 25px;
            padding-bottom: 15px;
            border-bottom: 3px solid #008043;
        }

        .endpoint-box {
            background: #f8fafc;
            padding: 20px;
            border-radius: 12px;
            margin: 15px 0;
            border-left: 5px solid #008043;
            font-family: 'Courier New', monospace;
            font-size: 15px;
            color: #475569;
            font-weight: 600;
        }

        .tech-stack {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            margin-top: 25px;
        }

        .tech-badge {
            background: linear-gradient(135deg, #008043, #00b359);
            color: white;
            padding: 12px 24px;
            border-radius: 30px;
            font-size: 14px;
            font-weight: 700;
            box-shadow: 0 5px 15px rgba(0,128,67,0.3);
        }

        /* Footer */
        footer {
            text-align: center;
            color: white;
            padding: 40px;
            margin-top: 50px;
        }

        footer p {
            margin: 12px 0;
            font-size: 16px;
            opacity: 0.95;
        }

        .highlight {
            color: #00ff88;
            font-weight: 700;
        }

        /* Loading Animation */
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }

        .loading {
            animation: pulse 1.5s ease-in-out infinite;
        }

        /* Responsive */
        @media (max-width: 768px) {
            h1 { font-size: 36px; }
            .dashboard-card { min-width: 300px; max-width: 300px; }
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Header -->
        <header>
            <h1>🎯 Tableau Insights Dashboard - POC</h1>
            <p class="subtitle">Real-time Dashboard Analytics from Tableau Database</p>
        </header>

        <!-- Status Banner -->
        <div class="status-banner" id="statusBanner">
            <span class="status-loading">⏳ Loading dashboard data from Insights API...</span>
        </div>

        <!-- Summary Statistics -->
        <div class="summary-grid" id="summaryGrid">
            <div class="summary-card">
                <div class="icon">📊</div>
                <div class="label">Total Dashboards</div>
                <div class="value loading" id="totalDashboards">--</div>
            </div>
            <div class="summary-card">
                <div class="icon">📁</div>
                <div class="label">Projects</div>
                <div class="value loading" id="totalProjects">--</div>
            </div>
            <div class="summary-card">
                <div class="icon">👁️</div>
                <div class="label">Total Views (30d)</div>
                <div class="value loading" id="totalViewCount">--</div>
            </div>
            <div class="summary-card">
                <div class="icon">🏢</div>
                <div class="label">Sites</div>
                <div class="value loading" id="totalSites">--</div>
            </div>
        </div>

        <!-- Dashboard Cards Section -->
        <h2 class="section-title">📈 Available Dashboards (Netflix-Style Scroll)</h2>
        <div class="scroll-container" id="dashboardContainer">
            <!-- Dashboard cards will be inserted here by JavaScript -->
            <div class="dashboard-card loading" style="min-width: 350px; height: 200px; display: flex; align-items: center; justify-content: center; color: #64748b;">
                Loading dashboards...
            </div>
        </div>

        <!-- API Information -->
        <div class="api-info">
            <h2>✅ Insights API Endpoint Information</h2>
            <p style="margin-bottom: 20px; font-size: 16px; color: #475569;">
                This POC uses the <strong>Insights Explore API</strong> to fetch dashboard data with complex joins across multiple tables.
            </p>
            
            <div class="endpoint-box">
                GET http://127.0.0.1:8000/api/tableau/insights/explore/?limit=20
            </div>

            <h3 style="margin-top: 35px; margin-bottom: 20px; color: #1e293b; font-size: 22px;">
                📊 Data Includes (Multi-Table Joins):
            </h3>
            <ul style="margin-left: 25px; color: #475569; line-height: 2; font-size: 15px;">
                <li><strong>View Info:</strong> Dashboard name, title, description, type</li>
                <li><strong>Workbook Info:</strong> Parent workbook name, owner details</li>
                <li><strong>Project Info:</strong> Project name (used as tag/category)</li>
                <li><strong>Site Info:</strong> Site name and configuration</li>
                <li><strong>View Counts:</strong> HTTP request analytics (last 30 days)</li>
                <li><strong>Timestamps:</strong> Created, updated, last refresh calculations</li>
            </ul>

            <h3 style="margin-top: 35px; margin-bottom: 20px; color: #1e293b; font-size: 22px;">
                🔧 Other Available Insights Endpoints:
            </h3>

            <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 15px;">
                <div class="endpoint-box">/api/tableau/insights/top_views/</div>
                <div class="endpoint-box">/api/tableau/insights/recent_dashboards/</div>
                <div class="endpoint-box">/api/tableau/insights/by_project/</div>
            </div>

            <h3 style="margin-top: 35px; margin-bottom: 20px; color: #1e293b; font-size: 22px;">
                💻 Technology Stack:
            </h3>
            <div class="tech-stack">
                <div class="tech-badge">Django 5.0</div>
                <div class="tech-badge">Django REST Framework</div>
                <div class="tech-badge">PostgreSQL</div>
                <div class="tech-badge">Complex ORM Joins</div>
                <div class="tech-badge">RESTful API</div>
                <div class="tech-badge">7 Tables Mapped</div>
                <div class="tech-badge">Real-time Analytics</div>
            </div>
        </div>

        <!-- Footer -->
        <footer>
            <p><strong>✅ Proof of Concept: Insights API with Multi-Table Joins</strong></p>
            <p>API Endpoint: <span class="highlight">/api/tableau/insights/explore/</span></p>
            <p>Database Joins: <span class="highlight">Views → Workbooks → Projects → Sites → Users</span></p>
            <p>Analytics: <span class="highlight">HTTP Requests (View Counts Last 30 Days)</span></p>
            <p style="margin-top: 25px; font-size: 14px; opacity: 0.85;">
                🏦 Developed by Micro @ BNP Paribas | November 2024
            </p>
            <p style="font-size: 13px; opacity: 0.75; margin-top: 10px;">
                Ready for Next.js integration with exact data structure for Insights page
            </p>
        </footer>
    </div>

    <script>
        // Fetch data from Insights Explore API
        fetch('http://127.0.0.1:8000/api/tableau/insights/explore/?limit=20')
            .then(response => {
                if (!response.ok) {
                    throw new Error(`HTTP error! status: ${response.status}`);
                }
                return response.json();
            })
            .then(data => {
                console.log('✅ Insights data loaded:', data);

                // Calculate summary statistics
                const dashboards = data.results || [];
                const uniqueProjects = new Set(dashboards.map(d => d.project_name));
                const uniqueSites = new Set(dashboards.map(d => d.site_name));
                const totalViews = dashboards.reduce((sum, d) => sum + (d.view_count_30_days || 0), 0);

                // Update summary cards
                document.getElementById('totalDashboards').textContent = dashboards.length;
                document.getElementById('totalProjects').textContent = uniqueProjects.size;
                document.getElementById('totalSites').textContent = uniqueSites.size;
                document.getElementById('totalViewCount').textContent = totalViews.toLocaleString();

                // Remove loading animation from summary
                document.querySelectorAll('.summary-card .value').forEach(el => {
                    el.classList.remove('loading');
                });

                // Create dashboard cards
                const container = document.getElementById('dashboardContainer');
                container.innerHTML = ''; // Clear loading message

                if (dashboards.length === 0) {
                    container.innerHTML = '<div style="color: white; text-align: center; width: 100%;">No dashboards found.</div>';
                } else {
                    dashboards.forEach(dashboard => {
                        const card = document.createElement('div');
                        card.className = 'dashboard-card';
                        
                        card.innerHTML = `
                            <div class="card-tag">${dashboard.project_name || 'General'}</div>
                            <div class="card-title">${dashboard.view_title || dashboard.view_name}</div>
                            <div class="card-subtitle">Last refresh: ${dashboard.last_refresh || 'N/A'}</div>
                            <div class="card-meta">
                                <div class="view-count">👁️ ${(dashboard.view_count_30_days || 0).toLocaleString()} views (30d)</div>
                                <div style="margin-top: 8px; font-size: 13px;">
                                    Owner: User ${dashboard.owner_system_user_id || 'Unknown'}
                                </div>
                                <div style="margin-top: 5px; font-size: 12px; color: #94a3b8;">
                                    Workbook: ${dashboard.workbook_name || 'N/A'}
                                </div>
                                <div style="margin-top: 5px; font-size: 12px; color: #94a3b8;">
                                    Site: ${dashboard.site_name || 'N/A'}
                                </div>
                            </div>
                        `;
                        
                        container.appendChild(card);
                    });
                }

                // Update status banner
                document.getElementById('statusBanner').innerHTML = 
                    `<span class="status-success">✅ Successfully loaded ${dashboards.length} dashboards from Insights API | Multi-table joins working perfectly!</span>`;

            })
            .catch(error => {
                console.error('❌ Error fetching data:', error);
                
                // Update status banner with error
                document.getElementById('statusBanner').innerHTML = 
                    `<span class="status-error">❌ Connection Error: Make sure Django server is running on http://127.0.0.1:8000</span>`;

                // Show error in summary cards
                document.querySelectorAll('.summary-card .value').forEach(el => {
                    el.textContent = 'Error';
                    el.classList.remove('loading');
                    el.style.fontSize = '24px';
                    el.style.color = '#ef4444';
                });

                // Show error in dashboard container
                document.getElementById('dashboardContainer').innerHTML = 
                    '<div style="color: white; text-align: center; width: 100%; padding: 40px;">Failed to load dashboards. Check console for details.</div>';
            });
    </script>
</body>
</html>
```

---

## 🎯 WHAT'S DIFFERENT NOW:

### ✅ Uses Insights Explore API:
```
GET /api/tableau/insights/explore/?limit=20
