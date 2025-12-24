<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SilicaManager Pro - Gestão de Perdas</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React e Babel -->
    <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- XLSX Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        .animate-in { animation: fadeIn 0.4s ease-out forwards; }
        #root { min-height: 100vh; }
        .loss-gradient { background: linear-gradient(135deg, #fee2e2 0%, #fef2f2 100%); }
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 18px;
            height: 18px;
            background: #059669;
            cursor: pointer;
            border-radius: 50%;
        }
    </style>
</head>
<body class="bg-gray-50 text-slate-900">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useMemo, useRef } = React;

        // Componente de Ícone corrigido para evitar erro de removeChild do React
        const Icon = ({ name, size = 20, className = "" }) => {
            const iconRef = useRef(null);

            useEffect(() => {
                if (window.lucide && iconRef.current) {
                    window.lucide.createIcons({
                        attrs: {
                            strokeWidth: 2,
                            width: size,
                            height: size,
                            class: className
                        },
                        nameAttr: 'data-lucide'
                    });
                }
            }, [name, size, className]);

            return (
                <span className="inline-flex items-center justify-center shrink-0">
                    <i data-lucide={name} ref={iconRef}></i>
                </span>
            );
        };

        const Card = ({ children, className = "" }) => (
            <div className={"bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden " + className}>{children}</div>
        );

        function App() {
            const [productionTarget, setProductionTarget] = useState(100);
            const [lossPercentage, setLossPercentage] = useState(5); 
            const [activeTab, setActiveTab] = useState('dashboard');
            const [showSaveAlert, setShowSaveAlert] = useState(false);
            
            const [recipe, setRecipe] = useState([
                { id: 1, name: 'Silicato de Sódio', quantityPerTon: 2.5, unit: 'Ton', costPerUnit: 800 },
                { id: 2, name: 'Ácido Sulfúrico', quantityPerTon: 0.35, unit: 'Ton', costPerUnit: 1200 },
                { id: 3, name: 'Água Industrial', quantityPerTon: 15, unit: 'm³', costPerUnit: 15 },
                { id: 4, name: 'Energia Elétrica', quantityPerTon: 250, unit: 'kWh', costPerUnit: 0.85 },
                { id: 7, name: 'Lenha (Caldeira)', quantityPerTon: 0.5, unit: 'Ton', costPerUnit: 220 },
                { id: 5, name: 'Gás Natural', quantityPerTon: 50, unit: 'm³', costPerUnit: 4.50 },
                { id: 6, name: 'Embalagens (Sacos 25kg)', quantityPerTon: 40, unit: 'Unid', costPerUnit: 2.50 },
            ]);

            const baseCost = useMemo(() => recipe.reduce((acc, item) => acc + (item.quantityPerTon * productionTarget * item.costPerUnit), 0), [recipe, productionTarget]);
            const lossValue = useMemo(() => baseCost * (lossPercentage / 100), [baseCost, lossPercentage]);
            const totalWithLoss = baseCost + lossValue;
            const costPerTon = totalWithLoss / (productionTarget || 1);

            const exportToExcel = () => {
                const now = new Date();
                const rows = [
                    ["RELATÓRIO DE CUSTOS E PERDAS - SÍLICA"],
                    ["Data", now.toLocaleDateString('pt-BR')],
                    ["Mês", now.toLocaleString('pt-BR', { month: 'long' }).toUpperCase()],
                    ["Produção", productionTarget + " Ton"],
                    ["Índice de Perda", lossPercentage + "%"],
                    [],
                    ["INSUMO", "QTD/TON", "UNID", "PREÇO UNIT", "CUSTO TOTAL", "VALOR PERDIDO (R$)"]
                ];

                recipe.forEach(item => {
                    const totalQty = item.quantityPerTon * productionTarget;
                    const itemCost = totalQty * item.costPerUnit;
                    const itemLoss = itemCost * (lossPercentage / 100);
                    rows.push([item.name, item.quantityPerTon, item.unit, item.costPerUnit, itemCost, itemLoss]);
                });

                rows.push([], ["RESUMO GERAL"]);
                rows.push(["Custo Base", baseCost]);
                rows.push(["Prejuízo Estimado (Perdas)", lossValue]);
                rows.push(["Custo Final com Perdas", totalWithLoss]);

                const ws = XLSX.utils.aoa_to_sheet(rows);
                const wb = XLSX.utils.book_new();
                XLSX.utils.book_append_sheet(wb, ws, "Relatório Perdas");
                XLSX.writeFile(wb, `Gestao_Perdas_Silica_${now.getDate()}_${now.getMonth()+1}.xlsx`);
            };

            const updateRecipeItem = (id, field, value) => {
                const newRecipe = recipe.map(item => 
                    item.id === id ? { ...item, [field]: field === 'unit' ? value : (parseFloat(value) || 0) } : item
                );
                setRecipe(newRecipe);
            };

            const formatCurrency = (val) => new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(val);

            return (
                <div className="min-h-screen flex flex-col">
                    <header className="bg-slate-900 text-white shadow-xl shrink-0 border-b border-blue-500/30">
                        <div className="container mx-auto px-4 py-5">
                            <div className="flex flex-col md:flex-row justify-between items-center gap-6">
                                <div className="flex items-center gap-4">
                                    <div className="bg-blue-600 p-2.5 rounded-2xl shadow-lg shadow-blue-500/20">
                                        <Icon name="activity" size={28} />
                                    </div>
                                    <div>
                                        <h1 className="text-xl font-black tracking-tighter uppercase">SilicaManager <span className="text-blue-500">Pro</span></h1>
                                        <div className="flex items-center gap-2 text-slate-400 text-[10px] font-bold uppercase tracking-widest">
                                            <Icon name="shield-check" size={12} className="text-emerald-500" />
                                            Eficiência Ativa
                                        </div>
                                    </div>
                                </div>
                                
                                <div className="flex bg-slate-800 rounded-2xl p-1.5 border border-slate-700">
                                    <button onClick={() => setActiveTab('dashboard')} className={`px-6 py-2 rounded-xl text-sm transition-all ${activeTab === 'dashboard' ? 'bg-blue-600 text-white font-bold shadow-lg shadow-blue-900/40' : 'text-slate-400 hover:text-white'}`}>Operação</button>
                                    <button onClick={() => setActiveTab('loss')} className={`px-6 py-2 rounded-xl text-sm transition-all flex items-center gap-2 ${activeTab === 'loss' ? 'bg-red-600 text-white font-bold shadow-lg shadow-red-900/40' : 'text-slate-400 hover:text-white'}`}><Icon name="trending-down" size={16}/> Prevenção</button>
                                    <button onClick={() => setActiveTab('settings')} className={`px-6 py-2 rounded-xl text-sm transition-all flex items-center gap-2 ${activeTab === 'settings' ? 'bg-slate-600 text-white font-bold shadow-lg shadow-slate-900/40' : 'text-slate-400 hover:text-white'}`}><Icon name="settings" size={16}/> Configs</button>
                                </div>
                            </div>
                        </div>
                    </header>

                    <main className="container mx-auto px-4 py-8 flex-grow">
                        {activeTab === 'dashboard' && (
                            <div className="space-y-6 animate-in">
                                <div className="grid grid-cols-1 lg:grid-cols-4 gap-6">
                                    <Card className="p-6 border-l-4 border-l-blue-600">
                                        <label className="text-[10px] font-black text-slate-400 uppercase tracking-widest block mb-1">Meta de Produção</label>
                                        <div className="flex items-end gap-2">
                                            <input type="number" value={productionTarget} onChange={(e) => setProductionTarget(e.target.value)} className="text-3xl font-black text-slate-800 bg-transparent outline-none w-full border-b-2 border-slate-100 focus:border-blue-500" />
                                            <span className="font-bold text-slate-300">TON</span>
                                        </div>
                                    </Card>

                                    <Card className="p-6 border-l-4 border-l-red-500 loss-gradient">
                                        <div className="flex justify-between items-start mb-1">
                                            <label className="text-[10px] font-black text-red-600 uppercase tracking-widest">Prejuízo Estimado</label>
                                            <div className="flex items-center gap-1 bg-red-100 text-red-600 px-2 py-0.5 rounded-full text-[10px] font-black">
                                                {lossPercentage}% TAXA
                                            </div>
                                        </div>
                                        <p className="text-3xl font-black text-red-700">{formatCurrency(lossValue)}</p>
                                        <p className="text-[10px] text-red-400 mt-2">Custo da perda por Ton: {formatCurrency(lossValue / (productionTarget || 1))}</p>
                                    </Card>

                                    <Card className="p-6 bg-slate-900 text-white lg:col-span-2">
                                        <div className="flex justify-between items-center mb-4">
                                            <p className="text-[10px] font-black text-slate-400 uppercase tracking-widest italic">Custo Final Estimado</p>
                                            <button onClick={exportToExcel} className="bg-emerald-600 hover:bg-emerald-700 p-2 rounded-lg transition-colors"><Icon name="download" size={18}/></button>
                                        </div>
                                        <div className="flex flex-col sm:flex-row justify-between sm:items-end gap-4">
                                            <p className="text-4xl font-black text-blue-400 tracking-tighter">{formatCurrency(totalWithLoss)}</p>
                                            <div className="text-right">
                                                <p className="text-[10px] text-slate-500 uppercase font-bold">Média p/ Ton</p>
                                                <p className="text-xl font-bold">{formatCurrency(costPerTon)}</p>
                                            </div>
                                        </div>
                                    </Card>
                                </div>

                                <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                                    {recipe.map(item => {
                                        const totalQty = item.quantityPerTon * productionTarget;
                                        const lossQty = totalQty * (lossPercentage / 100);
                                        return (
                                            <Card key={item.id} className="p-5 border-slate-100 hover:border-blue-200 transition-all group">
                                                <div className="flex justify-between mb-4">
                                                    <div className="text-slate-400 group-hover:text-blue-500 transition-colors"><Icon name="package" /></div>
                                                    <div className="text-right">
                                                        <span className="block text-[8px] font-black text-red-400 uppercase tracking-tighter">Perda Est.</span>
                                                        <span className="text-xs font-bold text-red-600">{new Intl.NumberFormat('pt-BR').format(lossQty)} {item.unit}</span>
                                                    </div>
                                                </div>
                                                <h3 className="font-bold text-slate-800 text-sm">{item.name}</h3>
                                                <p className="text-2xl font-black text-slate-900 mt-1">{new Intl.NumberFormat('pt-BR').format(totalQty)} <span className="text-xs font-medium text-slate-400">{item.unit}</span></p>
                                                <div className="mt-4 pt-3 border-t border-slate-50 flex justify-between items-center">
                                                    <span className="text-[9px] font-bold text-slate-300 uppercase italic">R$ Perda:</span>
                                                    <span className="text-xs font-bold text-red-400">{formatCurrency(lossQty * item.costPerUnit)}</span>
                                                </div>
                                            </Card>
                                        );
                                    })}
                                </div>
                            </div>
                        )}

                        {activeTab === 'loss' && (
                            <div className="max-w-6xl mx-auto space-y-6 animate-in">
                                <Card className="p-8 border-l-8 border-l-emerald-500">
                                    <div className="flex flex-col md:flex-row gap-8">
                                        <div className="md:w-1/3">
                                            <h2 className="text-2xl font-black text-slate-800 mb-4">Prevenção de Perdas</h2>
                                            <p className="text-slate-500 text-sm leading-relaxed mb-6">
                                                A produção de sílica precipitada exige controle rigoroso. Reduzir sua taxa de <span className="text-red-600 font-bold">{lossPercentage}%</span> para <span className="text-emerald-600 font-bold">2%</span> gera economia imediata.
                                            </p>
                                            <div className="bg-emerald-50 p-4 rounded-xl border border-emerald-100">
                                                <h4 className="text-emerald-700 font-bold text-xs uppercase mb-2">Ajustar Cenário de Perda</h4>
                                                <input type="range" min="0" max="15" value={lossPercentage} onChange={(e)=>setLossPercentage(e.target.value)} className="w-full h-2 bg-emerald-200 rounded-lg appearance-none cursor-pointer mt-2" />
                                                <div className="flex justify-between mt-1 text-[10px] font-bold text-emerald-700 uppercase"><span>0%</span><span>Meta: {lossPercentage}%</span><span>15%</span></div>
                                            </div>
                                        </div>

                                        <div className="md:w-2/3 grid grid-cols-1 sm:grid-cols-2 gap-4">
                                            <div className="p-4 bg-slate-50 rounded-xl border border-slate-100">
                                                <div className="bg-blue-100 text-blue-600 w-8 h-8 rounded-lg flex items-center justify-center mb-3"><Icon name="thermometer" size={16}/></div>
                                                <h4 className="font-bold text-sm text-slate-800">Temperatura Estável</h4>
                                                <p className="text-xs text-slate-500 mt-2">Oscilações térmicas no reator afetam a viscosidade, resultando em sílica que não filtra corretamente.</p>
                                            </div>
                                            <div className="p-4 bg-slate-50 rounded-xl border border-slate-100">
                                                <div className="bg-orange-100 text-orange-600 w-8 h-8 rounded-lg flex items-center justify-center mb-3"><Icon name="droplets" size={16}/></div>
                                                <h4 className="font-bold text-sm text-slate-800">Filtro-Prensa</h4>
                                                <p className="text-xs text-slate-500 mt-2">Vazamentos laterais são a maior causa de perda de massa. Verifique o fechamento hidráulico.</p>
                                            </div>
                                            <div className="p-4 bg-slate-50 rounded-xl border border-slate-100">
                                                <div className="bg-red-100 text-red-600 w-8 h-8 rounded-lg flex items-center justify-center mb-3"><Icon name="flame" size={16}/></div>
                                                <h4 className="font-bold text-sm text-slate-800">Spray Dryer</h4>
                                                <p className="text-xs text-slate-500 mt-2">Calibre o exaustor para evitar que o "pó fino" de alta qualidade seja sugado para fora do silo.</p>
                                            </div>
                                            <div className="p-4 bg-slate-50 rounded-xl border border-slate-100">
                                                <div className="bg-purple-100 text-purple-600 w-8 h-8 rounded-lg flex items-center justify-center mb-3"><Icon name="package-check" size={16}/></div>
                                                <h4 className="font-bold text-sm text-slate-800">Pesagem Precisa</h4>
                                                <p className="text-xs text-slate-500 mt-2">Erros na dosagem de Ácido Sulfúrico desperdiçam Silicato que não reagiu.</p>
                                            </div>
                                        </div>
                                    </div>
                                </Card>
                            </div>
                        )}

                        {activeTab === 'settings' && (
                            <div className="max-w-5xl mx-auto animate-in">
                                <Card className="overflow-hidden border-0 shadow-xl">
                                    <div className="p-6 bg-slate-900 text-white flex justify-between items-center">
                                        <div>
                                            <h3 className="font-bold">Configuração da Receita</h3>
                                            <p className="text-[10px] text-slate-400 uppercase tracking-widest italic">Base para processamento de 1 tonelada</p>
                                        </div>
                                    </div>
                                    <div className="p-4 overflow-x-auto">
                                        <table className="w-full min-w-[600px]">
                                            <thead className="text-[10px] text-slate-400 uppercase font-black border-b border-slate-100">
                                                <tr>
                                                    <th className="p-4 text-left">Insumo</th>
                                                    <th className="p-4 text-center">Consumo (/Ton)</th>
                                                    <th className="p-4 text-center">Unidade</th>
                                                    <th className="p-4 text-center">Preço Unit. (R$)</th>
                                                </tr>
                                            </thead>
                                            <tbody className="divide-y divide-slate-50">
                                                {recipe.map(item => (
                                                    <tr key={item.id} className="hover:bg-slate-50/50 transition-colors">
                                                        <td className="p-4 font-bold text-slate-700 text-sm">{item.name}</td>
                                                        <td className="p-4 text-center">
                                                            <input type="number" value={item.quantityPerTon} onChange={(e) => updateRecipeItem(item.id, 'quantityPerTon', e.target.value)} className="w-24 border-2 border-slate-100 rounded-lg p-2 text-center font-bold text-blue-600 outline-none" />
                                                        </td>
                                                        <td className="p-4 text-center">
                                                            <input type="text" value={item.unit} onChange={(e) => updateRecipeItem(item.id, 'unit', e.target.value)} className="w-20 border-2 border-slate-100 rounded-lg p-2 text-center font-bold text-slate-400 outline-none uppercase text-[10px]" />
                                                        </td>
                                                        <td className="p-4 text-center">
                                                            <input type="number" value={item.costPerUnit} onChange={(e) => updateRecipeItem(item.id, 'costPerUnit', e.target.value)} className="w-32 border-2 border-slate-100 rounded-lg p-2 text-center font-bold text-slate-600 outline-none" />
                                                        </td>
                                                    </tr>
                                                ))}
                                            </tbody>
                                        </table>
                                    </div>
                                    <div className="p-6 bg-slate-50 border-t flex justify-end">
                                        <button onClick={() => { setActiveTab('dashboard'); setShowSaveAlert(true); setTimeout(()=>setShowSaveAlert(false), 3000); }} className="bg-blue-600 hover:bg-blue-700 text-white px-10 py-3 rounded-xl font-bold shadow-lg shadow-blue-200 transition-all active:scale-95">Salvar Alterações</button>
                                    </div>
                                </Card>
                            </div>
                        )}
                    </main>

                    {showSaveAlert && (
                        <div className="fixed bottom-8 right-8 bg-slate-900 text-white px-8 py-4 rounded-2xl shadow-2xl flex items-center gap-3 animate-in border-b-4 border-b-blue-500 z-50">
                            <Icon name="check-circle" className="text-blue-500" />
                            <span className="font-bold text-sm uppercase tracking-widest">Sincronizado com Sucesso</span>
                        </div>
                    )}

                    <footer className="py-6 border-t border-slate-100 text-center bg-white mt-auto">
                        <p className="text-[10px] text-slate-400 font-bold uppercase tracking-[0.2em]">© 2024 SilicaManager Pro • Prevenção e Controle</p>
                    </footer>
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
