import React, { useState } from 'react';
import { Factory, Settings, Save, RotateCcw, TrendingUp, Package, AlertTriangle } from 'lucide-react';

/**
 * Componente de Cartão Simples
 * Utilizado para manter a consistência visual em toda a aplicação.
 */
const Card = ({ children, className = "" }) => (
  <div className={"bg-white rounded-lg shadow-md border border-gray-200 " + className}>
    {children}
  </div>
);

/**
 * Aplicação Principal: SilicaManager Pro
 * Gere o cálculo de insumos e custos para a produção de sílica.
 */
export default function App() {
  // Estado para a meta de produção mensal (em Toneladas)
  const [productionTarget, setProductionTarget] = useState(100);
  
  // Lista de insumos e custos base para a produção de 1 tonelada de sílica
  const [recipe, setRecipe] = useState([
    { id: 1, name: 'Silicato de Sódio', quantityPerTon: 2.5, unit: 'Ton', costPerUnit: 800 },
    { id: 2, name: 'Ácido Sulfúrico', quantityPerTon: 0.35, unit: 'Ton', costPerUnit: 1200 },
    { id: 3, name: 'Água Industrial', quantityPerTon: 15, unit: 'm³', costPerUnit: 15 },
    { id: 4, name: 'Energia Elétrica', quantityPerTon: 250, unit: 'kWh', costPerUnit: 0.85 },
    { id: 7, name: 'Lenha (Caldeira)', quantityPerTon: 0.5, unit: 'Ton', costPerUnit: 220 },
    { id: 5, name: 'Gás Natural / Combustível', quantityPerTon: 50, unit: 'm³', costPerUnit: 4.50 },
    { id: 6, name: 'Embalagens (Sacos 25kg)', quantityPerTon: 40, unit: 'Unid', costPerUnit: 2.50 },
  ]);

  const [activeTab, setActiveTab] = useState('dashboard');
  const [showSaveAlert, setShowSaveAlert] = useState(false);

  // Cálculo do custo total e custo por tonelada
  const totalCost = recipe.reduce((acc, item) => {
    return acc + (item.quantityPerTon * productionTarget * item.costPerUnit);
  }, 0);

  const costPerTon = totalCost / (productionTarget || 1);

  // Atualiza um valor específico na receita (quantidade ou custo unitário)
  const updateRecipeItem = (id, field, value) => {
    const newRecipe = recipe.map(item => {
      if (item.id === id) {
        return { ...item, [field]: parseFloat(value) || 0 };
      }
      return item;
    });
    setRecipe(newRecipe);
  };

  // Restaura os valores de fábrica da receita
  const resetRecipe = () => {
    if(confirm("Deseja restaurar os valores padrão da receita?")) {
      setRecipe([
        { id: 1, name: 'Silicato de Sódio', quantityPerTon: 2.5, unit: 'Ton', costPerUnit: 800 },
        { id: 2, name: 'Ácido Sulfúrico', quantityPerTon: 0.35, unit: 'Ton', costPerUnit: 1200 },
        { id: 3, name: 'Água Industrial', quantityPerTon: 15, unit: 'm³', costPerUnit: 15 },
        { id: 4, name: 'Energia Elétrica', quantityPerTon: 250, unit: 'kWh', costPerUnit: 0.85 },
        { id: 7, name: 'Lenha (Caldeira)', quantityPerTon: 0.5, unit: 'Ton', costPerUnit: 220 },
        { id: 5, name: 'Gás Natural / Combustível', quantityPerTon: 50, unit: 'm³', costPerUnit: 4.50 },
        { id: 6, name: 'Embalagens (Sacos 25kg)', quantityPerTon: 40, unit: 'Unid', costPerUnit: 2.50 },
      ]);
    }
  };

  // Funções de formatação para exibição
  const formatCurrency = (value) => {
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
  };

  const formatNumber = (value) => {
    return new Intl.NumberFormat('pt-BR', { maximumFractionDigits: 2 }).format(value);
  };

  return (
    <div className="min-h-screen bg-gray-50 font-sans text-gray-800">
      {/* Cabeçalho da Aplicação */}
      <header className="bg-blue-800 text-white shadow-lg">
        <div className="container mx-auto px-4 py-6">
          <div className="flex flex-col md:flex-row justify-between items-center gap-4">
            <div className="flex items-center gap-3">
              <div className="p-2 bg-white/10 rounded-lg">
                <Factory size={32} />
              </div>
              <div>
                <h1 className="text-2xl font-bold">SilicaManager Pro</h1>
                <p className="text-blue-200 text-sm">Gestão de Produção Mensal</p>
              </div>
            </div>
            
            {/* Navegação entre Painel e Configurações */}
            <div className="flex bg-blue-900/50 rounded-lg p-1">
              <button 
                onClick={() => setActiveTab('dashboard')}
                className={"px-4 py-2 rounded-md transition-all " + (activeTab === 'dashboard' ? 'bg-white text-blue-900 font-bold shadow' : 'text-blue-100 hover:bg-blue-800')}
              >
                Cálculos
              </button>
              <button 
                onClick={() => setActiveTab('settings')}
                className={"px-4 py-2 rounded-md transition-all flex items-center gap-2 " + (activeTab === 'settings' ? 'bg-white text-blue-900 font-bold shadow' : 'text-blue-100 hover:bg-blue-800')}
              >
                <Settings size={18} /> Receita
              </button>
            </div>
          </div>
        </div>
      </header>

      <main className="container mx-auto px-4 py-8">
        {activeTab === 'dashboard' && (
          <div className="space-y-8 animate-in fade-in duration-500">
            {/* Secção de Input de Meta e Resumo de Custos */}
            <div className="bg-white rounded-xl shadow-md p-6 border-l-4 border-blue-600">
              <div className="flex flex-col md:flex-row items-center justify-between gap-6">
                <div className="flex-1 w-full">
                  <label className="block text-sm font-semibold text-gray-600 mb-2 uppercase tracking-wide">
                    Meta de Produção (Toneladas de Sílica Final)
                  </label>
                  <div className="relative">
                    <input 
                      type="number" 
                      value={productionTarget}
                      onChange={(e) => setProductionTarget(Math.max(0, parseFloat(e.target.value) || 0))}
                      className="w-full pl-4 pr-16 py-3 text-3xl font-bold text-blue-900 border border-gray-300 rounded-lg focus:ring-4 focus:ring-blue-100 outline-none transition-all"
                    />
                    <span className="absolute right-4 top-1/2 -translate-y-1/2 text-gray-400 font-medium text-lg">Ton</span>
                  </div>
                </div>
                
                <div className="flex flex-col sm:flex-row gap-4 w-full md:w-auto">
                  <div className="bg-green-50 px-6 py-3 rounded-lg border border-green-100 min-w-[200px]">
                    <p className="text-xs text-green-600 font-bold uppercase mb-1">Custo Total Previsto</p>
                    <p className="text-2xl font-bold text-green-800">{formatCurrency(totalCost)}</p>
                  </div>
                  <div className="bg-blue-50 px-6 py-3 rounded-lg border border-blue-100 min-w-[200px]">
                    <p className="text-xs text-blue-600 font-bold uppercase mb-1">Custo por Tonelada</p>
                    <p className="text-2xl font-bold text-blue-800">{formatCurrency(costPerTon)}</p>
                  </div>
                </div>
              </div>
            </div>

            {/* Grid de Cartões de Insumos */}
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {recipe.map((item) => {
                const totalQuantity = item.quantityPerTon * productionTarget;
                const itemTotalCost = totalQuantity * item.costPerUnit;
                
                return (
                  <Card key={item.id} className="p-5 hover:shadow-lg transition-shadow border-t-2 border-t-transparent hover:border-t-blue-500">
                    <div className="flex justify-between items-start mb-4">
                      <div className="p-2 bg-blue-100 text-blue-600 rounded-lg">
                        <Package size={20} />
                      </div>
                      <span className="text-xs font-bold text-gray-500 bg-gray-100 px-2 py-1 rounded">
                        {item.quantityPerTon + " " + item.unit + "/ton"}
                      </span>
                    </div>
                    <h3 className="text-lg font-bold text-gray-800">{item.name}</h3>
                    <div className="mt-2">
                      <p className="text-xs text-gray-400 uppercase font-semibold">Necessidade Total:</p>
                      <p className="text-2xl font-bold text-blue-900">
                        {formatNumber(totalQuantity) + " "} 
                        <span className="text-sm font-normal text-gray-500">{item.unit}</span>
                      </p>
                    </div>
                    <div className="mt-4 pt-3 border-t border-gray-100 flex justify-between items-center">
                      <span className="text-sm text-gray-500 font-medium">Custo Insumo:</span>
                      <span className="font-bold text-gray-700">{formatCurrency(itemTotalCost)}</span>
                    </div>
                  </Card>
                );
              })}
            </div>

            {/* Secção de Distribuição Visual (Gráfico de Barras Simples) */}
            <Card className="p-6">
              <h3 className="text-lg font-bold text-gray-800 mb-6 flex items-center gap-2">
                <TrendingUp size={20} className="text-blue-600" /> 
                Impacto Financeiro por Insumo (%)
              </h3>
              <div className="space-y-5">
                {recipe
                  .sort((a, b) => (b.quantityPerTon * b.costPerUnit) - (a.quantityPerTon * a.costPerUnit))
                  .map(item => {
                    const itemCost = item.quantityPerTon * productionTarget * item.costPerUnit;
                    const percent = totalCost > 0 ? (itemCost / totalCost) * 100 : 0;
                    
                    return (
                      <div key={item.id}>
                        <div className="flex justify-between text-sm mb-2">
                          <span className="font-semibold text-gray-700">{item.name}</span>
                          <span className="text-blue-700 font-bold">{percent.toFixed(1) + "%"}</span>
                        </div>
                        <div className="w-full bg-gray-100 rounded-full h-2.5 overflow-hidden">
                          <div 
                            className="bg-blue-600 h-2.5 rounded-full transition-all duration-700 ease-out"
                            style={{ width: percent + '%' }}
                          ></div>
                        </div>
                      </div>
                    )
                  })}
              </div>
            </Card>
          </div>
        )}

        {/* --- ABA DE CONFIGURAÇÕES (RECEITA) --- */}
        {activeTab === 'settings' && (
          <div className="max-w-4xl mx-auto animate-in slide-in-from-bottom-4 duration-500">
            <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-4 mb-6 flex gap-3 items-center">
              <AlertTriangle className="text-yellow-600 shrink-0" size={24} />
              <p className="text-sm text-yellow-800 font-medium">
                Ajuste os consumos médios e os preços unitários abaixo para refletir a realidade atual da fábrica.
              </p>
            </div>

            <Card className="overflow-hidden">
              <div className="p-6 bg-gray-50 flex justify-between items-center border-b">
                <h3 className="font-bold text-lg text-gray-800">Parâmetros de Insumos</h3>
                <button 
                  onClick={resetRecipe} 
                  className="text-xs text-red-600 hover:text-red-800 flex items-center gap-1 font-bold transition-colors"
                >
                  <RotateCcw size={14} /> Restaurar Padrões
                </button>
              </div>
              
              <div className="overflow-x-auto">
                <table className="w-full text-left">
                  <thead className="bg-gray-100 text-xs uppercase text-gray-500">
                    <tr>
                      <th className="p-4">Material / Insumo</th>
                      <th className="p-4 text-center">Consumo p/ Ton</th>
                      <th className="p-4 text-center">Preço Unitário (R$)</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-gray-100">
                    {recipe.map((item) => (
                      <tr key={item.id} className="hover:bg-blue-50/30 transition-colors">
                        <td className="p-4">
                          <span className="font-bold text-gray-700">{item.name}</span>
                          <div className="text-[10px] text-gray-400 uppercase tracking-tighter">Medido em {item.unit}</div>
                        </td>
                        <td className="p-4">
                          <div className="flex justify-center items-center gap-2">
                            <input 
                              type="number" 
                              step="0.01"
                              value={item.quantityPerTon}
                              onChange={(e) => updateRecipeItem(item.id, 'quantityPerTon', e.target.value)}
                              className="w-24 p-2 border border-gray-300 rounded text-center font-semibold focus:ring-2 focus:ring-blue-500 outline-none"
                            />
                            <span className="text-xs text-gray-400 w-8">{item.unit}</span>
                          </div>
                        </td>
                        <td className="p-4 text-center">
                          <div className="flex justify-center items-center gap-1">
                            <span className="text-gray-400 text-sm">R$</span>
                            <input 
                              type="number" 
                              step="0.01"
                              value={item.costPerUnit}
                              onChange={(e) => updateRecipeItem(item.id, 'costPerUnit', e.target.value)}
                              className="w-28 p-2 border border-gray-300 rounded text-center font-semibold focus:ring-2 focus:ring-blue-500 outline-none"
                            />
                          </div>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
              
              <div className="p-6 bg-gray-50 flex justify-end">
                <button 
                  onClick={() => {
                    setActiveTab('dashboard'); 
                    setShowSaveAlert(true); 
                    setTimeout(() => setShowSaveAlert(false), 3000);
                  }} 
                  className="bg-blue-600 hover:bg-blue-700 text-white px-8 py-3 rounded-lg font-bold shadow-lg flex items-center gap-2 transition-all transform active:scale-95"
                >
                  <Save size={20} />
                  Salvar e Atualizar Painel
                </button>
              </div>
            </Card>
          </div>
        )}
      </main>

      {/* Alerta de Sucesso (Toast) */}
      {showSaveAlert && (
        <div className="fixed bottom-6 right-6 bg-green-600 text-white px-8 py-4 rounded-xl shadow-2xl animate-in slide-in-from-right-10 duration-300 flex items-center gap-3">
          <div className="bg-white/20 p-1 rounded-full">
            <Save size={18} />
          </div>
          <span className="font-bold">Receita e Custos atualizados com sucesso!</span>
        </div>
      )}
    </div>
  );
}
