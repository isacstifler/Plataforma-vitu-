<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plataforma de Lançamento e Análise</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutrals with Subtle Accents -->
    <!-- Application Structure Plan: Single-page app with tabs for 'Lançamento', 'Painel de Análise', and 'Reembolsos'. Data persists to localStorage. 'Lançamento' form includes 'Plataforma' and optional 'Nome do Cliente'. 'Painel de Análise' features a dynamic monthly calendar, period AND simplified platform selectors; its key metrics reflect net values considering refunds and selected filters. Gemini API for strategic advice now accepts specific user questions. 'Lançamentos Recentes' order IDs are clickable. 'Todos os Lançamentos' modal includes 'Plataforma', 'Nome do Cliente', and edit button, with an improved layout for filters (including new simplified platform filter buttons) and table (zebra-striping, sticky header). 'Reembolsos' tab for logging/viewing refunds also includes 'Nome do Cliente' and a new Gemini API feature to analyze refund patterns. Gemini API for description generation remains. -->
    <!-- Visualization & Content Choices: Forms for data entry. Dashboard: Period/Platform selectors, stat cards, tables, Chart.js bar chart, dynamic calendar. Modals for full log, editing, AI (strategic and refund analysis). Reembolso section. Platform and Customer Name info added. Key dashboard metrics reflect refund impact and selected filters. Data source is localStorage. Calendar uses different background colors/indicators. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8f7f4; }
        .tab-button { transition: all 0.3s ease; }
        .tab-button.active { border-color: #4f46e5; color: #4f46e5; background-color: #eef2ff; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .stat-card { background-color: white; border-radius: 0.75rem; padding: 1.5rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1); }
        .zona-alta { background-color: #dcfce7; color: #166534; }
        .zona-media { background-color: #fef9c3; color: #854d0e; }
        .zona-baixa { background-color: #fee2e2; color: #991b1b; }
        .tendencia-estavel { background-color: #dcfce7; color: #166534; }
        .tendencia-moderada { background-color: #fef3c7; color: #b45309; }
        .tendencia-altavariacao { background-color: #ffe4e6; color: #a21caf; }
        .tendencia-neutra { background-color: #e5e7eb; color: #4b5563;}
        .chart-container { position: relative; width: 100%; max-width: 600px; margin-left: auto; margin-right: auto; height: 300px; max-height: 400px; }
        #toast-message { position: fixed; top: 1.5rem; right: 1.5rem; background-color: #22c55e; color: white; padding: 1rem 1.5rem; border-radius: 0.5rem; box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1); transform: translateX(120%); transition: transform 0.5s ease-in-out; z-index: 100; }
        #toast-message.show { transform: translateX(0); }
        .modal-overlay { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background-color: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center; z-index: 50; opacity: 0; visibility: hidden; transition: opacity 0.3s ease, visibility 0.3s ease;}
        .modal-overlay.active { opacity: 1; visibility: visible; }
        .modal-content { background-color: white; padding: 2rem; border-radius: 0.75rem; box-shadow: 0 10px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1); width: 90%; max-width: 6xl; max-height: 90vh; display: flex; flex-direction: column; }
        .modal-filters-section { background-color: #f9fafb; padding: 1rem; border-radius: 0.5rem; margin-bottom: 1.5rem; border: 1px solid #e5e7eb;}
        .modal-table-container { flex-grow: 1; overflow-y: auto; max-height: calc(90vh - 250px); }
        .table-scroll-container { max-height: 300px; overflow-y: auto; }
        .gemini-spinner, .description-spinner { border: 3px solid rgba(0, 0, 0, 0.1); width: 20px; height: 20px; border-radius: 50%; border-left-color: #4f46e5; animation: spin 1s ease infinite; display: inline-block; vertical-align: middle; margin-left: 8px; }
        .gemini-spinner {width: 36px; height: 36px; margin: 20px auto;}
        @keyframes spin { to { transform: rotate(360deg); } }
        .status-select { padding: 0.25rem 0.5rem; border-radius: 0.375rem; border: 1px solid #d1d5db; background-color: white; min-width: 120px;}
        .input-error { border-color: #ef4444 !important; box-shadow: 0 0 0 2px rgba(239, 68, 68, 0.2) !important; }
        #form-lancamento button[type="submit"]:disabled, #formReembolso button[type="submit"]:disabled { opacity: 0.6; cursor: not-allowed; }
        .edit-btn-icon { display: inline-block; width: 1.25rem; height: 1.25rem; vertical-align: middle; }
        .period-button, .platform-filter-button, .platform-modal-filter-button { background-color: #e5e7eb; color: #374151; padding: 0.5rem 1rem; border-radius: 0.5rem; font-weight: 500; transition: background-color 0.2s ease-in-out; border: 1px solid transparent;}
        .period-button.active, .platform-filter-button.active, .platform-modal-filter-button.active { background-color: #4f46e5; color: white; border-color: #4338ca;}
        .period-button:hover:not(.active), .platform-filter-button:hover:not(.active), .platform-modal-filter-button:hover:not(.active) { background-color: #d1d5db; }
        .clickable-order { cursor: pointer; color: #4f46e5; text-decoration: underline; }
        .clickable-order:hover { color: #3730a3; }
        #tabelaModalTodosLancamentos tr:nth-child(even) { background-color: #f9fafb; }
        #tabelaModalTodosLancamentos th { background-color: #f3f4f6; position: sticky; top: 0; z-index: 10; }
        #tabelaReembolsos tr:nth-child(even) { background-color: #f9fafb; }
        #tabelaReembolsos th { background-color: #f3f4f6; position: sticky; top: 0; z-index: 10; }

        /* Calendar Styles */
        .calendar-container { background-color: white; padding: 1.5rem; border-radius: 0.75rem; box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1); margin-top: 1.5rem; }
        .calendar-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem; }
        .calendar-grid { display: grid; grid-template-columns: repeat(7, minmax(0, 1fr)); gap: 1px; background-color: #e5e7eb; border: 1px solid #e5e7eb; }
        .calendar-day-name { font-weight: 600; text-align: center; padding: 0.5rem 0.25rem; font-size: 0.75rem; background-color: #f9fafb; color: #4b5563; }
        .calendar-day { background-color: white; padding: 0.5rem; text-align: center; font-size: 0.875rem; min-height: 70px; position: relative; cursor: default; }
        .calendar-day.weekday { background-color: #e0f2fe; /* Light Sky Blue for weekdays */ }
        .calendar-day.weekend { background-color: #f3f4f6; /* Lighter Gray for weekends */ }
        .calendar-day.other-month { background-color: #e5e7eb; color: #9ca3af; }
        .calendar-day .day-number { font-weight: 500; }
        .calendar-day.today .day-number { background-color: #4f46e5; color: white; border-radius: 50%; width: 1.75rem; height: 1.75rem; line-height: 1.75rem; display: inline-block; margin: 0 auto; }
        .calendar-day.holiday { background-color: #fef08a !important; /* Light Yellow for holidays */ }
        .calendar-day.event-campanha { border-bottom: 3px solid #60a5fa; /* Blue border for campaigns */ }
        .calendar-day.event-pico_vendas { border-bottom: 3px solid #f59e0b; /* Amber border for peak sales */ }
        .calendar-day .event-marker { font-size: 0.65rem; display: block; margin-top: 0.25rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; padding: 0.1rem 0.25rem; border-radius: 0.25rem; color: #374151;}
        .calendar-day.holiday .event-marker { color: #7f1d1d; font-weight: 500;}
        .calendar-day .tooltip { visibility: hidden; width: max-content; max-width: 150px; background-color: #333; color: #fff; text-align: center; border-radius: 6px; padding: 5px 8px; position: absolute; z-index: 10; bottom: 125%; left: 50%; transform: translateX(-50%); opacity: 0; transition: opacity 0.3s; font-size: 0.75rem; }
        .calendar-day:hover .tooltip { visibility: visible; opacity: 1; }
        .calendar-legend { margin-top: 1rem; display: flex; flex-wrap: wrap; gap: 1rem; font-size: 0.75rem;}
        .legend-item { display: flex; align-items: center; }
        .legend-color-box { width: 1rem; height: 1rem; margin-right: 0.5rem; border-radius: 0.125rem; border: 1px solid #ccc; }
    </style>
</head>
<body class="text-gray-800">

    <div id="toast-message"></div>

    <div class="container mx-auto p-4 md:p-8">
        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-900">Plataforma de Lançamento e Análise</h1>
            <p class="text-gray-600 mt-2">Insira os dados da transação e veja a análise de desempenho ser atualizada em tempo real.</p>
        </header>

        <div class="mb-8 border-b border-gray-200">
            <nav class="flex space-x-2" aria-label="Tabs">
                <button id="tab-lancamento" class="tab-button active text-sm font-medium py-3 px-4 rounded-t-lg border-b-2 border-transparent hover:border-gray-300 hover:text-gray-600">
                    🚀 Lançamento
                </button>
                <button id="tab-painel" class="tab-button text-sm font-medium py-3 px-4 rounded-t-lg border-b-2 border-transparent hover:border-gray-300 hover:text-gray-600">
                    📊 Painel de Análise
                </button>
                <button id="tab-reembolso" class="tab-button text-sm font-medium py-3 px-4 rounded-t-lg border-b-2 border-transparent hover:border-gray-300 hover:text-gray-600">
                    🔄 Reembolsos
                </button>
            </nav>
        </div>

        <main>
            <div id="lancamento-page" class="tab-content active">
                <div class="max-w-4xl mx-auto bg-white p-6 md:p-8 rounded-xl shadow-lg">
                    <h2 class="text-2xl font-semibold mb-6 text-gray-800">Nova Transação</h2>
                    <form id="form-lancamento" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label for="data" class="block text-sm font-medium text-gray-700 mb-1">Data</label>
                            <input type="date" id="data" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="ordemBM" class="block text-sm font-medium text-gray-700 mb-1">Ordem (BM)</label>
                            <input type="text" id="ordemBM" placeholder="Ex: 11C2DFD793" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                         <div>
                            <label for="nomeCliente" class="block text-sm font-medium text-gray-700 mb-1">Nome do Cliente</label>
                            <input type="text" id="nomeCliente" placeholder="Nome do cliente (opcional)" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="plataforma" class="block text-sm font-medium text-gray-700 mb-1">Plataforma</label>
                            <select id="plataforma" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition bg-white">
                                <option value="ALIPAY">ALIPAY</option>
                                <option value="CSSBUY">CSSBUY</option>
                                <option value="Página4">Página4 (Exemplo)</option>
                                <option value="Outra">Outra</option>
                            </select>
                        </div>
                        <div>
                            <label for="statusOrdem" class="block text-sm font-medium text-gray-700 mb-1">Status da Ordem</label>
                            <select id="statusOrdem" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition bg-white">
                                <option value="Concluída">Concluída</option>
                                <option value="Pendente">Pendente</option>
                                <option value="Cancelada">Cancelada</option>
                            </select>
                        </div>
                        <div>
                            <label for="volumeCNY" class="block text-sm font-medium text-gray-700 mb-1">Volume (CNY)</label>
                            <input type="number" id="volumeCNY" placeholder="Ex: 1000.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="valorVenda" class="block text-sm font-medium text-gray-700 mb-1">Valor de Venda (BRL)</label>
                            <input type="number" id="valorVenda" placeholder="Ex: 442.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="valorCusto" class="block text-sm font-medium text-gray-700 mb-1">Valor de Custo (BRL)</label>
                            <input type="number" id="valorCusto" placeholder="Ex: 380.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                         <div class="md:col-span-2">
                            <label for="descricaoOrdem" class="block text-sm font-medium text-gray-700 mb-1">
                                Descrição da Ordem
                                <button type="button" id="btnGerarDescricaoIA" title="Gerar descrição com IA" class="ml-2 text-purple-600 hover:text-purple-800">
                                    ✨<span id="descricaoIASpinner" class="description-spinner hidden"></span>
                                </button>
                            </label>
                            <textarea id="descricaoOrdem" rows="3" placeholder="Detalhes adicionais sobre a ordem..." class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition"></textarea>
                        </div>
                        <div class="md:col-span-2 text-right mt-4">
                            <button type="submit" id="btnLancar" class="bg-indigo-600 text-white font-semibold py-3 px-8 rounded-lg hover:bg-indigo-700 focus:outline-none focus:ring-4 focus:ring-indigo-300 transition-all duration-300 transform hover:scale-105">
                                SALVAR ORDEM
                            </button>
                        </div>
                    </form>
                </div>
            </div>

            <div id="painel-page" class="tab-content">
                <div class="mb-6 bg-white p-4 rounded-xl shadow">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div>
                            <h3 class="text-lg font-semibold text-gray-700 mb-3">Selecionar Período da Análise:</h3>
                            <div id="periodo-selector" class="flex flex-wrap gap-2">
                                <button data-periodo="hoje" class="period-button">Hoje</button>
                                <button data-periodo="7dias" class="period-button">Últimos 7 dias</button>
                                <button data-periodo="30dias" class="period-button">Últimos 30 dias</button>
                                <button data-periodo="todos" class="period-button active">Desde o Início</button>
                            </div>
                        </div>
                        <div>
                            <h3 class="text-lg font-semibold text-gray-700 mb-3">Filtrar por Plataforma (Painel):</h3>
                            <div id="plataforma-filter-selector" class="flex flex-wrap gap-2">
                                <button data-plataforma="todas" class="platform-filter-button active">Todas</button>
                                <button data-plataforma="ALIPAY" class="platform-filter-button">ALIPAY</button>
                                <button data-plataforma="CSSBUY" class="platform-filter-button">CSSBUY</button>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <div class="lg:col-span-3 grid grid-cols-2 md:grid-cols-4 gap-6">
                        <div class="stat-card text-center">
                            <h3 class="text-sm font-medium text-gray-500">Lucro no Período (BRL)</h3>
                            <p id="lucroTotalBRL" class="text-3xl font-bold text-green-600 mt-2">R$ 0,00</p>
                        </div>
                        <div class="stat-card text-center">
                            <h3 class="text-sm font-medium text-gray-500">Margem no Período</h3>
                            <p id="margemGeral" class="text-3xl font-bold text-indigo-600 mt-2">0,00%</p>
                        </div>
                        <div class="stat-card text-center">
                            <h3 class="text-sm font-medium text-gray-500">Vendas no Período</h3>
                            <p id="totalVendas" class="text-3xl font-bold text-gray-800 mt-2">0</p>
                        </div>
                        <div class="stat-card text-center">
                            <h3 class="text-sm font-medium text-gray-500">Zona de Ação (Período)</h3>
                            <p id="zonaAcaoGeral" class="text-xl font-bold mt-2 py-2 px-4 rounded-full inline-block">N/A</p>
                        </div>
                    </div>

                    <div class="lg:col-span-2 stat-card">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold">Lançamentos Recentes (Geral - Últimos 10)</h3>
                            <button id="btnVerTodos" class="text-sm text-indigo-600 hover:text-indigo-800 font-medium">Ver Todos &rarr;</button>
                        </div>
                        <div class="table-scroll-container">
                            <table class="min-w-full divide-y divide-gray-200">
                                <thead class="bg-gray-50 sticky top-0">
                                    <tr>
                                        <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Data</th>
                                        <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ordem</th>
                                        <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Lucro (BRL)</th>
                                        <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Margem</th>
                                        <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Zona</th>
                                    </tr>
                                </thead>
                                <tbody id="tabelaLancamentosCorpo" class="bg-white divide-y divide-gray-200">
                                    <tr><td colspan="5" class="px-4 py-4 text-center text-gray-500">Nenhum lançamento ainda.</td></tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    
                    <div class="stat-card">
                        <h3 class="text-lg font-semibold mb-4">Métricas de Rendimento (Período)</h3>
                        <div class="space-y-4">
                            <div class="flex justify-between items-center">
                                <span class="text-sm font-medium text-gray-600">Total (CNY)</span>
                                <span id="totalCNY" class="text-base font-semibold">¥ 0,00</span>
                            </div>
                            <div class="flex justify-between items-center">
                                <span class="text-sm font-medium text-gray-600">Total Vendas (BRL)</span>
                                <span id="totalVendasBRL" class="text-base font-semibold">R$ 0,00</span>
                            </div>
                            <hr>
                            <div class="flex justify-between items-center">
                                <span class="text-sm font-medium text-gray-600">Ticket Médio (BRL)</span>
                                <span id="ticketMedioBRL" class="text-base font-semibold">R$ 0,00</span>
                            </div>
                            <div class="flex justify-between items-center">
                                <span class="text-sm font-medium text-gray-600">Lucro Médio (BRL)</span>
                                <span id="lucroMedio" class="text-base font-semibold">R$ 0,00</span>
                            </div>
                        </div>
                    </div>

                    <div class="lg:col-span-3 stat-card">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-semibold">Zonas de Ação e Estratégia (Período)</h3>
                            <button id="btnAnalisarEstrategiaIA" class="bg-purple-600 text-white text-sm font-semibold py-2 px-4 rounded-lg hover:bg-purple-700 focus:outline-none focus:ring-4 focus:ring-purple-300 transition-all">
                                ✨ Analisar Estratégia com IA
                            </button>
                        </div>
                        <div class="space-y-3">
                            <div class="grid grid-cols-12 gap-4 items-center p-3 rounded-lg zona-alta">
                                <div class="col-span-12 sm:col-span-2 font-bold">ALTA</div>
                                <div class="col-span-6 sm:col-span-2"><span id="zonaAltaOcorrencias">0</span> ocorrências</div>
                                <div class="col-span-6 sm:col-span-2 font-semibold">R$ <span id="zonaAltaVolume">0,00</span></div>
                                <div class="col-span-12 sm:col-span-6 text-sm">Escalar vendas, manter preço, rodar mais volume.</div>
                            </div>
                            <div class="grid grid-cols-12 gap-4 items-center p-3 rounded-lg zona-media">
                                <div class="col-span-12 sm:col-span-2 font-bold">MÉDIA</div>
                                <div class="col-span-6 sm:col-span-2"><span id="zonaMediaOcorrencias">0</span> ocorrências</div>
                                <div class="col-span-6 sm:col-span-2 font-semibold">R$ <span id="zonaMediaVolume">0,00</span></div>
                                <div class="col-span-12 sm:col-span-6 text-sm">Monitorar de perto, ajustar preço com cautela.</div>
                            </div>
                            <div class="grid grid-cols-12 gap-4 items-center p-3 rounded-lg zona-baixa">
                                <div class="col-span-12 sm:col-span-2 font-bold">BAIXA</div>
                                <div class="col-span-6 sm:col-span-2"><span id="zonaBaixaOcorrencias">0</span> ocorrências</div>
                                <div class="col-span-6 sm:col-span-2 font-semibold">R$ <span id="zonaBaixaVolume">0,00</span></div>
                                <div class="col-span-12 sm:col-span-6 text-sm">Reagir: ou baixa o preço para girar mais ou pausa o volume.</div>
                            </div>
                        </div>
                    </div>
                    
                    <div class="lg:col-span-3 stat-card">
                        <h3 class="text-lg font-semibold mb-4">Análise de Tendência de Margem (Período)</h3>
                        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 items-center">
                             <div class="md:col-span-1 text-center">
                                <p class="text-sm font-medium text-gray-500">Desvio Padrão (Margem)</p>
                                <p id="desvioPadraoMargem" class="text-4xl font-bold text-purple-600 my-2">0,00%</p>
                                <p id="interpretacaoTendencia" class="text-md font-semibold p-2 rounded-full">N/A</p>
                             </div>
                             <div class="md:col-span-2">
                                <div class="chart-container">
                                    <canvas id="tendenciaChart"></canvas>
                                </div>
                             </div>
                        </div>
                    </div>
                     <div class="lg:col-span-3 calendar-container">
                        <div class="calendar-header">
                            <button id="prevMonthBtn" class="p-2 rounded-md hover:bg-gray-200">
                                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                    <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
                                </svg>
                            </button>
                            <h2 id="calendarMonthYear" class="text-xl font-semibold text-gray-800"></h2>
                            <button id="nextMonthBtn" class="p-2 rounded-md hover:bg-gray-200">
                                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                                    <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
                                </svg>
                            </button>
                        </div>
                         <div class="calendar-grid" id="calendarDayNames">
                            <!-- Nomes dos dias da semana -->
                        </div>
                        <div class="calendar-grid" id="calendarGrid">
                            <!-- Dias do calendário serão inseridos aqui pelo JS -->
                        </div>
                        <div class="calendar-legend mt-4">
                            <div class="legend-item"><span class="legend-color-box today"></span>Dia Atual</div>
                            <div class="legend-item"><span class="legend-color-box weekday" style="background-color: #e0f2fe;"></span>Dia Normal</div>
                            <div class="legend-item"><span class="legend-color-box weekend" style="background-color: #f3f4f6;"></span>Fim de Semana</div>
                            <div class="legend-item"><span class="legend-color-box holiday" style="background-color: #fef08a;"></span>Feriado</div>
                            <div class="legend-item"><span class="legend-color-box event-campanha" style="border-bottom: 3px solid #60a5fa;"></span>Campanha</div>
                            <div class="legend-item"><span class="legend-color-box event-pico_vendas" style="border-bottom: 3px solid #f59e0b;"></span>Pico Vendas</div>
                        </div>
                    </div>
                </div>
            </div>
            <div id="reembolso-page" class="tab-content">
                 <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8">
                    <div class="stat-card text-center">
                        <h3 class="text-sm font-medium text-gray-500">Total Reembolsado (BRL)</h3>
                        <p id="totalReembolsado" class="text-3xl font-bold text-red-600 mt-2">R$ 0,00</p>
                    </div>
                    <div class="stat-card text-center">
                        <h3 class="text-sm font-medium text-gray-500">Número de Reembolsos</h3>
                        <p id="numeroReembolsos" class="text-3xl font-bold text-gray-800 mt-2">0</p>
                    </div>
                 </div>
                  <div class="mb-6 flex justify-center">
                     <button id="btnAnalisarReembolsosIA" class="bg-pink-600 text-white text-sm font-semibold py-2 px-4 rounded-lg hover:bg-pink-700 focus:outline-none focus:ring-4 focus:ring-pink-300 transition-all">
                         ✨ Analisar Padrões de Reembolso com IA
                     </button>
                 </div>

                <div class="max-w-4xl mx-auto bg-white p-6 md:p-8 rounded-xl shadow-lg mb-8">
                    <h2 class="text-2xl font-semibold mb-6 text-gray-800">Registrar Novo Reembolso</h2>
                    <form id="formReembolso" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div>
                            <label for="dataReembolso" class="block text-sm font-medium text-gray-700 mb-1">Data do Reembolso</label>
                            <input type="date" id="dataReembolso" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="reembolsoNomeCliente" class="block text-sm font-medium text-gray-700 mb-1">Nome do Cliente</label>
                            <input type="text" id="reembolsoNomeCliente" placeholder="Nome do cliente (opcional)" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="clienteOrdemOriginal" class="block text-sm font-medium text-gray-700 mb-1">Ordem Original (Referência)</label>
                            <input type="text" id="clienteOrdemOriginal" placeholder="ID da ordem (opcional)" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="valorReembolso" class="block text-sm font-medium text-gray-700 mb-1">Valor do Reembolso (BRL)</label>
                            <input type="number" id="valorReembolso" placeholder="Ex: 84.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                        </div>
                        <div>
                            <label for="plataformaReembolso" class="block text-sm font-medium text-gray-700 mb-1">Plataforma do Reembolso</label>
                            <select id="plataformaReembolso" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition bg-white">
                                <option value="ALIPAY">ALIPAY</option>
                                <option value="CSSBUY">CSSBUY</option>
                                <option value="Outra">Outra</option>
                            </select>
                        </div>
                        <div class="md:col-span-2">
                            <label for="observacoesReembolso" class="block text-sm font-medium text-gray-700 mb-1">Observações</label>
                            <textarea id="observacoesReembolso" rows="3" placeholder="Motivo do reembolso, detalhes adicionais..." class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition"></textarea>
                        </div>
                        <div class="md:col-span-2 text-right mt-4">
                            <button type="submit" class="bg-red-600 text-white font-semibold py-3 px-8 rounded-lg hover:bg-red-700 focus:outline-none focus:ring-4 focus:ring-red-300 transition-all">
                                Adicionar Reembolso
                            </button>
                        </div>
                    </form>
                </div>

                <div class="bg-white p-6 md:p-8 rounded-xl shadow-lg">
                    <h2 class="text-2xl font-semibold mb-6 text-gray-800">Histórico de Reembolsos</h2>
                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-gray-200">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Data</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Cliente</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ordem Ref.</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Valor (BRL)</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Plataforma</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Observações</th>
                                    <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Valor Deduzido (BRL)</th>
                                </tr>
                            </thead>
                            <tbody id="tabelaReembolsos" class="bg-white divide-y divide-gray-200">
                                <tr><td colspan="7" class="px-4 py-4 text-center text-gray-500">Nenhum reembolso registrado.</td></tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <div id="modalVerTodos" class="modal-overlay">
        <div class="modal-content">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-2xl font-semibold text-gray-800">Todos os Lançamentos</h2>
                <button id="btnFecharModalVerTodos" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div class="modal-filters-section">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                    <div>
                        <label for="filtroDataInicio" class="block text-sm font-medium text-gray-700 mb-1">Data Início</label>
                        <input type="date" id="filtroDataInicio" class="w-full p-2 border border-gray-300 rounded-lg">
                    </div>
                    <div>
                        <label for="filtroDataFim" class="block text-sm font-medium text-gray-700 mb-1">Data Fim</label>
                        <input type="date" id="filtroDataFim" class="w-full p-2 border border-gray-300 rounded-lg">
                    </div>
                </div>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-1">Plataforma</label>
                    <div id="modalPlataformaFilter" class="flex flex-wrap gap-2">
                        <button data-plataforma="todas" class="platform-modal-filter-button active">Todas</button>
                        <button data-plataforma="ALIPAY" class="platform-modal-filter-button">ALIPAY</button>
                        <button data-plataforma="CSSBUY" class="platform-modal-filter-button">CSSBUY</button>
                    </div>
                </div>
                <button id="btnAplicarFiltro" class="w-full bg-indigo-600 text-white py-2.5 px-4 rounded-lg hover:bg-indigo-700 text-sm">Aplicar Filtros</button>
            </div>
            <div class="modal-table-container">
                <table class="min-w-full divide-y divide-gray-200">
                    <thead class="bg-gray-100">
                        <tr>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Data</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ordem</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Cliente</th>
                             <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Plataforma</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Status</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Volume (CNY)</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Venda (BRL)</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Custo (BRL)</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Lucro (BRL)</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Margem</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Zona</th>
                            <th class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="tabelaModalTodosLancamentos" class="bg-white divide-y divide-gray-200"></tbody>
                </table>
            </div>
        </div>
    </div>

    <div id="modalAnaliseIA" class="modal-overlay">
        <div class="modal-content max-w-2xl">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-semibold text-gray-800">✨ Análise Estratégica da IA</h2>
                <button id="btnFecharModalIA" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div class="mb-4">
                <label for="perguntaEspecificaIA" class="block text-sm font-medium text-gray-700 mb-1">Faça uma pergunta específica sobre seus dados (opcional):</label>
                <textarea id="perguntaEspecificaIA" rows="2" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500" placeholder="Ex: Qual o impacto da plataforma X na minha margem de lucro?"></textarea>
            </div>
            <div id="conteudoAnaliseIA" class="text-gray-700 space-y-4">
                <div class="gemini-spinner"></div>
                <p class="text-center text-gray-500">Aguarde, analisando sua estratégia...</p>
            </div>
        </div>
    </div>
    
    <div id="modalAnaliseReembolsoIA" class="modal-overlay">
        <div class="modal-content max-w-2xl">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-semibold text-gray-800">✨ Análise de Padrões de Reembolso</h2>
                <button id="btnFecharModalReembolsoIA" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div id="conteudoAnaliseReembolsoIA" class="text-gray-700 space-y-4">
                <div class="gemini-spinner"></div>
                <p class="text-center text-gray-500">Aguarde, analisando padrões de reembolso...</p>
            </div>
        </div>
    </div>


    <div id="modalEditarLancamento" class="modal-overlay">
        <div class="modal-content max-w-2xl">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-semibold text-gray-800">Editar Lançamento</h2>
                <button id="btnFecharModalEditar" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <form id="formEditarLancamento" class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <input type="hidden" id="editTransacaoId">
                <div>
                    <label for="editData" class="block text-sm font-medium text-gray-700 mb-1">Data</label>
                    <input type="date" id="editData" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                </div>
                <div>
                    <label for="editOrdemBM" class="block text-sm font-medium text-gray-700 mb-1">Ordem (BM)</label>
                    <input type="text" id="editOrdemBM" placeholder="Ex: 11C2DFD793" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                </div>
                 <div>
                    <label for="editNomeCliente" class="block text-sm font-medium text-gray-700 mb-1">Nome do Cliente</label>
                    <input type="text" id="editNomeCliente" placeholder="Nome do cliente (opcional)" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                </div>
                 <div>
                    <label for="editPlataforma" class="block text-sm font-medium text-gray-700 mb-1">Plataforma</label>
                    <select id="editPlataforma" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition bg-white">
                        <option value="ALIPAY">ALIPAY</option>
                        <option value="CSSBUY">CSSBUY</option>
                        <option value="Página4">Página4 (Exemplo)</option>
                        <option value="Outra">Outra</option>
                    </select>
                </div>
                <div>
                    <label for="editStatusOrdem" class="block text-sm font-medium text-gray-700 mb-1">Status da Ordem</label>
                    <select id="editStatusOrdem" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition bg-white">
                        <option value="Concluída">Concluída</option>
                        <option value="Pendente">Pendente</option>
                        <option value="Cancelada">Cancelada</option>
                    </select>
                </div>
                <div>
                    <label for="editVolumeCNY" class="block text-sm font-medium text-gray-700 mb-1">Volume (CNY)</label>
                    <input type="number" id="editVolumeCNY" placeholder="Ex: 1000.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                </div>
                <div>
                    <label for="editValorVenda" class="block text-sm font-medium text-gray-700 mb-1">Valor de Venda (BRL)</label>
                    <input type="number" id="editValorVenda" placeholder="Ex: 442.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                </div>
                <div>
                    <label for="editValorCusto" class="block text-sm font-medium text-gray-700 mb-1">Valor de Custo (BRL)</label>
                    <input type="number" id="editValorCusto" placeholder="Ex: 380.00" required step="0.01" class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition">
                </div>
                 <div class="md:col-span-2">
                    <label for="editDescricaoOrdem" class="block text-sm font-medium text-gray-700 mb-1">Descrição da Ordem</label>
                    <textarea id="editDescricaoOrdem" rows="3" placeholder="Detalhes adicionais sobre a ordem..." class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition"></textarea>
                </div>
                <div class="md:col-span-2 text-right mt-4">
                    <button type="submit" id="btnSalvarEdicao" class="bg-green-600 text-white font-semibold py-3 px-8 rounded-lg hover:bg-green-700 focus:outline-none focus:ring-4 focus:ring-green-300 transition-all">
                        Salvar Alterações
                    </button>
                </div>
            </form>
        </div>
    </div>

    <script type="module">
        const LIMITE_ALTA_MARGEM = 15;
        const LIMITE_MEDIA_MARGEM = 5;
        const LIMITE_TENDENCIA_ESTAVEL = 2.0;
        const LIMITE_TENDENCIA_MODERADA = 5.0;

        let transacoes = [];
        let reembolsos = [];
        let tendenciaChartInstance = null;
        let periodoSelecionado = 'todos'; 
        let plataformaSelecionada = 'todas';
        let plataformaModalSelecionada = 'todas';
        
        let currentCalendarDate = new Date();


        const tabLancamento = document.getElementById('tab-lancamento');
        const tabPainel = document.getElementById('tab-painel');
        const tabReembolso = document.getElementById('tab-reembolso');
        const pageLancamento = document.getElementById('lancamento-page');
        const pagePainel = document.getElementById('painel-page');
        const pageReembolso = document.getElementById('reembolso-page');

        const formLancamento = document.getElementById('form-lancamento');
        const btnLancar = document.getElementById('btnLancar');
        const toastMessage = document.getElementById('toast-message');
        
        const modalVerTodos = document.getElementById('modalVerTodos');
        const btnVerTodos = document.getElementById('btnVerTodos');
        const btnFecharModalVerTodos = document.getElementById('btnFecharModalVerTodos');
        const btnAplicarFiltro = document.getElementById('btnAplicarFiltro');
        const filtroDataInicio = document.getElementById('filtroDataInicio');
        const filtroDataFim = document.getElementById('filtroDataFim');
        const tabelaModalTodosLancamentos = document.getElementById('tabelaModalTodosLancamentos');
        const dataInput = document.getElementById('data');
        const nomeClienteInput = document.getElementById('nomeCliente');
        const descricaoOrdemInput = document.getElementById('descricaoOrdem');
        const btnGerarDescricaoIA = document.getElementById('btnGerarDescricaoIA');
        const descricaoIASpinner = document.getElementById('descricaoIASpinner');
        const periodoSelectorContainer = document.getElementById('periodo-selector');
        const plataformaFilterSelectorContainer = document.getElementById('plataforma-filter-selector');
        const modalPlataformaFilterContainer = document.getElementById('modalPlataformaFilter');


        const modalAnaliseIA = document.getElementById('modalAnaliseIA');
        const btnAnalisarEstrategiaIA = document.getElementById('btnAnalisarEstrategiaIA');
        const btnFecharModalIA = document.getElementById('btnFecharModalIA');
        const conteudoAnaliseIA = document.getElementById('conteudoAnaliseIA');
        const perguntaEspecificaIAInput = document.getElementById('perguntaEspecificaIA');

        const modalAnaliseReembolsoIA = document.getElementById('modalAnaliseReembolsoIA');
        const btnAnalisarReembolsosIA = document.getElementById('btnAnalisarReembolsosIA'); // Precisa ser adicionado ao HTML
        const btnFecharModalReembolsoIA = document.getElementById('btnFecharModalReembolsoIA'); // Precisa ser adicionado ao HTML
        const conteudoAnaliseReembolsoIA = document.getElementById('conteudoAnaliseReembolsoIA');


        const modalEditarLancamento = document.getElementById('modalEditarLancamento');
        const btnFecharModalEditar = document.getElementById('btnFecharModalEditar');
        const formEditarLancamento = document.getElementById('formEditarLancamento');

        const formReembolso = document.getElementById('formReembolso');
        const tabelaReembolsos = document.getElementById('tabelaReembolsos');
        const totalReembolsadoEl = document.getElementById('totalReembolsado');
        const numeroReembolsosEl = document.getElementById('numeroReembolsos');

        const calendarGrid = document.getElementById('calendarGrid');
        const calendarMonthYear = document.getElementById('calendarMonthYear');
        const prevMonthBtn = document.getElementById('prevMonthBtn');
        const nextMonthBtn = document.getElementById('nextMonthBtn');
        const calendarDayNamesContainer = document.getElementById('calendarDayNames');

        const feriadosNacionais2025 = [ 
            { date: '2025-01-01', name: 'Confraternização Universal' },
            { date: '2025-03-03', name: 'Carnaval (Ponto Facultativo)' },
            { date: '2025-03-04', name: 'Carnaval' },
            { date: '2025-03-05', name: 'Quarta-feira de Cinzas (Ponto Facultativo até 14h)' },
            { date: '2025-04-18', name: 'Paixão de Cristo' },
            { date: '2025-04-21', name: 'Tiradentes' },
            { date: '2025-05-01', name: 'Dia do Trabalho' },
            { date: '2025-06-19', name: 'Corpus Christi (Ponto Facultativo)' },
            { date: '2025-09-07', name: 'Independência do Brasil' },
            { date: '2025-10-12', name: 'Nossa Senhora Aparecida' },
            { date: '2025-11-02', name: 'Finados' },
            { date: '2025-11-15', name: 'Proclamação da República' },
            { date: '2025-12-25', name: 'Natal' }
        ];

        const eventosInternos = [
            { date: '2025-06-15', type: 'campanha', description: 'Início Campanha Dia dos Pais' },
            { date: '2025-06-20', type: 'pico_vendas', description: 'Pico de Vendas - Promoção X' },
        ];


        const parseValorMonetario = (valorString) => {
            if (typeof valorString !== 'string' && typeof valorString !== 'number') return 0;
            if (typeof valorString === 'number') return valorString;
            return parseFloat(String(valorString).replace('R$', '').replace('¥', '').replace(/\./g, '').replace(',', '.').trim()) || 0;
        };

        const formatarDataParaInput = (dataDDMMYYYY) => {
            if (!dataDDMMYYYY) return '';
            const partes = dataDDMMYYYY.split('/');
            if (partes.length === 3) return `${partes[2]}-${partes[1]}-${partes[0]}`; 
            return dataDDMMYYYY; 
        };
        
        const dadosIniciais = [
            { data: "05/04/2025", nomeCliente: "Cliente Alpha", ordemBM: "11C2DFD793", plataforma: "ALIPAY", statusOrdem: "Concluída", volumeCNY: 1000.00, valorVenda: 442.00, valorCusto: 380.00, descricaoOrdem: "Venda de eletrônicos importados para Cliente Alpha.", id: 1717544000001 },
            { data: "09/04/2025", nomeCliente: "Cliente Beta", ordemBM: "11C2DFD797", plataforma: "CSSBUY", statusOrdem: "Concluída", volumeCNY: 1000.00, valorVenda: 398.00, valorCusto: 380.00, descricaoOrdem: "Pedido de cliente regular, baixo volume.", id: 1717544000002 },
            { data: "15/04/2025", nomeCliente: "Cliente Gamma", ordemBM: "11C2DFD805", plataforma: "ALIPAY", statusOrdem: "Concluída", volumeCNY: 1500.00, valorVenda: 680.00, valorCusto: 550.00, descricaoOrdem: "Venda com margem alta para Cliente Gamma, produto específico.", id: 1717544000003 },
            { data: "20/04/2025", nomeCliente: "Cliente Delta", ordemBM: "11C2DFD812", plataforma: "Outra", statusOrdem: "Pendente",  volumeCNY: 800.00,  valorVenda: 350.00, valorCusto: 300.00, descricaoOrdem: "Aguardando pagamento do Cliente Delta.", id: 1717544000004 },
            { data: "28/04/2025", nomeCliente: "Cliente Epsilon", ordemBM: "11C2DFD820", plataforma: "CSSBUY", statusOrdem: "Concluída", volumeCNY: 1200.00, valorVenda: 500.00, valorCusto: 450.00, descricaoOrdem: "Venda promocional para Cliente Epsilon, grande quantidade.", id: 1717544000005 },
            { data: "03/06/2025", nomeCliente: "Cliente Zeta", ordemBM: "9061E5C0F0", plataforma: "CSSBUY", statusOrdem: "Concluída", volumeCNY: 122.35, valorVenda: 122.35, valorCusto: 125.00, descricaoOrdem: "Ordem CSSBUY (Exemplo) para Cliente Zeta.", id: 1717544000006 },
            { data: "03/06/2025", nomeCliente: "Cliente Eta", ordemBM: "71BD89D53A", plataforma: "CSSBUY", statusOrdem: "Concluída", volumeCNY: 450.00, valorVenda: 450.00, valorCusto: 443.53, descricaoOrdem: "Ordem CSSBUY (Exemplo) para Cliente Eta.", id: 1717544000007 },
        ].map((item, index) => ({ 
            ...item,
            data: formatarDataParaInput(item.data),
            id: item.id || Date.now() + index 
        }));
        
        const dadosIniciaisReembolsos = [
            { dataReembolso: formatarDataParaInput("10/04/2025"), nomeCliente: "Cliente X", clienteOrdemOriginal: "11C2DFD793", valorReembolso: 50.00, plataformaReembolso: "ALIPAY", observacoesReembolso: "Produto com defeito", id: Date.now() - 100000 }
        ];

        function popularDadosIniciaisSeVazio() {
            const transacoesSalvas = localStorage.getItem('transacoesPlataforma');
            if (!transacoesSalvas || JSON.parse(transacoesSalvas).length === 0) {
                localStorage.setItem('transacoesPlataforma', JSON.stringify(dadosIniciais));
                transacoes = [...dadosIniciais];
                console.log("Transações iniciais populadas no localStorage.");
            }

            const reembolsosSalvos = localStorage.getItem('reembolsosPlataforma');
            if (!reembolsosSalvos || JSON.parse(reembolsosSalvos).length === 0) {
                localStorage.setItem('reembolsosPlataforma', JSON.stringify(dadosIniciaisReembolsos));
                reembolsos = [...dadosIniciaisReembolsos];
                console.log("Reembolsos iniciais populados no localStorage.");
            }
        }


        const switchTab = (activeTabId) => {
            ['lancamento', 'painel', 'reembolso'].forEach(tabId => {
                document.getElementById(`tab-${tabId}`).classList.remove('active');
                document.getElementById(`${tabId}-page`).classList.remove('active');
            });
            document.getElementById(`tab-${activeTabId}`).classList.add('active');
            document.getElementById(`${activeTabId}-page`).classList.add('active');
        };

        tabLancamento.addEventListener('click', () => switchTab('lancamento'));
        tabPainel.addEventListener('click', () => switchTab('painel'));
        tabReembolso.addEventListener('click', () => switchTab('reembolso'));
        
        const showToast = (message, isError = false) => {
            toastMessage.textContent = message;
            toastMessage.style.backgroundColor = isError ? '#ef4444' : '#22c55e';
            toastMessage.classList.add('show');
            setTimeout(() => { toastMessage.classList.remove('show');}, 3000);
        };

        const validarFormularioGeral = (formElement) => {
            let isValid = true;
            const inputsObrigatorios = formElement.querySelectorAll('[required]');
            inputsObrigatorios.forEach(input => {
                if (!input.value.trim()) { 
                    isValid = false;
                    input.classList.add('input-error');
                    input.addEventListener('input', () => input.classList.remove('input-error'), { once: true });
                } else {
                    input.classList.remove('input-error');
                }
            });
            const submitButton = formElement.querySelector('button[type="submit"]');
            if(submitButton) submitButton.disabled = !isValid;
            return isValid;
        };
        
        formLancamento.addEventListener('input', () => validarFormularioGeral(formLancamento));

        formLancamento.addEventListener('submit', async (event) => {
            event.preventDefault();
            if (!validarFormularioGeral(formLancamento)) {
                showToast("Por favor, preencha todos os campos obrigatórios destacados em vermelho.", true);
                return;
            }
            const novaTransacao = {
                data: dataInput.value, 
                ordemBM: document.getElementById('ordemBM').value,
                nomeCliente: nomeClienteInput.value.trim(),
                plataforma: document.getElementById('plataforma').value,
                statusOrdem: document.getElementById('statusOrdem').value,
                volumeCNY: parseFloat(document.getElementById('volumeCNY').value),
                valorVenda: parseFloat(document.getElementById('valorVenda').value),
                valorCusto: parseFloat(document.getElementById('valorCusto').value),
                descricaoOrdem: descricaoOrdemInput.value.trim(),
                id: Date.now() 
            };
            transacoes.push(novaTransacao);
            localStorage.setItem('transacoesPlataforma', JSON.stringify(transacoes));
            
            atualizarPainelAnalise();
            formLancamento.reset();
            dataInput.valueAsDate = new Date();
            validarFormularioGeral(formLancamento);
            showToast("Ordem salva com sucesso!");
            switchTab('painel');
        });

        btnGerarDescricaoIA.addEventListener('click', async () => {
            const ordemBM = document.getElementById('ordemBM').value;
            const nomeCliente = nomeClienteInput.value.trim();
            const volumeCNY = document.getElementById('volumeCNY').value;
            const valorVenda = document.getElementById('valorVenda').value;
            const plataforma = document.getElementById('plataforma').value;

            if (!ordemBM || !volumeCNY || !valorVenda) {
                showToast("Preencha Ordem (BM), Plataforma, Volume (CNY) e Valor de Venda para gerar a descrição.", true);
                return;
            }
            descricaoIASpinner.classList.remove('hidden');
            btnGerarDescricaoIA.disabled = true;
            const prompt = `Gere uma breve descrição concisa para uma ordem de venda na plataforma ${plataforma} para o cliente ${nomeCliente || 'não especificado'}, com os seguintes detalhes para um registro interno:
- ID da Ordem (BM): ${ordemBM}
- Volume (CNY): ${volumeCNY}
- Valor de Venda (BRL): ${valorVenda}
Foque nos aspectos chave para fácil identificação e resumo da transação. Máximo de 150 caracteres. Responda apenas com a descrição.`;
            try {
                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
                const payload = { contents: chatHistory };
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload)});
                if (!response.ok) throw new Error(`Erro da API: ${response.status}`);
                const result = await response.json();
                if (result.candidates && result.candidates[0].content && result.candidates[0].content.parts[0]) {
                    descricaoOrdemInput.value = result.candidates[0].content.parts[0].text.trim();
                    showToast("Descrição gerada pela IA!");
                } else { throw new Error("Resposta inesperada da API."); }
            } catch (error) {
                console.error("Erro ao gerar descrição com IA:", error);
                let errorMsgContent = "Erro ao gerar descrição. Tente novamente.";
                 if (error && typeof error.message === 'string') { 
                    errorMsgContent = `Erro ao gerar descrição: ${error.message}`;
                } else if (error) {
                    errorMsgContent = `Erro ao gerar descrição: ${JSON.stringify(error)}`;
                }
                showToast(errorMsgContent, true);
            } finally {
                descricaoIASpinner.classList.add('hidden');
                btnGerarDescricaoIA.disabled = false;
            }
        });

        const calcularDetalhesTransacao = (transacao) => {
            const valorVenda = transacao.valorVenda || transacao.valor_venda || 0; 
            const valorCusto = transacao.valorCusto || transacao.valor_custo || 0; 
            const lucroBRL = valorVenda - valorCusto;
            const margemPercent = valorCusto !== 0 ? (lucroBRL / valorCusto) * 100 : (lucroBRL > 0 ? 100 : 0);
            let zonaAcao;
            if (margemPercent >= LIMITE_ALTA_MARGEM) zonaAcao = "Alta";
            else if (margemPercent >= LIMITE_MEDIA_MARGEM) zonaAcao = "Média";
            else zonaAcao = "Baixa";
            return { ...transacao, lucroBRL, margemPercent, zonaAcao, valorVenda, valorCusto };
        };
        
        const formatCurrency = (value, currency = 'BRL') => {
            const options = { style: 'currency', currency, minimumFractionDigits: 2, maximumFractionDigits: 2 };
            if (currency === 'CNY') {
                options.currencyDisplay = 'symbol';
                return new Intl.NumberFormat('zh-CN', options).format(value);
            }
            return new Intl.NumberFormat('pt-BR', options).format(value);
        };

        const calcularMedia = (arr) => arr.length > 0 ? arr.reduce((a, b) => a + b, 0) / arr.length : 0;

        const calcularDesvioPadrao = (arr) => {
            if (arr.length < 2) return 0;
            const media = calcularMedia(arr);
            const diffQuadradasSoma = arr.reduce((acc, val) => acc + Math.pow(val - media, 2), 0);
            const variancia = diffQuadradasSoma / (arr.length -1); 
            return Math.sqrt(variancia);
        };
        
        const filtrarTransacoesPorPeriodo = (periodo, listaTransacoesParam = transacoes) => {
            const hoje = new Date();
            hoje.setHours(0, 0, 0, 0); 
            switch (periodo) {
                case 'hoje':
                    return listaTransacoesParam.filter(t => {
                        const dataTransacao = new Date(t.data); 
                        dataTransacao.setHours(0,0,0,0);
                        return dataTransacao.getTime() === hoje.getTime();
                    });
                case '7dias':
                    const seteDiasAtras = new Date(hoje);
                    seteDiasAtras.setDate(hoje.getDate() - 6); 
                    return listaTransacoesParam.filter(t => {
                        const dataTransacao = new Date(t.data);
                        return dataTransacao >= seteDiasAtras && dataTransacao <= new Date(hoje.getFullYear(), hoje.getMonth(), hoje.getDate(), 23, 59, 59, 999);
                    });
                case '30dias':
                    const trintaDiasAtras = new Date(hoje);
                    trintaDiasAtras.setDate(hoje.getDate() - 29);
                    return listaTransacoesParam.filter(t => {
                         const dataTransacao = new Date(t.data);
                         return dataTransacao >= trintaDiasAtras && dataTransacao <= new Date(hoje.getFullYear(), hoje.getMonth(), hoje.getDate(), 23, 59, 59, 999);
                    });
                case 'todos': default: return [...listaTransacoesParam];
            }
        };
        
        const filtrarReembolsosPorPeriodo = (periodo, listaReembolsosParam = reembolsos) => {
            const hoje = new Date();
            hoje.setHours(0, 0, 0, 0);
            switch (periodo) {
                case 'hoje':
                    return listaReembolsosParam.filter(r => {
                        const dataReembolso = new Date(r.dataReembolso); 
                        dataReembolso.setHours(0,0,0,0);
                        return dataReembolso.getTime() === hoje.getTime();
                    });
                case '7dias':
                    const seteDiasAtras = new Date(hoje);
                    seteDiasAtras.setDate(hoje.getDate() - 6);
                    return listaReembolsosParam.filter(r => {
                        const dataReembolso = new Date(r.dataReembolso);
                        return dataReembolso >= seteDiasAtras && dataReembolso <= new Date(hoje.getFullYear(), hoje.getMonth(), hoje.getDate(), 23, 59, 59, 999);
                    });
                case '30dias':
                    const trintaDiasAtras = new Date(hoje);
                    trintaDiasAtras.setDate(hoje.getDate() - 29);
                    return listaReembolsosParam.filter(r => {
                         const dataReembolso = new Date(r.dataReembolso);
                         return dataReembolso >= trintaDiasAtras && dataReembolso <= new Date(hoje.getFullYear(), hoje.getMonth(), hoje.getDate(), 23, 59, 59, 999);
                    });
                case 'todos': default: return [...listaReembolsosParam];
            }
        };

        const getTransacoesFiltradas = () => {
            let transacoesFiltradasPeriodo = filtrarTransacoesPorPeriodo(periodoSelecionado, transacoes);
            if (plataformaSelecionada !== 'todas') {
                transacoesFiltradasPeriodo = transacoesFiltradasPeriodo.filter(t => t.plataforma === plataformaSelecionada);
            }
            return transacoesFiltradasPeriodo;
        };

        const getReembolsosFiltrados = () => {
             let reembolsosFiltradosPeriodo = filtrarReembolsosPorPeriodo(periodoSelecionado, reembolsos);
             if (plataformaSelecionada !== 'todas') {
                reembolsosFiltradosPeriodo = reembolsosFiltradosPeriodo.filter(r => r.plataformaReembolso === plataformaSelecionada); 
             }
             return reembolsosFiltradosPeriodo;
        }


        const atualizarPainelAnalise = () => {
            const transacoesDoPeriodoEPlataforma = getTransacoesFiltradas();
            const reembolsosDoPeriodoEPlataforma = getReembolsosFiltrados(); 
            const totalReembolsadoNoPeriodo = reembolsosDoPeriodoEPlataforma.reduce((sum, r) => sum + r.valorReembolso, 0); 

            const tabelaCorpoRecentes = document.getElementById('tabelaLancamentosCorpo');
            tabelaCorpoRecentes.innerHTML = '';
             if (transacoes.length === 0) {
                tabelaCorpoRecentes.innerHTML = '<tr><td colspan="5" class="px-4 py-4 text-center text-gray-500">Nenhum lançamento ainda.</td></tr>';
            } else {
                [...transacoes].sort((a,b) => new Date(b.data).getTime() - new Date(a.data).getTime() || b.id - a.id).slice(0, 10).forEach(t => { 
                    const detalhes = calcularDetalhesTransacao(t);
                    const row = tabelaCorpoRecentes.insertRow();
                    
                    const cellData = row.insertCell();
                    cellData.className = "px-4 py-3 whitespace-nowrap text-sm";
                    cellData.textContent = new Date(detalhes.data).toLocaleDateString('pt-BR', {timeZone: 'UTC'});

                    const cellOrdem = row.insertCell();
                    cellOrdem.textContent = detalhes.ordemBM || detalhes.ordem_bm; 
                    cellOrdem.className = "px-4 py-3 whitespace-nowrap text-sm clickable-order";
                    cellOrdem.title = `Ver detalhes da ordem ${detalhes.ordemBM || detalhes.ordem_bm}`;
                    cellOrdem.onclick = () => { abrirModalVerTodosFiltrado(detalhes.id); }; 
                    
                    const cellLucro = row.insertCell();
                    cellLucro.className = "px-4 py-3 whitespace-nowrap text-sm";
                    cellLucro.textContent = formatCurrency(detalhes.lucroBRL);

                    const cellMargem = row.insertCell();
                    cellMargem.className = "px-4 py-3 whitespace-nowrap text-sm";
                    cellMargem.textContent = `${detalhes.margemPercent.toFixed(2)}%`;
                    
                    const cellZona = row.insertCell();
                    cellZona.className = "px-4 py-3 whitespace-nowrap text-sm";
                    cellZona.innerHTML = `<span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full zona-${detalhes.zonaAcao.toLowerCase()}">${detalhes.zonaAcao}</span>`;
                });
            }

            if (transacoesDoPeriodoEPlataforma.length === 0 && (periodoSelecionado !== 'todos' || plataformaSelecionada !== 'todas') && transacoes.length > 0) {
                 const nomePeriodo = periodoSelectorContainer.querySelector(`[data-periodo="${periodoSelecionado}"]`).textContent;
                 const nomePlataforma = plataformaFilterSelectorContainer.querySelector(`[data-plataforma="${plataformaSelecionada}"]`).textContent;
                 showToast(`Nenhum dado para Período: ${nomePeriodo} e Plataforma: ${nomePlataforma}.`, true);
            } else if (transacoes.length === 0 && reembolsos.length === 0) { 
                 // Não mostrar toast aqui
            }
            
            let lucroTotalBRLBrutoPeriodo = 0;
            let custoTotalPeriodo = 0;
            let transacoesProcessadasPeriodo = [];

            if (transacoesDoPeriodoEPlataforma.length > 0) {
                transacoesProcessadasPeriodo = transacoesDoPeriodoEPlataforma.map(t => calcularDetalhesTransacao(t));
                lucroTotalBRLBrutoPeriodo = transacoesProcessadasPeriodo.reduce((sum, t) => sum + t.lucroBRL, 0);
                custoTotalPeriodo = transacoesProcessadasPeriodo.reduce((sum, t) => sum + t.valorCusto, 0); 
            }
            
            const lucroTotalBRLLiquidoPeriodo = lucroTotalBRLBrutoPeriodo - totalReembolsadoNoPeriodo;
            const margemGeralPeriodo = custoTotalPeriodo !== 0 ? (lucroTotalBRLLiquidoPeriodo / custoTotalPeriodo) * 100 : (lucroTotalBRLLiquidoPeriodo > 0 ? 100 : 0);
            const zonaAcaoGeralCalculatedPeriodo = calcularDetalhesTransacao({valorVenda: 0, valorCusto: 0, margemPercent: margemGeralPeriodo}).zonaAcao; 

            document.getElementById('lucroTotalBRL').textContent = formatCurrency(lucroTotalBRLLiquidoPeriodo);
            document.getElementById('margemGeral').textContent = `${margemGeralPeriodo.toFixed(2)}%`;
            document.getElementById('totalVendas').textContent = transacoesProcessadasPeriodo.length;
            const zonaGeralEl = document.getElementById('zonaAcaoGeral');
            zonaGeralEl.textContent = zonaAcaoGeralCalculatedPeriodo;
            zonaGeralEl.className = `text-xl font-bold mt-2 py-2 px-4 rounded-full inline-block zona-${zonaAcaoGeralCalculatedPeriodo.toLowerCase()}`;

            const totalCNYPeriodo = transacoesProcessadasPeriodo.reduce((s, t) => s + t.volumeCNY, 0); 
            const totalVendasBRLPeriodo = transacoesProcessadasPeriodo.reduce((s, t) => s + t.valorVenda, 0); 
            const numTransacoesPeriodo = transacoesProcessadasPeriodo.length;
            
            document.getElementById('totalCNY').textContent = formatCurrency(totalCNYPeriodo, 'CNY');
            document.getElementById('totalVendasBRL').textContent = formatCurrency(totalVendasBRLPeriodo);
            document.getElementById('ticketMedioBRL').textContent = formatCurrency(numTransacoesPeriodo > 0 ? totalVendasBRLPeriodo / numTransacoesPeriodo : 0);
            document.getElementById('lucroMedio').textContent = formatCurrency(numTransacoesPeriodo > 0 ? lucroTotalBRLLiquidoPeriodo / numTransacoesPeriodo : 0);

            const ocorrenciasZonaPeriodo = { Alta: 0, Média: 0, Baixa: 0 };
            const volumeBRLZonaPeriodo = { Alta: 0, Média: 0, Baixa: 0 };
            transacoesProcessadasPeriodo.forEach(t => {
                ocorrenciasZonaPeriodo[t.zonaAcao]++;
                volumeBRLZonaPeriodo[t.zonaAcao] += t.lucroBRL;
            });
            document.getElementById('zonaAltaOcorrencias').textContent = ocorrenciasZonaPeriodo.Alta;
            document.getElementById('zonaMediaOcorrencias').textContent = ocorrenciasZonaPeriodo.Média;
            document.getElementById('zonaBaixaOcorrencias').textContent = ocorrenciasZonaPeriodo.Baixa;
            document.getElementById('zonaAltaVolume').textContent = volumeBRLZonaPeriodo.Alta.toFixed(2);
            document.getElementById('zonaMediaVolume').textContent = volumeBRLZonaPeriodo.Média.toFixed(2);
            document.getElementById('zonaBaixaVolume').textContent = volumeBRLZonaPeriodo.Baixa.toFixed(2);
            
            const margensIndividuaisPeriodo = transacoesProcessadasPeriodo.map(t => t.margemPercent);
            const desvioPadraoMargemPeriodo = calcularDesvioPadrao(margensIndividuaisPeriodo);
            document.getElementById('desvioPadraoMargem').textContent = `${desvioPadraoMargemPeriodo.toFixed(2)}%`;
            
            let interpretacaoTendencia, classeTendencia;
             if (transacoesProcessadasPeriodo.length < 2) {
                interpretacaoTendencia = "Dados insuficientes";
                classeTendencia = 'neutra';
            } else if (desvioPadraoMargemPeriodo <= LIMITE_TENDENCIA_ESTAVEL) {
                interpretacaoTendencia = "Estável (ideal)";
                classeTendencia = 'estavel';
            } else if (desvioPadraoMargemPeriodo <= LIMITE_TENDENCIA_MODERADA) {
                interpretacaoTendencia = "Moderada";
                classeTendencia = 'moderada';
            } else {
                interpretacaoTendencia = "Alta variação";
                classeTendencia = 'altavariacao';
            }
            const interEl = document.getElementById('interpretacaoTendencia');
            interEl.textContent = interpretacaoTendencia;
            interEl.className = `text-md font-semibold p-2 rounded-full tendencia-${classeTendencia}`;
            renderTendenciaChart(desvioPadraoMargemPeriodo);
        };

        const renderTendenciaChart = (desvioPadrao) => {
            const ctx = document.getElementById('tendenciaChart').getContext('2d');
            if (tendenciaChartInstance) tendenciaChartInstance.destroy();
            tendenciaChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Desvio Padrão Atual'],
                    datasets: [{
                        label: 'Desvio Padrão da Margem (%)',
                        data: [desvioPadrao],
                        backgroundColor: ['#8b5cf6'],
                        borderColor: ['#7c3aed'],
                        borderWidth: 1,
                        barThickness: 50,
                        borderRadius: 4,
                    }]
                },
                options: {
                    indexAxis: 'y', maintainAspectRatio: false, responsive: true,
                    plugins: { legend: { display: false }, tooltip: { callbacks: { label: (c) => `${c.dataset.label}: ${c.raw.toFixed(2)}%` }}},
                    scales: { x: { beginAtZero: true, title: { display: true, text: 'Percentual de Desvio (%)', font: {size: 12, weight: '500'}}, grid: { display: false }, ticks: { callback: (v) => v + '%' }}, y: { display: false }},
                    layout: { padding: { top: 10, bottom: 10 }}
                }
            });
        };
        
        const popularTabelaModal = (dadosParaExibir) => {
            tabelaModalTodosLancamentos.innerHTML = '';
            if (dadosParaExibir.length === 0) {
                tabelaModalTodosLancamentos.innerHTML = '<tr><td colspan="11" class="px-4 py-4 text-center text-gray-500">Nenhum lançamento encontrado para os filtros aplicados.</td></tr>';
                return;
            }
            dadosParaExibir.forEach(t => {
                const detalhes = calcularDetalhesTransacao(t); 
                const row = tabelaModalTodosLancamentos.insertRow();
                row.insertCell().textContent = new Date(detalhes.data).toLocaleDateString('pt-BR', {timeZone: 'UTC'});
                row.insertCell().textContent = detalhes.ordemBM || detalhes.ordem_bm; 
                row.insertCell().textContent = detalhes.nomeCliente || detalhes.nome_cliente || '-'; 
                row.insertCell().textContent = detalhes.plataforma || 'N/A';
                const statusCell = row.insertCell();
                const statusSelect = document.createElement('select');
                statusSelect.className = 'status-select text-sm';
                statusSelect.dataset.id = detalhes.id; 
                ['Concluída', 'Pendente', 'Cancelada'].forEach(opt => {
                    const option = document.createElement('option');
                    option.value = opt;
                    option.text = opt;
                    if (opt === (detalhes.statusOrdem || detalhes.status_ordem)) option.selected = true; 
                    statusSelect.appendChild(option);
                });
                statusCell.appendChild(statusSelect);
                row.insertCell().textContent = formatCurrency(detalhes.volumeCNY || detalhes.volume_cny, 'CNY'); 
                row.insertCell().textContent = formatCurrency(detalhes.valorVenda || detalhes.valor_venda); 
                row.insertCell().textContent = formatCurrency(detalhes.valorCusto || detalhes.valor_custo); 
                row.insertCell().textContent = formatCurrency(detalhes.lucroBRL); 
                row.insertCell().textContent = `${detalhes.margemPercent.toFixed(2)}%`; 
                row.insertCell().innerHTML = `<span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full zona-${detalhes.zonaAcao.toLowerCase()}">${detalhes.zonaAcao}</span>`; 
                const acoesCell = row.insertCell();
                const editButton = document.createElement('button');
                editButton.innerHTML = `<svg class="edit-btn-icon text-indigo-600 hover:text-indigo-800" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"></path></svg>`;
                editButton.title = "Editar Lançamento";
                editButton.classList.add('p-1');
                editButton.dataset.id = detalhes.id; 
                editButton.addEventListener('click', () => abrirModalEdicao(detalhes.id));
                acoesCell.appendChild(editButton);
            });
        };

        const abrirModalVerTodosFiltrado = (transacaoId) => {
            const transacaoEspecifica = transacoes.find(t => t.id === transacaoId);
            if (transacaoEspecifica) {
                popularTabelaModal([transacaoEspecifica]);
                modalVerTodos.classList.add('active');
                filtroDataInicio.value = ''; 
                filtroDataFim.value = '';
                modalPlataformaFilterContainer.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
                modalPlataformaFilterContainer.querySelector('[data-plataforma="todas"]').classList.add('active');
                plataformaModalSelecionada = 'todas';
            }
        };

        tabelaModalTodosLancamentos.addEventListener('change', (event) => {
            if (event.target.classList.contains('status-select')) {
                const transacaoId = parseInt(event.target.dataset.id); 
                const novoStatus = event.target.value;
                const indexTransacao = transacoes.findIndex(t => t.id === transacaoId);
                if (indexTransacao > -1) {
                    transacoes[indexTransacao].statusOrdem = novoStatus;
                    transacoes[indexTransacao].status_ordem = novoStatus; 
                    localStorage.setItem('transacoesPlataforma', JSON.stringify(transacoes));
                    atualizarPainelAnalise();
                    showToast(`Status da ordem ${transacoes[indexTransacao].ordemBM || transacoes[indexTransacao].ordem_bm} atualizado para ${novoStatus}.`);
                }
            }
        });


        btnVerTodos.addEventListener('click', () => {
            filtroDataInicio.value = '';
            filtroDataFim.value = '';
            modalPlataformaFilterContainer.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
            modalPlataformaFilterContainer.querySelector('[data-plataforma="todas"]').classList.add('active');
            plataformaModalSelecionada = 'todas';
            popularTabelaModal([...transacoes].sort((a,b) => new Date(b.data).getTime() - new Date(a.data).getTime() || b.id - a.id));
            modalVerTodos.classList.add('active');
        });
        btnFecharModalVerTodos.addEventListener('click', () => modalVerTodos.classList.remove('active'));
        modalVerTodos.addEventListener('click', (event) => { 
            if (event.target === modalVerTodos) modalVerTodos.classList.remove('active');
        });

        modalPlataformaFilterContainer.addEventListener('click', (event) => {
            if (event.target.tagName === 'BUTTON') {
                modalPlataformaFilterContainer.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
                event.target.classList.add('active');
                plataformaModalSelecionada = event.target.dataset.plataforma;
            }
        });

        btnAplicarFiltro.addEventListener('click', () => {
            const dataInicio = filtroDataInicio.value;
            const dataFim = filtroDataFim.value;
            let transacoesFiltradas = [...transacoes];

            if (dataInicio) transacoesFiltradas = transacoesFiltradas.filter(t => new Date(t.data) >= new Date(dataInicio));
            if (dataFim) transacoesFiltradas = transacoesFiltradas.filter(t => new Date(t.data) <= new Date(dataFim));
            if (plataformaModalSelecionada !== 'todas') {
                transacoesFiltradas = transacoesFiltradas.filter(t => t.plataforma === plataformaModalSelecionada);
            }
            popularTabelaModal(transacoesFiltradas.sort((a,b) => new Date(b.data).getTime() - new Date(a.data).getTime() || b.id - a.id ));
        });

        const abrirModalEdicao = (transacaoId) => {
            const transacaoParaEditar = transacoes.find(t => t.id === transacaoId);
            if (!transacaoParaEditar) return;
            document.getElementById('editTransacaoId').value = transacaoParaEditar.id; 
            document.getElementById('editData').value = transacaoParaEditar.data;
            document.getElementById('editNomeCliente').value = transacaoParaEditar.nomeCliente || transacaoParaEditar.nome_cliente || ''; 
            document.getElementById('editOrdemBM').value = transacaoParaEditar.ordemBM || transacaoParaEditar.ordem_bm; 
            document.getElementById('editPlataforma').value = transacaoParaEditar.plataforma || 'Outra';
            document.getElementById('editStatusOrdem').value = transacaoParaEditar.statusOrdem || transacaoParaEditar.status_ordem; 
            document.getElementById('editVolumeCNY').value = transacaoParaEditar.volumeCNY || transacaoParaEditar.volume_cny; 
            document.getElementById('editValorVenda').value = transacaoParaEditar.valorVenda || transacaoParaEditar.valor_venda; 
            document.getElementById('editValorCusto').value = transacaoParaEditar.valorCusto || transacaoParaEditar.valor_custo; 
            document.getElementById('editDescricaoOrdem').value = transacaoParaEditar.descricaoOrdem || transacaoParaEditar.descricao_ordem || ''; 
            validarFormularioGeral(formEditarLancamento);
            modalEditarLancamento.classList.add('active');
        };
        
        formEditarLancamento.addEventListener('input', () => validarFormularioGeral(formEditarLancamento));
        
        formEditarLancamento.addEventListener('submit', async (event) => {
            event.preventDefault();
            if (!validarFormularioGeral(formEditarLancamento)) {
                showToast("Por favor, preencha todos os campos obrigatórios na edição.", true);
                return;
            }
            const idParaEditar = parseInt(document.getElementById('editTransacaoId').value);
            const indexTransacao = transacoes.findIndex(t => t.id === idParaEditar);

            if (indexTransacao > -1) {
                transacoes[indexTransacao] = {
                    id: idParaEditar, 
                    data: document.getElementById('editData').value,
                    nomeCliente: document.getElementById('editNomeCliente').value.trim(), 
                    ordemBM: document.getElementById('editOrdemBM').value,
                    plataforma: document.getElementById('editPlataforma').value,
                    statusOrdem: document.getElementById('editStatusOrdem').value,
                    volumeCNY: parseFloat(document.getElementById('editVolumeCNY').value),
                    valorVenda: parseFloat(document.getElementById('editValorVenda').value),
                    valorCusto: parseFloat(document.getElementById('editValorCusto').value),
                    descricaoOrdem: document.getElementById('editDescricaoOrdem').value.trim(),
                    updated_at: new Date().toISOString() 
                };
                localStorage.setItem('transacoesPlataforma', JSON.stringify(transacoes));
                atualizarPainelAnalise();
                popularTabelaModal([...transacoes].sort((a,b) => new Date(b.data).getTime() - new Date(a.data).getTime() || b.id - a.id )); 
                modalEditarLancamento.classList.remove('active');
                showToast("Lançamento atualizado com sucesso!");
            } else {
                showToast("Erro ao encontrar lançamento para editar.", true);
            }
        });

        btnFecharModalEditar.addEventListener('click', () => modalEditarLancamento.classList.remove('active'));
        modalEditarLancamento.addEventListener('click', (event) => {
            if (event.target === modalEditarLancamento) modalEditarLancamento.classList.remove('active');
        });

        btnAnalisarEstrategiaIA.addEventListener('click', async () => {
            conteudoAnaliseIA.innerHTML = '<div class="gemini-spinner"></div><p class="text-center text-gray-500">Aguarde, analisando sua estratégia...</p>';
            modalAnaliseIA.classList.add('active');
            const transacoesDoPeriodo = getTransacoesFiltradas();
            const reembolsosDoPeriodo = getReembolsosFiltrados();
            const totalReembolsadoNoPeriodo = reembolsosDoPeriodo.reduce((sum, r) => sum + r.valorReembolso, 0);

            const resumoDados = transacoesDoPeriodo.map(t => ({ lucro: t.lucroBRL, margem: t.margemPercent, zonaAcao: t.zonaAcao, plataforma: t.plataforma }));
            const lucroTotalPeriodo = document.getElementById('lucroTotalBRL').textContent; 
            const margemGeralPeriodo = document.getElementById('margemGeral').textContent;
            const zonaAcaoGeralPeriodo = document.getElementById('zonaAcaoGeral').textContent;
            const desvioPadraoPeriodo = document.getElementById('desvioPadraoMargem').textContent;
            const interpretacaoTendPeriodo = document.getElementById('interpretacaoTendencia').textContent;
            const nomePlataformaSelecionada = plataformaFilterSelectorContainer.querySelector('.active').textContent;
            const perguntaUsuario = perguntaEspecificaIAInput.value.trim();

            const prompt = `Você é um consultor financeiro especialista em e-commerce e pequenos negócios. Analise os seguintes dados de desempenho de vendas para o período selecionado (${periodoSelectorContainer.querySelector('.active').textContent}) e plataforma selecionada (${nomePlataformaSelecionada}), e forneça um conselho estratégico detalhado e acionável em português do Brasil. Considere os pontos fortes, fracos e oportunidades. Seja específico nas recomendações, utilizando marcadores para facilitar a leitura.
Métricas do Período e Plataforma Selecionados:
- Lucro Líquido Total (BRL) (após reembolsos): ${lucroTotalPeriodo}
- Total Reembolsado no Período (BRL): ${formatCurrency(totalReembolsadoNoPeriodo)}
- Margem Geral (baseada no lucro líquido): ${margemGeralPeriodo}
- Zona de Ação Geral (baseada na margem líquida): ${zonaAcaoGeralPeriodo}
- Desvio Padrão da Margem (calculado sobre margens brutas individuais): ${desvioPadraoPeriodo}
- Interpretação da Tendência Atual (baseada no desvio padrão): ${interpretacaoTendPeriodo}
- Número de Transações no Período: ${transacoesDoPeriodo.length}
- Resumo das Transações (primeiras 5, se houver - lucro e margem são brutos por transação): ${JSON.stringify(resumoDados.slice(0,5), null, 2)}
${perguntaUsuario ? `\nAdicionalmente, o usuário pergunta: "${perguntaUsuario}". Por favor, aborde esta questão na sua análise.` : ''}
Conselho Estratégico Detalhado (em marcadores):`;
            try {
                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const chatHistory = [{ role: "user", parts: [{ text: prompt }] }];
                const payload = { contents: chatHistory };
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload)});
                if (!response.ok) throw new Error(`Erro da API: ${response.status} ${response.statusText}`);
                const result = await response.json();
                let iaText = "Não foi possível obter uma análise da IA neste momento.";
                if (result.candidates && result.candidates[0].content && result.candidates[0].content.parts[0]) {
                    iaText = result.candidates[0].content.parts[0].text;
                }
                conteudoAnaliseIA.innerHTML = iaText.replace(/\n/g, '<br>');
            } catch (error) {
                console.error("Erro ao chamar Gemini API:", error);
                let errorMsgContent = `<p class="text-red-500">Erro ao obter análise da IA. Tente novamente mais tarde.</p>`;
                 if (error && typeof error.message === 'string') {
                    errorMsgContent = `<p class="text-red-500">Erro ao obter análise da IA: ${error.message}. Tente novamente mais tarde.</p>`;
                } else if (error) {
                     errorMsgContent = `<p class="text-red-500">Erro ao obter análise da IA: ${JSON.stringify(error)}. Tente novamente mais tarde.</p>`;
                }
                conteudoAnaliseIA.innerHTML = errorMsgContent;
            }
        });

        btnFecharModalIA.addEventListener('click', () => {
            modalAnaliseIA.classList.remove('active');
            perguntaEspecificaIAInput.value = ''; // Limpa o campo ao fechar
        });
        modalAnaliseIA.addEventListener('click', (event) => {
            if (event.target === modalAnaliseIA) {
                modalAnaliseIA.classList.remove('active');
                perguntaEspecificaIAInput.value = ''; // Limpa o campo ao fechar
            }
        });

        periodoSelectorContainer.addEventListener('click', (event) => {
            if (event.target.tagName === 'BUTTON') {
                periodoSelectorContainer.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
                event.target.classList.add('active');
                periodoSelecionado = event.target.dataset.periodo;
                atualizarPainelAnalise();
            }
        });
        
        plataformaFilterSelectorContainer.addEventListener('click', (event) => {
            if (event.target.tagName === 'BUTTON') {
                plataformaFilterSelectorContainer.querySelectorAll('button').forEach(btn => btn.classList.remove('active'));
                event.target.classList.add('active');
                plataformaSelecionada = event.target.dataset.plataforma;
                atualizarPainelAnalise();
            }
        });


        const atualizarDisplayReembolsos = () => {
            tabelaReembolsos.innerHTML = '';
            let totalReembolsadoValor = 0;
            if (reembolsos.length === 0) {
                tabelaReembolsos.innerHTML = '<tr><td colspan="7" class="px-4 py-4 text-center text-gray-500">Nenhum reembolso registrado.</td></tr>';
            } else {
                [...reembolsos].sort((a,b) => new Date(b.dataReembolso).getTime() - new Date(a.dataReembolso).getTime() || b.id - a.id).forEach(r => { 
                    const row = tabelaReembolsos.insertRow();
                    row.innerHTML = `
                        <td class="px-4 py-3 whitespace-nowrap text-sm">${new Date(r.dataReembolso).toLocaleDateString('pt-BR', {timeZone: 'UTC'})}</td>
                        <td class="px-4 py-3 whitespace-nowrap text-sm">${r.nomeCliente || '-'}</td>
                        <td class="px-4 py-3 whitespace-nowrap text-sm">${r.clienteOrdemOriginal || '-'}</td>
                        <td class="px-4 py-3 whitespace-nowrap text-sm text-red-600">${formatCurrency(r.valorReembolso)}</td>
                        <td class="px-4 py-3 whitespace-nowrap text-sm">${r.plataformaReembolso}</td>
                        <td class="px-4 py-3 text-sm">${r.observacoesReembolso || '-'}</td>
                        <td class="px-4 py-3 whitespace-nowrap text-sm">${formatCurrency(-r.valorReembolso)}</td>
                    `;
                    totalReembolsadoValor += r.valorReembolso;
                });
            }
            totalReembolsadoEl.textContent = formatCurrency(totalReembolsadoValor);
            numeroReembolsosEl.textContent = reembolsos.length;
        };
        
        formReembolso.addEventListener('input', () => validarFormularioGeral(formReembolso));
        formReembolso.addEventListener('submit', async (event) => {
            event.preventDefault();
            if (!validarFormularioGeral(formReembolso)) {
                showToast("Por favor, preencha os campos obrigatórios do reembolso.", true);
                return;
            }
            const novoReembolso = {
                dataReembolso: document.getElementById('dataReembolso').value, 
                nomeCliente: document.getElementById('reembolsoNomeCliente').value.trim(),
                clienteOrdemOriginal: document.getElementById('clienteOrdemOriginal').value,
                valorReembolso: parseFloat(document.getElementById('valorReembolso').value),
                plataformaReembolso: document.getElementById('plataformaReembolso').value,
                observacoesReembolso: document.getElementById('observacoesReembolso').value.trim(),
                id: Date.now() 
            };
            reembolsos.push(novoReembolso);
            localStorage.setItem('reembolsosPlataforma', JSON.stringify(reembolsos));
            atualizarDisplayReembolsos();
            atualizarPainelAnalise(); 
            formReembolso.reset();
            document.getElementById('dataReembolso').valueAsDate = new Date();
            validarFormularioGeral(formReembolso);
            showToast("Reembolso adicionado com sucesso!");
        });

        // Nova função para Análise de Reembolso com IA
        if(btnAnalisarReembolsosIA) { // Verifica se o botão existe no HTML
            btnAnalisarReembolsosIA.addEventListener('click', async () => {
                document.getElementById('conteudoAnaliseReembolsoIA').innerHTML = '<div class="gemini-spinner"></div><p class="text-center text-gray-500">Aguarde, analisando padrões de reembolso...</p>';
                modalAnaliseReembolsoIA.classList.add('active');

                const ultimosReembolsos = [...reembolsos].sort((a,b) => new Date(b.dataReembolso).getTime() - new Date(a.dataReembolso).getTime()).slice(0, 10);

                const promptReembolso = `Você é um analista de operações de e-commerce. Analise os seguintes dados dos últimos ${ultimosReembolsos.length} reembolsos. Identifique possíveis padrões, como plataformas com mais reembolsos, valores médios, e se possível, temas comuns nas observações. Forneça sugestões em português do Brasil para reduzir a taxa de reembolso ou melhorar processos relacionados.
    Dados de Reembolsos:
    - Total Geral Reembolsado (BRL): ${totalReembolsadoEl.textContent}
    - Número Total de Reembolsos: ${numeroReembolsosEl.textContent}
    - Lista dos Últimos ${ultimosReembolsos.length} Reembolsos: 
    ${JSON.stringify(ultimosReembolsos.map(r => ({ data: r.dataReembolso, cliente: r.nomeCliente, ordem_ref: r.clienteOrdemOriginal, valor: r.valorReembolso, plataforma: r.plataformaReembolso, obs: r.observacoesReembolso?.substring(0, 50) + (r.observacoesReembolso?.length > 50 ? '...' : '') })), null, 2)}
    Análise e Sugestões (em marcadores):`;

                try {
                    const apiKey = ""; 
                    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                    const chatHistory = [{ role: "user", parts: [{ text: promptReembolso }] }];
                    const payload = { contents: chatHistory };
                    const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                    if (!response.ok) throw new Error(`Erro da API: ${response.status} ${response.statusText}`);
                    const result = await response.json();
                    let iaText = "Não foi possível obter uma análise dos reembolsos neste momento.";
                    if (result.candidates && result.candidates[0].content && result.candidates[0].content.parts[0]) {
                        iaText = result.candidates[0].content.parts[0].text;
                    }
                    document.getElementById('conteudoAnaliseReembolsoIA').innerHTML = iaText.replace(/\n/g, '<br>');
                } catch (error) {
                    console.error("Erro ao chamar Gemini API para análise de reembolsos:", error);
                     let errorMsgContent = `<p class="text-red-500">Erro ao obter análise de reembolsos. Tente novamente mais tarde.</p>`;
                    if (error && typeof error.message === 'string') {
                        errorMsgContent = `<p class="text-red-500">Erro ao obter análise de reembolsos: ${error.message}. Tente novamente mais tarde.</p>`;
                    } else if (error) {
                        errorMsgContent = `<p class="text-red-500">Erro ao obter análise de reembolsos: ${JSON.stringify(error)}. Tente novamente mais tarde.</p>`;
                    }
                    document.getElementById('conteudoAnaliseReembolsoIA').innerHTML = errorMsgContent;
                }
            });
        }
        if(btnFecharModalReembolsoIA){
            btnFecharModalReembolsoIA.addEventListener('click', () => modalAnaliseReembolsoIA.classList.remove('active'));
        }
        if(modalAnaliseReembolsoIA){
            modalAnaliseReembolsoIA.addEventListener('click', (event) => {
                if (event.target === modalAnaliseReembolsoIA) modalAnaliseReembolsoIA.classList.remove('active');
            });
        }
        // --- Fim da lógica de Análise de Reembolso com IA ---


        function carregarDadosDoLocalStorage() { 
            const displayStatus = document.getElementById('userIdDisplay'); 
            if (displayStatus) { 
                displayStatus.textContent = "Dados carregados localmente.";
            }
            
            const dadosTransacoesSalvos = localStorage.getItem('transacoesPlataforma');
            const dadosReembolsosSalvos = localStorage.getItem('reembolsosPlataforma');

            if (dadosTransacoesSalvos) {
                transacoes = JSON.parse(dadosTransacoesSalvos);
            } else {
                transacoes = [...dadosIniciais.map(d => ({...d, id: d.id || Date.now() + Math.random() }))]; 
                localStorage.setItem('transacoesPlataforma', JSON.stringify(transacoes));
            }

            if (dadosReembolsosSalvos) {
                reembolsos = JSON.parse(dadosReembolsosSalvos);
            } else {
                reembolsos = [...dadosIniciaisReembolsos.map(r => ({...r, id: r.id || Date.now() + Math.random() }))];
                localStorage.setItem('reembolsosPlataforma', JSON.stringify(reembolsos));
            }
            
            console.log("Transações carregadas do localStorage:", transacoes);
            console.log("Reembolsos carregados do localStorage:", reembolsos);

            atualizarPainelAnalise();
            atualizarDisplayReembolsos();
            renderizarCalendario(); 
        }


        async function inicializarApp() {
            const localDataInput = document.getElementById('data');
            if (localDataInput) {
                localDataInput.valueAsDate = new Date();
                localDataInput.addEventListener('focus', () => { localDataInput.valueAsDate = new Date(); });
            }
            const dataReembolsoInput = document.getElementById('dataReembolso');
            if (dataReembolsoInput) {
                dataReembolsoInput.valueAsDate = new Date();
            }
            
            const localAuthStatusDisplay = document.getElementById('userIdDisplay'); 
            if(localAuthStatusDisplay) { 
                localAuthStatusDisplay.textContent = "Carregando dados...";
            }
            
            carregarDadosDoLocalStorage(); 
            
            validarFormularioGeral(formLancamento); 
            validarFormularioGeral(formEditarLancamento);
            validarFormularioGeral(formReembolso);
            
            let showPanel = transacoes.length > 0 || reembolsos.length > 0;
            switchTab(showPanel ? 'painel' : 'lancamento');

            if(localAuthStatusDisplay) { 
                if (transacoes.length > 0 || reembolsos.length > 0) {
                    localAuthStatusDisplay.textContent = "Dados locais carregados.";
                } else {
                    localAuthStatusDisplay.textContent = "Nenhum dado local. Dados de exemplo carregados.";
                }
            }
        }
        
        const diasDaSemanaNomes = ['Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb'];

        function renderizarCalendario() {
            calendarGrid.innerHTML = '';
            calendarDayNamesContainer.innerHTML = ''; 
            diasDaSemanaNomes.forEach(dia => {
                const diaEl = document.createElement('div');
                diaEl.className = 'calendar-day-name';
                diaEl.textContent = dia;
                calendarDayNamesContainer.appendChild(diaEl);
            });


            const ano = currentCalendarDate.getFullYear();
            const mes = currentCalendarDate.getMonth(); 
            calendarMonthYear.textContent = `${currentCalendarDate.toLocaleString('pt-BR', { month: 'long' }).toUpperCase()} ${ano}`;

            const primeiroDiaDoMes = new Date(ano, mes, 1).getDay(); 
            const diasNoMes = new Date(ano, mes + 1, 0).getDate();

            for (let i = 0; i < primeiroDiaDoMes; i++) {
                const diaVazioEl = document.createElement('div');
                diaVazioEl.className = 'calendar-day other-month';
                calendarGrid.appendChild(diaVazioEl);
            }

            for (let dia = 1; dia <= diasNoMes; dia++) {
                const diaEl = document.createElement('div');
                diaEl.className = 'calendar-day';
                
                const diaNumeroEl = document.createElement('span');
                diaNumeroEl.className = 'day-number';
                diaNumeroEl.textContent = dia;
                diaEl.appendChild(diaNumeroEl);

                const dataAtual = new Date(ano, mes, dia);
                const hoje = new Date();
                if (dia === hoje.getDate() && mes === hoje.getMonth() && ano === hoje.getFullYear()) {
                    diaEl.classList.add('today');
                }

                const diaDaSemana = dataAtual.getDay(); // 0 = Domingo, 6 = Sábado
                if (diaDaSemana === 0 || diaDaSemana === 6) {
                    diaEl.classList.add('weekend');
                } else {
                    diaEl.classList.add('weekday');
                }


                const dataCompletaFormatada = `${ano}-${String(mes + 1).padStart(2, '0')}-${String(dia).padStart(2, '0')}`;

                const feriado = feriadosNacionais2025.find(f => f.date === dataCompletaFormatada);
                if (feriado) {
                    diaEl.classList.add('holiday');
                    const tooltip = document.createElement('span');
                    tooltip.className = 'tooltip';
                    tooltip.textContent = feriado.name;
                    diaEl.appendChild(tooltip);
                }

                const evento = eventosInternos.find(e => e.date === dataCompletaFormatada);
                if (evento) {
                    diaEl.classList.add(`event-${evento.type.replace(/\s+/g, '_').toLowerCase()}`);
                    const eventMarker = document.createElement('span');
                    eventMarker.className = 'event-marker';
                    eventMarker.textContent = evento.description.substring(0,10) + (evento.description.length > 10 ? '...' : ''); 
                    
                    if (!feriado) { 
                        const tooltip = document.createElement('span');
                        tooltip.className = 'tooltip';
                        tooltip.textContent = evento.description;
                        diaEl.appendChild(tooltip);
                    } else { 
                         diaEl.querySelector('.tooltip').textContent += ` | ${evento.description}`;
                    }
                     diaEl.appendChild(eventMarker);
                }
                calendarGrid.appendChild(diaEl);
            }
        }

        prevMonthBtn.addEventListener('click', () => {
            currentCalendarDate.setMonth(currentCalendarDate.getMonth() - 1);
            renderizarCalendario();
        });

        nextMonthBtn.addEventListener('click', () => {
            currentCalendarDate.setMonth(currentCalendarDate.getMonth() + 1);
            renderizarCalendario();
        });


        document.addEventListener('DOMContentLoaded', inicializarApp);
    </script>
</body>
</html>
