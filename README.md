import React, { useState } from 'react';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { LineChart, Line, PieChart, Pie, Cell, BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';
import { Wallet, TrendingUp, CreditCard, PiggyBank, Plus, ArrowUpRight, ArrowDownRight, Filter, Trash2 } from 'lucide-react';

interface Transaction {
  id: string;
  date: string;
  description: string;
  amount: number;
  category: string;
  type: 'expense' | 'income';
}

interface Investment {
  id: string;
  name: string;
  type: string;
  amount: number;
  returns: number;
  allocation: number;
}

export default function FinanceDashboard() {
  const [activeTab, setActiveTab] = useState<'dashboard' | 'transactions' | 'investments' | 'analytics'>('dashboard');
  const [showAddTransaction, setShowAddTransaction] = useState(false);
  const [showAddInvestment, setShowAddInvestment] = useState(false);
  const [showAddInvoice, setShowAddInvoice] = useState(false);
  const [filterCategory, setFilterCategory] = useState<string>('all');
  const [newInvoice, setNewInvoice] = useState({
    month: new Date().toISOString().slice(0, 7),
    amount: ''
  });
  const [monthlyInvoices, setMonthlyInvoices] = useState<{[key: string]: number}>(() => {
    const saved = localStorage.getItem('monthlyInvoices');
    return saved ? JSON.parse(saved) : {};
  });
  const [dashboardFilter, setDashboardFilter] = useState<string>('all'); // 'all' ou 'YYYY-MM'

  // Mock data - In real app, this would come from backend/database
  const [transactions, setTransactions] = useState<Transaction[]>(() => {
    const saved = localStorage.getItem('transactions');
    return saved ? JSON.parse(saved) : [];
  });

  const [investments, setInvestments] = useState<Investment[]>(() => {
    const saved = localStorage.getItem('investments');
    return saved ? JSON.parse(saved) : [];
  });

  const [newTransaction, setNewTransaction] = useState({
    description: '',
    amount: '',
    category: '',
    type: 'expense' as 'expense' | 'income',
    date: new Date().toISOString().split('T')[0]
  });

  const [newInvestment, setNewInvestment] = useState({
    name: '',
    type: '',
    amount: '',
    returns: ''
  });

  const categories = ['Alimentação', 'Transporte', 'Entretenimento', 'Saúde', 'Compras', 'Contas', 'Educação', 'Salário', 'Investimentos', 'Outros'];
  const investmentTypes = ['Renda Fixa', 'Ações', 'Fundos Imobiliários', 'Criptomoedas', 'Fundos de Investimento'];

  // Calculations
  const filteredDashboardTransactions = dashboardFilter === 'all' 
    ? transactions 
    : transactions.filter(t => t.date.startsWith(dashboardFilter));
  
  const totalIncome = filteredDashboardTransactions.filter(t => t.type === 'income').reduce((sum, t) => sum + t.amount, 0);
  const totalExpenses = filteredDashboardTransactions.filter(t => t.type === 'expense').reduce((sum, t) => sum + t.amount, 0);
  const balance = totalIncome - totalExpenses;
  const totalInvestments = investments.reduce((sum, inv) => sum + inv.amount, 0);
  const investmentReturns = investments.reduce((sum, inv) => sum + (inv.amount * inv.returns / 100), 0);

  // Chart data
  const categoryData = categories.map(cat => {
    const total = filteredDashboardTransactions
      .filter(t => t.category === cat && t.type === 'expense')
      .reduce((sum, t) => sum + t.amount, 0);
    return { name: cat, value: total };
  }).filter(d => d.value > 0);

  const monthlyData = [
    { month: 'Set', receita: 5200, despesa: 3800, investido: 1000 },
    { month: 'Out', receita: 5500, despesa: 4200, investido: 1200 },
    { month: 'Nov', receita: 5500, despesa: 3900, investido: 1400 },
    { month: 'Dez', receita: 6200, despesa: 5100, investido: 900 },
    { month: 'Jan', receita: 5500, despesa: totalExpenses, investido: 800 },
  ];

  const investmentAllocation = investments.map(inv => ({
    name: inv.name,
    value: inv.allocation
  }));

  const COLORS = ['#3b82f6', '#8b5cf6', '#ec4899', '#f59e0b', '#10b981', '#06b6d4', '#6366f1', '#f97316'];

  const handleAddTransaction = () => {
    if (newTransaction.description && newTransaction.amount && newTransaction.category) {
      const transaction: Transaction = {
        id: Date.now().toString(),
        date: newTransaction.date,
        description: newTransaction.description,
        amount: parseFloat(newTransaction.amount),
        category: newTransaction.category,
        type: newTransaction.type
      };
      const updatedTransactions = [transaction, ...transactions];
      setTransactions(updatedTransactions);
      localStorage.setItem('transactions', JSON.stringify(updatedTransactions));
      setNewTransaction({
        description: '',
        amount: '',
        category: '',
        type: 'expense',
        date: new Date().toISOString().split('T')[0]
      });
      setShowAddTransaction(false);
    }
  };

  const handleAddInvestment = () => {
    if (newInvestment.name && newInvestment.type && newInvestment.amount) {
      const amount = parseFloat(newInvestment.amount);
      const totalWithNew = totalInvestments + amount;
      const allocation = (amount / totalWithNew) * 100;
      
      const investment: Investment = {
        id: Date.now().toString(),
        name: newInvestment.name,
        type: newInvestment.type,
        amount: amount,
        returns: parseFloat(newInvestment.returns) || 0,
        allocation: allocation
      };
      
      const updatedInvestments = investments.map(inv => ({
        ...inv,
        allocation: (inv.amount / totalWithNew) * 100
      }));
      
      const finalInvestments = [investment, ...updatedInvestments];
      setInvestments(finalInvestments);
      localStorage.setItem('investments', JSON.stringify(finalInvestments));
      setNewInvestment({ name: '', type: '', amount: '', returns: '' });
      setShowAddInvestment(false);
    }
  };

  const handleAddInvoice = () => {
    if (newInvoice.month && newInvoice.amount) {
      const amount = parseFloat(newInvoice.amount);
      const updatedInvoices = {...monthlyInvoices, [newInvoice.month]: amount};
      setMonthlyInvoices(updatedInvoices);
      localStorage.setItem('monthlyInvoices', JSON.stringify(updatedInvoices));
      
      // Adiciona como despesa também
      const transaction: Transaction = {
        id: Date.now().toString(),
        date: `${newInvoice.month}-01`,
        description: 'Fatura do Cartão de Crédito',
        amount: amount,
        category: 'Contas',
        type: 'expense'
      };
      const updatedTransactions = [transaction, ...transactions];
      setTransactions(updatedTransactions);
      localStorage.setItem('transactions', JSON.stringify(updatedTransactions));
      
      setNewInvoice({
        month: new Date().toISOString().slice(0, 7),
        amount: ''
      });
      setShowAddInvoice(false);
    }
  };

  const handleDeleteTransaction = (id: string) => {
    if (confirm('Tem certeza que deseja excluir esta transação?')) {
      const updatedTransactions = transactions.filter(t => t.id !== id);
      setTransactions(updatedTransactions);
      localStorage.setItem('transactions', JSON.stringify(updatedTransactions));
    }
  };

  const handleDeleteInvestment = (id: string) => {
    if (confirm('Tem certeza que deseja excluir este investimento?')) {
      const investmentToDelete = investments.find(inv => inv.id === id);
      const remainingInvestments = investments.filter(inv => inv.id !== id);
      
      // Recalcula a alocação dos investimentos restantes
      const totalRemaining = remainingInvestments.reduce((sum, inv) => sum + inv.amount, 0);
      const updatedInvestments = remainingInvestments.map(inv => ({
        ...inv,
        allocation: totalRemaining > 0 ? (inv.amount / totalRemaining) * 100 : 0
      }));
      
      setInvestments(updatedInvestments);
      localStorage.setItem('investments', JSON.stringify(updatedInvestments));
    }
  };

  

  const filteredTransactions = filterCategory === 'all' 
    ? transactions 
    : transactions.filter(t => t.category === filterCategory);

  return (
    <div className="min-h-screen bg-background">
      {/* Header */}
      <header className="border-b border-border bg-card">
        <div className="container mx-auto px-4 py-4">
          <div className="flex items-center justify-between">
            <h1 className="text-2xl font-bold text-foreground">Gestor Financeiro</h1>
            <div className="flex gap-2">
              <Button
                variant={activeTab === 'dashboard' ? 'default' : 'ghost'}
                onClick={() => setActiveTab('dashboard')}
                className="text-sm"
              >
                Dashboard
              </Button>
              <Button
                variant={activeTab === 'transactions' ? 'default' : 'ghost'}
                onClick={() => setActiveTab('transactions')}
                className="text-sm"
              >
                Transações
              </Button>
              <Button
                variant={activeTab === 'investments' ? 'default' : 'ghost'}
                onClick={() => setActiveTab('investments')}
                className="text-sm"
              >
                Investimentos
              </Button>
              <Button
                variant={activeTab === 'analytics' ? 'default' : 'ghost'}
                onClick={() => setActiveTab('analytics')}
                className="text-sm"
              >
                Análises
              </Button>
            </div>
          </div>
        </div>
      </header>

      <main className="container mx-auto px-4 py-8">
        {/* Dashboard Tab */}
        {activeTab === 'dashboard' && (
          <div className="space-y-6">
            {/* Month Filter */}
            <Card className="bg-card border-border">
              <CardContent className="p-4">
                <div className="flex items-center gap-4">
                  <Label htmlFor="dashboard-filter" className="text-foreground font-medium">Filtrar por período:</Label>
                  <Select value={dashboardFilter} onValueChange={setDashboardFilter}>
                    <SelectTrigger className="w-[200px]">
                      <SelectValue />
                    </SelectTrigger>
                    <SelectContent>
                      <SelectItem value="all">Todos os meses</SelectItem>
                      {Object.keys(monthlyInvoices)
                        .sort((a, b) => b.localeCompare(a))
                        .map(month => {
                          const [year, monthNum] = month.split('-');
                          const monthNames = ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'];
                          const monthName = monthNames[parseInt(monthNum) - 1];
                          return (
                            <SelectItem key={month} value={month}>{monthName}/{year}</SelectItem>
                          );
                        })}
                    </SelectContent>
                  </Select>
                </div>
              </CardContent>
            </Card>

            {/* Summary Cards */}
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
              <Card className="bg-card border-border">
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium text-muted-foreground">Saldo Atual</CardTitle>
                  <Wallet className="h-4 w-4 text-muted-foreground" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold text-foreground">R$ {balance.toFixed(2)}</div>
                  <p className="text-xs text-muted-foreground mt-1">
                    {balance > 0 ? '+' : ''}{((balance / totalIncome) * 100).toFixed(1)}% do mês
                  </p>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium text-muted-foreground">Receitas</CardTitle>
                  <ArrowUpRight className="h-4 w-4 text-green-500" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold text-foreground">R$ {totalIncome.toFixed(2)}</div>
                  <p className="text-xs text-green-600 mt-1">Este mês</p>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium text-muted-foreground">Despesas</CardTitle>
                  <ArrowDownRight className="h-4 w-4 text-red-500" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold text-foreground">R$ {totalExpenses.toFixed(2)}</div>
                  <p className="text-xs text-red-600 mt-1">{((totalExpenses / totalIncome) * 100).toFixed(1)}% da receita</p>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader className="flex flex-row items-center justify-between pb-2">
                  <CardTitle className="text-sm font-medium text-muted-foreground">Investimentos</CardTitle>
                  <TrendingUp className="h-4 w-4 text-muted-foreground" />
                </CardHeader>
                <CardContent>
                  <div className="text-2xl font-bold text-foreground">R$ {totalInvestments.toFixed(2)}</div>
                  <p className="text-xs text-green-600 mt-1">+R$ {investmentReturns.toFixed(2)} retorno</p>
                </CardContent>
              </Card>
            </div>

            {/* Charts Row */}
            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Fluxo Mensal</CardTitle>
                  <CardDescription className="text-muted-foreground">Receitas vs Despesas</CardDescription>
                </CardHeader>
                <CardContent>
                  <ResponsiveContainer width="100%" height={300}>
                    <BarChart data={monthlyData}>
                      <CartesianGrid strokeDasharray="3 3" stroke="#444" />
                      <XAxis dataKey="month" stroke="#888" />
                      <YAxis stroke="#888" />
                      <Tooltip 
                        contentStyle={{ backgroundColor: '#1f1f1f', border: '1px solid #333' }}
                        labelStyle={{ color: '#fff' }}
                      />
                      <Legend />
                      <Bar dataKey="receita" fill="#10b981" name="Receita" />
                      <Bar dataKey="despesa" fill="#ef4444" name="Despesa" />
                      <Bar dataKey="investido" fill="#3b82f6" name="Investido" />
                    </BarChart>
                  </ResponsiveContainer>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Gastos por Categoria</CardTitle>
                  <CardDescription className="text-muted-foreground">Distribuição das despesas</CardDescription>
                </CardHeader>
                <CardContent>
                  <ResponsiveContainer width="100%" height={300}>
                    <PieChart>
                      <Pie
                        data={categoryData}
                        cx="50%"
                        cy="50%"
                        labelLine={false}
                        label={({ name, percent }) => `${name} ${(percent * 100).toFixed(0)}%`}
                        outerRadius={80}
                        fill="#8884d8"
                        dataKey="value"
                      >
                        {categoryData.map((entry, index) => (
                          <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                        ))}
                      </Pie>
                      <Tooltip 
                        contentStyle={{ backgroundColor: '#1f1f1f', border: '1px solid #333' }}
                        formatter={(value: number) => `R$ ${value.toFixed(2)}`}
                      />
                    </PieChart>
                  </ResponsiveContainer>
                </CardContent>
              </Card>
            </div>

            {/* Recent Transactions */}
            <Card className="bg-card border-border">
              <CardHeader>
                <CardTitle className="text-foreground">Transações Recentes</CardTitle>
                <CardDescription className="text-muted-foreground">Últimas movimentações</CardDescription>
              </CardHeader>
              <CardContent>
                {filteredDashboardTransactions.length === 0 ? (
                  <div className="text-center py-8">
                    <CreditCard className="h-12 w-12 text-muted-foreground mx-auto mb-4" />
                    <p className="text-muted-foreground">Nenhuma transação neste período</p>
                    <p className="text-sm text-muted-foreground mt-2">
                      {dashboardFilter === 'all' ? 'Adicione transações ou importe sua fatura para começar' : 'Não há transações para o mês selecionado'}
                    </p>
                  </div>
                ) : (
                  <div className="space-y-4">
                    {filteredDashboardTransactions.slice(0, 5).map(transaction => (
                      <div key={transaction.id} className="flex items-center justify-between p-3 bg-muted rounded-lg">
                        <div className="flex items-center gap-3">
                          <div className={`p-2 rounded-full ${transaction.type === 'income' ? 'bg-green-500/20' : 'bg-red-500/20'}`}>
                            <CreditCard className={`h-4 w-4 ${transaction.type === 'income' ? 'text-green-500' : 'text-red-500'}`} />
                          </div>
                          <div>
                            <p className="font-medium text-foreground">{transaction.description}</p>
                            <p className="text-sm text-muted-foreground">{transaction.category} • {new Date(transaction.date).toLocaleDateString('pt-BR')}</p>
                          </div>
                        </div>
                        <p className={`font-semibold ${transaction.type === 'income' ? 'text-green-500' : 'text-red-500'}`}>
                          {transaction.type === 'income' ? '+' : '-'}R$ {transaction.amount.toFixed(2)}
                        </p>
                      </div>
                    ))}
                  </div>
                )}
              </CardContent>
            </Card>
          </div>
        )}

        {/* Transactions Tab */}
        {activeTab === 'transactions' && (
          <div className="space-y-6">
            <div className="flex justify-between items-center">
              <div className="flex gap-4 items-center">
                <h2 className="text-2xl font-bold text-foreground">Transações</h2>
                <div className="flex gap-2 items-center">
                  <Filter className="h-4 w-4 text-muted-foreground" />
                  <Select value={filterCategory} onValueChange={setFilterCategory}>
                    <SelectTrigger className="w-[180px]">
                      <SelectValue placeholder="Categoria" />
                    </SelectTrigger>
                    <SelectContent>
                      <SelectItem value="all">Todas</SelectItem>
                      {categories.map(cat => (
                        <SelectItem key={cat} value={cat}>{cat}</SelectItem>
                      ))}
                    </SelectContent>
                  </Select>
                </div>
              </div>
              <div className="flex gap-2">
                <Button onClick={() => setShowAddInvoice(true)} variant="outline" className="gap-2">
                  <CreditCard className="h-4 w-4" />
                  Adicionar Fatura
                </Button>
                <Button onClick={() => setShowAddTransaction(true)} className="gap-2">
                  <Plus className="h-4 w-4" />
                  Nova Transação
                </Button>
              </div>
            </div>

            {Object.keys(monthlyInvoices).length > 0 && (
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Faturas por Mês</CardTitle>
                  <CardDescription className="text-muted-foreground">Total gasto no cartão por mês</CardDescription>
                </CardHeader>
                <CardContent>
                  <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
                    {Object.entries(monthlyInvoices)
                      .sort(([a], [b]) => b.localeCompare(a))
                      .map(([month, total]) => {
                        const [year, monthNum] = month.split('-');
                        const monthNames = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
                        const monthName = monthNames[parseInt(monthNum) - 1];
                        return (
                          <div key={month} className="p-4 bg-muted rounded-lg">
                            <p className="text-sm text-muted-foreground mb-1">{monthName}/{year}</p>
                            <p className="text-xl font-bold text-foreground">R$ {total.toFixed(2)}</p>
                          </div>
                        );
                      })}
                  </div>
                </CardContent>
              </Card>
            )}

            {showAddInvoice && (
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Adicionar Fatura do Cartão</CardTitle>
                  <CardDescription className="text-muted-foreground">
                    Registre o valor total da fatura de um mês
                  </CardDescription>
                </CardHeader>
                <CardContent>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <Label htmlFor="invoice-month" className="text-foreground">Mês da Fatura</Label>
                      <Input
                        id="invoice-month"
                        type="month"
                        value={newInvoice.month}
                        onChange={(e) => setNewInvoice({...newInvoice, month: e.target.value})}
                      />
                    </div>
                    <div>
                      <Label htmlFor="invoice-amount" className="text-foreground">Valor Total</Label>
                      <Input
                        id="invoice-amount"
                        type="number"
                        value={newInvoice.amount}
                        onChange={(e) => setNewInvoice({...newInvoice, amount: e.target.value})}
                        placeholder="0.00"
                      />
                    </div>
                  </div>
                  <div className="flex gap-2 mt-4">
                    <Button onClick={handleAddInvoice}>Adicionar Fatura</Button>
                    <Button variant="outline" onClick={() => setShowAddInvoice(false)}>Cancelar</Button>
                  </div>
                </CardContent>
              </Card>
            )}

            {showAddTransaction && (
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Adicionar Transação</CardTitle>
                </CardHeader>
                <CardContent>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <Label htmlFor="description" className="text-foreground">Descrição</Label>
                      <Input
                        id="description"
                        value={newTransaction.description}
                        onChange={(e) => setNewTransaction({...newTransaction, description: e.target.value})}
                        placeholder="Ex: Supermercado"
                      />
                    </div>
                    <div>
                      <Label htmlFor="amount" className="text-foreground">Valor</Label>
                      <Input
                        id="amount"
                        type="number"
                        value={newTransaction.amount}
                        onChange={(e) => setNewTransaction({...newTransaction, amount: e.target.value})}
                        placeholder="0.00"
                      />
                    </div>
                    <div>
                      <Label htmlFor="category" className="text-foreground">Categoria</Label>
                      <Select value={newTransaction.category} onValueChange={(value) => setNewTransaction({...newTransaction, category: value})}>
                        <SelectTrigger>
                          <SelectValue placeholder="Selecione" />
                        </SelectTrigger>
                        <SelectContent>
                          {categories.map(cat => (
                            <SelectItem key={cat} value={cat}>{cat}</SelectItem>
                          ))}
                        </SelectContent>
                      </Select>
                    </div>
                    <div>
                      <Label htmlFor="type" className="text-foreground">Tipo</Label>
                      <Select value={newTransaction.type} onValueChange={(value: 'expense' | 'income') => setNewTransaction({...newTransaction, type: value})}>
                        <SelectTrigger>
                          <SelectValue />
                        </SelectTrigger>
                        <SelectContent>
                          <SelectItem value="expense">Despesa</SelectItem>
                          <SelectItem value="income">Receita</SelectItem>
                        </SelectContent>
                      </Select>
                    </div>
                    <div>
                      <Label htmlFor="date" className="text-foreground">Data</Label>
                      <Input
                        id="date"
                        type="date"
                        value={newTransaction.date}
                        onChange={(e) => setNewTransaction({...newTransaction, date: e.target.value})}
                      />
                    </div>
                  </div>
                  <div className="flex gap-2 mt-4">
                    <Button onClick={handleAddTransaction}>Adicionar</Button>
                    <Button variant="outline" onClick={() => setShowAddTransaction(false)}>Cancelar</Button>
                  </div>
                </CardContent>
              </Card>
            )}

            {filteredTransactions.length === 0 ? (
              <Card className="bg-card border-border">
                <CardContent className="p-12 text-center">
                  <CreditCard className="h-12 w-12 text-muted-foreground mx-auto mb-4" />
                  <p className="text-foreground font-medium mb-2">Nenhuma transação encontrada</p>
                  <p className="text-sm text-muted-foreground mb-4">
                    Comece adicionando transações manualmente ou importe sua fatura
                  </p>
                </CardContent>
              </Card>
            ) : (
              <div className="grid gap-4">
                {filteredTransactions.map(transaction => (
                  <Card key={transaction.id} className="bg-card border-border">
                    <CardContent className="p-4">
                      <div className="flex items-center justify-between">
                        <div className="flex items-center gap-4 flex-1">
                          <div className={`p-3 rounded-full ${transaction.type === 'income' ? 'bg-green-500/20' : 'bg-red-500/20'}`}>
                            <CreditCard className={`h-5 w-5 ${transaction.type === 'income' ? 'text-green-500' : 'text-red-500'}`} />
                          </div>
                          <div className="flex-1">
                            <p className="font-semibold text-foreground">{transaction.description}</p>
                            <p className="text-sm text-muted-foreground">{transaction.category} • {new Date(transaction.date).toLocaleDateString('pt-BR')}</p>
                          </div>
                        </div>
                        <div className="flex items-center gap-4">
                          <div className="text-right">
                            <p className={`text-xl font-bold ${transaction.type === 'income' ? 'text-green-500' : 'text-red-500'}`}>
                              {transaction.type === 'income' ? '+' : '-'}R$ {transaction.amount.toFixed(2)}
                            </p>
                          </div>
                          <Button
                            variant="ghost"
                            size="sm"
                            onClick={() => handleDeleteTransaction(transaction.id)}
                            className="text-red-500 hover:text-red-600 hover:bg-red-500/10"
                          >
                            <Trash2 className="h-4 w-4" />
                          </Button>
                        </div>
                      </div>
                    </CardContent>
                  </Card>
                ))}
              </div>
            )}
          </div>
        )}

        {/* Investments Tab */}
        {activeTab === 'investments' && (
          <div className="space-y-6">
            <div className="flex justify-between items-center">
              <h2 className="text-2xl font-bold text-foreground">Investimentos</h2>
              <Button onClick={() => setShowAddInvestment(true)} className="gap-2">
                <Plus className="h-4 w-4" />
                Novo Investimento
              </Button>
            </div>

            {showAddInvestment && (
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Adicionar Investimento</CardTitle>
                </CardHeader>
                <CardContent>
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                      <Label htmlFor="inv-name" className="text-foreground">Nome</Label>
                      <Input
                        id="inv-name"
                        value={newInvestment.name}
                        onChange={(e) => setNewInvestment({...newInvestment, name: e.target.value})}
                        placeholder="Ex: Tesouro Selic"
                      />
                    </div>
                    <div>
                      <Label htmlFor="inv-type" className="text-foreground">Tipo</Label>
                      <Select value={newInvestment.type} onValueChange={(value) => setNewInvestment({...newInvestment, type: value})}>
                        <SelectTrigger>
                          <SelectValue placeholder="Selecione" />
                        </SelectTrigger>
                        <SelectContent>
                          {investmentTypes.map(type => (
                            <SelectItem key={type} value={type}>{type}</SelectItem>
                          ))}
                        </SelectContent>
                      </Select>
                    </div>
                    <div>
                      <Label htmlFor="inv-amount" className="text-foreground">Valor Investido</Label>
                      <Input
                        id="inv-amount"
                        type="number"
                        value={newInvestment.amount}
                        onChange={(e) => setNewInvestment({...newInvestment, amount: e.target.value})}
                        placeholder="0.00"
                      />
                    </div>
                    <div>
                      <Label htmlFor="inv-returns" className="text-foreground">Retorno (%)</Label>
                      <Input
                        id="inv-returns"
                        type="number"
                        value={newInvestment.returns}
                        onChange={(e) => setNewInvestment({...newInvestment, returns: e.target.value})}
                        placeholder="0.0"
                      />
                    </div>
                  </div>
                  <div className="flex gap-2 mt-4">
                    <Button onClick={handleAddInvestment}>Adicionar</Button>
                    <Button variant="outline" onClick={() => setShowAddInvestment(false)}>Cancelar</Button>
                  </div>
                </CardContent>
              </Card>
            )}

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <Card className="lg:col-span-2 bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Portfólio</CardTitle>
                  <CardDescription className="text-muted-foreground">Seus investimentos</CardDescription>
                </CardHeader>
                <CardContent>
                  {investments.length === 0 ? (
                    <div className="text-center py-8">
                      <TrendingUp className="h-12 w-12 text-muted-foreground mx-auto mb-4" />
                      <p className="text-muted-foreground">Nenhum investimento ainda</p>
                      <p className="text-sm text-muted-foreground mt-2">
                        Comece adicionando seus investimentos
                      </p>
                    </div>
                  ) : (
                    <div className="space-y-4">
                      {investments.map(inv => (
                        <div key={inv.id} className="p-4 bg-muted rounded-lg">
                          <div className="flex justify-between items-start mb-2">
                            <div className="flex-1">
                              <p className="font-semibold text-foreground">{inv.name}</p>
                              <p className="text-sm text-muted-foreground">{inv.type}</p>
                            </div>
                            <div className="flex items-center gap-2">
                              <p className={`text-sm font-semibold ${inv.returns >= 0 ? 'text-green-500' : 'text-red-500'}`}>
                                {inv.returns >= 0 ? '+' : ''}{inv.returns.toFixed(2)}%
                              </p>
                              <Button
                                variant="ghost"
                                size="sm"
                                onClick={() => handleDeleteInvestment(inv.id)}
                                className="text-red-500 hover:text-red-600 hover:bg-red-500/10"
                              >
                                <Trash2 className="h-4 w-4" />
                              </Button>
                            </div>
                          </div>
                          <div className="flex justify-between items-end">
                            <div>
                              <p className="text-2xl font-bold text-foreground">R$ {inv.amount.toFixed(2)}</p>
                              <p className="text-xs text-muted-foreground">{inv.allocation.toFixed(1)}% do portfólio</p>
                            </div>
                            <p className={`text-sm ${inv.returns >= 0 ? 'text-green-600' : 'text-red-600'}`}>
                              {inv.returns >= 0 ? '+' : ''}R$ {(inv.amount * inv.returns / 100).toFixed(2)}
                            </p>
                          </div>
                        </div>
                      ))}
                    </div>
                  )}
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Alocação</CardTitle>
                  <CardDescription className="text-muted-foreground">Distribuição do portfólio</CardDescription>
                </CardHeader>
                <CardContent>
                  <ResponsiveContainer width="100%" height={250}>
                    <PieChart>
                      <Pie
                        data={investmentAllocation}
                        cx="50%"
                        cy="50%"
                        labelLine={false}
                        label={({ value }) => `${value.toFixed(0)}%`}
                        outerRadius={80}
                        fill="#8884d8"
                        dataKey="value"
                      >
                        {investmentAllocation.map((entry, index) => (
                          <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                        ))}
                      </Pie>
                      <Tooltip 
                        contentStyle={{ backgroundColor: '#1f1f1f', border: '1px solid #333' }}
                        formatter={(value: number) => `${value.toFixed(1)}%`}
                      />
                    </PieChart>
                  </ResponsiveContainer>
                  <div className="mt-4 space-y-2">
                    {investments.map((inv, index) => (
                      <div key={inv.id} className="flex items-center gap-2">
                        <div className="w-3 h-3 rounded-full" style={{ backgroundColor: COLORS[index % COLORS.length] }} />
                        <p className="text-xs text-foreground truncate flex-1">{inv.name}</p>
                        <p className="text-xs text-muted-foreground">{inv.allocation.toFixed(0)}%</p>
                      </div>
                    ))}
                  </div>
                </CardContent>
              </Card>
            </div>
          </div>
        )}

        {/* Analytics Tab */}
        {activeTab === 'analytics' && (
          <div className="space-y-6">
            <h2 className="text-2xl font-bold text-foreground">Análises Financeiras</h2>

            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-sm text-muted-foreground">Taxa de Poupança</CardTitle>
                </CardHeader>
                <CardContent>
                  <p className="text-3xl font-bold text-foreground">{((balance / totalIncome) * 100).toFixed(1)}%</p>
                  <p className="text-xs text-muted-foreground mt-1">Do que você ganha</p>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-sm text-muted-foreground">Maior Despesa</CardTitle>
                </CardHeader>
                <CardContent>
                  <p className="text-3xl font-bold text-foreground">{categoryData.sort((a, b) => b.value - a.value)[0]?.name || 'N/A'}</p>
                  <p className="text-xs text-muted-foreground mt-1">R$ {categoryData[0]?.value.toFixed(2) || '0.00'}</p>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-sm text-muted-foreground">Retorno Investimentos</CardTitle>
                </CardHeader>
                <CardContent>
                  <p className="text-3xl font-bold text-green-500">+{((investmentReturns / totalInvestments) * 100).toFixed(2)}%</p>
                  <p className="text-xs text-muted-foreground mt-1">R$ {investmentReturns.toFixed(2)}</p>
                </CardContent>
              </Card>
            </div>

            <Card className="bg-card border-border">
              <CardHeader>
                <CardTitle className="text-foreground">Evolução Patrimonial</CardTitle>
                <CardDescription className="text-muted-foreground">Últimos 5 meses</CardDescription>
              </CardHeader>
              <CardContent>
                <ResponsiveContainer width="100%" height={300}>
                  <LineChart data={monthlyData}>
                    <CartesianGrid strokeDasharray="3 3" stroke="#444" />
                    <XAxis dataKey="month" stroke="#888" />
                    <YAxis stroke="#888" />
                    <Tooltip 
                      contentStyle={{ backgroundColor: '#1f1f1f', border: '1px solid #333' }}
                      labelStyle={{ color: '#fff' }}
                    />
                    <Legend />
                    <Line type="monotone" dataKey="receita" stroke="#10b981" strokeWidth={2} name="Receita" />
                    <Line type="monotone" dataKey="despesa" stroke="#ef4444" strokeWidth={2} name="Despesa" />
                    <Line type="monotone" dataKey="investido" stroke="#3b82f6" strokeWidth={2} name="Investido" />
                  </LineChart>
                </ResponsiveContainer>
              </CardContent>
            </Card>

            <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Despesas Detalhadas</CardTitle>
                  <CardDescription className="text-muted-foreground">Por categoria</CardDescription>
                </CardHeader>
                <CardContent>
                  <div className="space-y-4">
                    {categoryData.sort((a, b) => b.value - a.value).map((cat, index) => {
                      const percentage = (cat.value / totalExpenses) * 100;
                      return (
                        <div key={cat.name}>
                          <div className="flex justify-between mb-1">
                            <span className="text-sm text-foreground">{cat.name}</span>
                            <span className="text-sm font-semibold text-foreground">R$ {cat.value.toFixed(2)}</span>
                          </div>
                          <div className="w-full bg-muted rounded-full h-2">
                            <div
                              className="h-2 rounded-full"
                              style={{
                                width: `${percentage}%`,
                                backgroundColor: COLORS[index % COLORS.length]
                              }}
                            />
                          </div>
                          <p className="text-xs text-muted-foreground mt-1">{percentage.toFixed(1)}% do total</p>
                        </div>
                      );
                    })}
                  </div>
                </CardContent>
              </Card>

              <Card className="bg-card border-border">
                <CardHeader>
                  <CardTitle className="text-foreground">Metas Financeiras</CardTitle>
                  <CardDescription className="text-muted-foreground">Objetivos do mês</CardDescription>
                </CardHeader>
                <CardContent>
                  <div className="space-y-6">
                    <div>
                      <div className="flex justify-between mb-2">
                        <span className="text-sm text-foreground">Economizar R$ 1.000</span>
                        <span className="text-sm font-semibold text-foreground">R$ {balance.toFixed(2)}</span>
                      </div>
                      <div className="w-full bg-muted rounded-full h-2">
                        <div
                          className="bg-green-500 h-2 rounded-full"
                          style={{ width: `${Math.min((balance / 1000) * 100, 100)}%` }}
                        />
                      </div>
                      <p className="text-xs text-muted-foreground mt-1">{Math.min((balance / 1000) * 100, 100).toFixed(0)}% alcançado</p>
                    </div>

                    <div>
                      <div className="flex justify-between mb-2">
                        <span className="text-sm text-foreground">Limitar gastos a R$ 4.000</span>
                        <span className="text-sm font-semibold text-foreground">R$ {totalExpenses.toFixed(2)}</span>
                      </div>
                      <div className="w-full bg-muted rounded-full h-2">
                        <div
                          className={`h-2 rounded-full ${totalExpenses > 4000 ? 'bg-red-500' : 'bg-blue-500'}`}
                          style={{ width: `${Math.min((totalExpenses / 4000) * 100, 100)}%` }}
                        />
                      </div>
                      <p className="text-xs text-muted-foreground mt-1">{((totalExpenses / 4000) * 100).toFixed(0)}% utilizado</p>
                    </div>

                    <div>
                      <div className="flex justify-between mb-2">
                        <span className="text-sm text-foreground">Aportar R$ 800 em investimentos</span>
                        <span className="text-sm font-semibold text-foreground">R$ 800.00</span>
                      </div>
                      <div className="w-full bg-muted rounded-full h-2">
                        <div className="bg-purple-500 h-2 rounded-full" style={{ width: '100%' }} />
                      </div>
                      <p className="text-xs text-muted-foreground mt-1">100% alcançado</p>
                    </div>
                  </div>
                </CardContent>
              </Card>
            </div>
          </div>
        )}
      </main>
    </div>
  );
}
