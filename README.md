import React, { useState, useEffect } from 'react';
import { Factory, Calculator, DollarSign, Settings, Save, RotateCcw, TrendingUp, Package, AlertTriangle } from 'lucide-react';

// Componente de Cartão Simples
const Card = ({ children, className = "" }) => (
  <div className={`bg-white rounded-lg shadow-md border border-gray-200 ${className}`}>
    {children}
  </div>
);

// Componente Principal
export default function App() {
  // Estado para a meta de produção (em Toneladas)
  const [productionTarget, setProductionTarget] = useState(100);
  
  // Estado para a "Receita" (Insumos necessários para produzir 1 Tonelada de Sílica)
  // Valores iniciais hipotéticos para Sílica Precipitada (Exemplo)
  const [recipe, setRecipe] = useState([
    { id: 1, name: 'Silicato de Sódio', quantityPerTon: 2.5, unit: 'Ton', costPerUnit: 800 },
    { id: 2, name: 'Ácido Sulfúrico', quantityPerTon: 0.35, unit: 'Ton', costPerUnit: 1200 },
    { id: 3, name: 'Água Industrial', quantityPerTon: 15, unit: 'm³', costPerUnit: 15 },
    { id: 4, name: 'Energia Elétrica', quantityPerTon: 250, unit: 'kWh', costPerUnit: 0.85 },
    { id: 7, name: 'Lenha (Caldeira)', quantityPerTon: 0.5, unit: 'Ton', costPerUnit: 220 },
    { id: 5, name: 'Gás Natural / Combustível', quantityPerTon: 50, unit: 'm³', costPerUnit: 4.50 },
    { id: 6, name: 'Embalagens (Sacos 25kg)', quantityPerTon: 40, unit: 'Unid', costPerUnit: 2.50 },
  ]);

  const [activeTab, setActiveTab] = useState('dashboard'); // 'dashboard' ou 'settings'
  const [showSaveAlert, setShowSaveAlert] = useState(false);

  // Totais calculados
  const totalCost = recipe.reduce((acc, item) => {
    return acc + (item.quantityPerTon * productionTarget * item.costPerUnit);
  }, 0);

  const costPerTon = totalCost / (productionTarget || 1);

  // Função para atualizar um item da receita
  const updateRecipeItem = (id, field, value) => {
    const newRecipe = recipe.map(item => {
      if (item.id === id) {
        return { ...item, [field]: parseFloat(value) || 0 };
      }
      return item;
    });
    setRecipe(newRecipe);
  };

  // Resetar para padrão
  const resetRecipe = () => {
    if(confirm("Tem certeza que deseja restaurar os valores padrão?")) {
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

  // Formatação de Moeda
  const formatCurrency = (value) => {
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
  };

  // Formatação de Números
  const formatNumber = (value) => {
    return new Intl.NumberFormat('pt-BR', { maximumFractionDigits: 2 }).format(value);
  };

  return (
    <div className="min-h-screen bg-gray-50 font-sans text-gray-800">
      {/* Header */}
      <header className="bg-blue-800 text-white shadow-lg">
        <div className="container mx-auto px-4 py-6">
          <div className="flex flex-col md:flex-row justify-between items-center gap-4">
            <div className="flex items-center gap-3">
              <div className="p-2 bg-white/10 rounded-lg">
                <Factory size={32} />
              </div>
              <div>
                <h1 className="text-2xl font-bold">SilicaManager Pro</h1>
                <p className="text-blue-200 text-sm">Sistema de Planejamento de Produção de Sílica</p>
              </div>
            </div>
            
            <div className="flex bg-blue-900/50 rounded-lg p-1">
              <button 
                onClick={() => setActiveTab('dashboard')}
                className={`px-4 py-2 rounded-md transition-all ${activeTab === 'dashboard' ? 'bg-white text-blue-900 font-bold shadow' : 'text-blue-100 hover:bg-blue-800'}`}
              >
                Painel de Cálculo
              </button>
              <button 
                onClick={() => setActiveTab('settings')}
                className={`px-4 py-2 rounded-md transition-all flex items-center gap-2 ${activeTab === 'settings' ? 'bg-white text-blue-900 font-bold shadow' : 'text-blue-100 hover:bg-blue-800'}`}
              >
                <Settings size={18} /> Configurar Receita
              </button>
            </div>
          </div>
        </div>
      </header>

      <main className="container mx-auto px-4 py-8">
        
        {/* --- ABA DASHBOARD --- */}
        {activeTab === 'dashboard' && (
          <div className="space-y-8">
            
            {/* Input de Meta Principal */}
            <div className="bg-white rounded-xl shadow-md p-6 border-l-4 border-blue-600">
              <div className="flex flex-col md:flex-row items-center justify-between gap-6">
                <div className="flex-1 w-full">
                  <label className="block text-sm font-semibold text-gray-600 mb-2 uppercase tracking-wide">
                    Meta de Produção Mensal (Sílica Final)
                  </label>
                  <div className="relative">
                    <input 
                      type="number" 
                      value={productionTarget}
                      onChange={(e) => setProductionTarget(Math.max(0, parseFloat(e.target.value) || 0))}
                      className="w-full pl-4 pr-16 py-3 text-3xl font-bold text-blue-900 border border-gray-300 rounded-lg focus:ring-4 focus:ring-blue-100 focus:border-blue-500 outline-none transition-all"
                    />
                    <span className="absolute right-4 top-1/2 -translate-y-1/2 text-gray-400 font-medium">Toneladas</span>
                  </div>
                </div>
                
                <div className="flex gap-4 w-full md:w-auto">
                  <div className="bg-green-50 px-6 py-3 rounded-lg border border-green-100 flex-1 md:flex-none">
                    <p className="text-xs text-green-600 font-bold uppercase mb-1">Custo Estimado Total</p>
                    <p className="text-xl font-bold text-green-800">{formatCurrency(totalCost)}</p>
                  </div>
                  <div className="bg-blue-50 px-6 py-3 rounded-lg border border-blue-100 flex-1 md:flex-none">
                    <p className="text-xs text-blue-600 font-bold uppercase mb-1">Custo por Tonelada</p>
                    <p className="text-xl font-bold text-blue-800">{formatCurrency(costPerTon)}</p>
                  </div>
                </div>
              </div>
            </div>

            {/* Grid de Resultados */}
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {recipe.map((item) => {
                const totalQuantity = item.quantityPerTon * productionTarget;
                const totalItemCost = totalQuantity * item.costPerUnit;
                
                return (
                  <Card key={item.id} className="hover:shadow-lg transition-shadow p-5 relative overflow-hidden group">
                    <div className="absolute top-0 right-0 w-24 h-24 bg-blue-50 rounded-bl-full -mr-8 -mt-8 opacity-50 group-hover:scale-110 transition-transform"></div>
                    
                    <div className="relative z-10">
                      <div className="flex justify-between items-start mb-4">
                        <div className="p-2 bg-blue-100 text-blue-600 rounded-lg">
                          <Package size={20} />
                        </div>
                        <span className="text-xs font-bold bg-gray-100 text-gray-500 px-2 py-1 rounded">
                          {item.quantityPerTon} {item.unit}/ton
                        </span>
                      </div>
                      
                      <h3 className="text-lg font-bold text-gray-800 mb-1">{item.name}</h3>
                      <div className="my-3">
                        <p className="text-sm text-gray-500">Necessidade Mensal:</p>
                        <p className="text-2xl font-bold text-blue-900">
                          {formatNumber(totalQuantity)} <span className="text-base font-normal text-gray-500">{item.unit}</span>
                        </p>
                      </div>
                      
                      <div className="pt-3 border-t border-gray-100 flex justify-between items-center">
                        <span className="text-sm text-gray-500">Custo prev:</span>
                        <span className="font-bold text-gray-700">{formatCurrency(totalItemCost)}</span>
                      </div>
                    </div>
                  </Card>
                );
              })}
            </div>

            {/* Gráfico Visual Simplificado (Barra de Custo) */}
            <Card className="p-6">
              <h3 className="text-lg font-bold text-gray-800 mb-4 flex items-center gap-2">
                <TrendingUp size={20} /> Distribuição de Custos
              </h3>
              <div className="space-y-4">
                {recipe.sort((a,b) => (b.quantityPerTon * b.costPerUnit) - (a.quantityPerTon * a.costPerUnit)).map(item => {
                  const itemCost = item.quantityPerTon * productionTarget * item.costPerUnit;
                  const percent = totalCost > 0 ? (itemCost / totalCost) * 100 : 0;
                  
                  return (
                    <div key={item.id}>
                      <div className="flex justify-between text-sm mb-1">
                        <span className="font-medium text-gray-700">{item.name}</span>
                        <span className="text-gray-500">{percent.toFixed(1)}%</span>
                      </div>
                      <div className="w-full bg-gray-100 rounded-full h-3 overflow-hidden">
                        <div 
                          className="bg-blue-600 h-3 rounded-full transition-all duration-500"
                          style={{ width: `${percent}%` }}
                        ></div>
                      </div>
                    </div>
                  )
                })}
              </div>
            </Card>
          </div>
        )}

        {/* --- ABA CONFIGURAÇÕES (RECEITA) --- */}
        {activeTab === 'settings' && (
          <div className="max-w-4xl mx-auto">
            <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-4 mb-6 flex gap-3 items-start">
              <AlertTriangle className="text-yellow-600 shrink-0 mt-1" size={20} />
              <div>
                <h4 className="font-bold text-yellow-800">Calibragem da Fábrica</h4>
                <p className="text-sm text-yellow-700">
                  Ajuste os valores abaixo conforme a eficiência real da sua planta. 
                  O sistema calcula: <strong>Quantidade necessária para produzir 1 Tonelada de Sílica.</strong>
                </p>
              </div>
            </div>

            <Card className="overflow-hidden">
              <div className="p-6 border-b border-gray-200 flex justify-between items-center bg-gray-50">
                <h3 className="font-bold text-lg text-gray-800">Configuração de Insumos (Receita)</h3>
                <button 
                  onClick={resetRecipe}
                  className="text-sm text-red-600 hover:text-red-800 flex items-center gap-1 font-medium"
                >
                  <RotateCcw size={14} /> Restaurar Padrões
                </button>
              </div>
              
              <div className="overflow-x-auto">
                <table className="w-full text-left border-collapse">
                  <thead>
                    <tr className="bg-gray-100 text-gray-600 text-sm uppercase">
                      <th className="p-4 font-semibold">Insumo / Material</th>
                      <th className="p-4 font-semibold">Unidade</th>
                      <th className="p-4 font-semibold w-40">Qtd. por Ton de Produto</th>
                      <th className="p-4 font-semibold w-40">Custo Unitário (R$)</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-gray-100">
                    {recipe.map((item) => (
                      <tr key={item.id} className="hover:bg-blue-50/50 transition-colors">
                        <td className="p-4">
                          <input 
                            type="text" 
                            value={item.name}
                            onChange={(e) => updateRecipeItem(item.id, 'name', e.target.value)}
                            className="w-full bg-transparent font-medium text-gray-800 border-b border-transparent focus:border-blue-500 outline-none"
                          />
                        </td>
                        <td className="p-4 text-gray-500">
                          <input 
                            type="text" 
                            value={item.unit}
                            onChange={(e) => updateRecipeItem(item.id, 'unit', e.target.value)}
                            className="w-20 bg-transparent border-b border-transparent focus:border-blue-500 outline-none"
                          />
                        </td>
                        <td className="p-4">
                          <input 
                            type="number" 
                            step="0.01"
                            value={item.quantityPerTon}
                            onChange={(e) => updateRecipeItem(item.id, 'quantityPerTon', e.target.value)}
                            className="w-full p-2 border border-gray-300 rounded focus:ring-2 focus:ring-blue-200 outline-none"
                          />
                        </td>
                        <td className="p-4">
                          <div className="relative">
                            <span className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 text-xs">R$</span>
                            <input 
                              type="number" 
                              step="0.01"
                              value={item.costPerUnit}
                              onChange={(e) => updateRecipeItem(item.id, 'costPerUnit', e.target.value)}
                              className="w-full pl-8 p-2 border border-gray-300 rounded focus:ring-2 focus:ring-blue-200 outline-none"
                            />
                          </div>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
              
              <div className="p-4 bg-gray-50 text-right">
                <button 
                  onClick={() => {
                    setActiveTab('dashboard');
                    setShowSaveAlert(true);
                    setTimeout(() => setShowSaveAlert(false), 3000);
                  }}
                  className="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2 rounded-lg font-medium shadow transition-colors flex items-center gap-2 ml-auto"
                >
                  <Save size={18} /> Salvar e Voltar
                </button>
              </div>
            </Card>
          </div>
        )}

        {/* Notificação Flutuante */}
        {showSaveAlert && (
          <div className="fixed bottom-4 right-4 bg-green-600 text-white px-6 py-3 rounded-lg shadow-xl flex items-center gap-3 animate-bounce">
            <Save size={20} /> Configurações atualizadas com sucesso!
          </div>
        )}

      </main>
    </div>
  );
}
