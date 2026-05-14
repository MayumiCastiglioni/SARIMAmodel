# 🚀 Guia Rápido: Como Usar o Notebook SARIMA

**Arquivo:** `2_pipeline_sarima_revisado.ipynb`

---

## 📋 Checklist de Uso

### 1️⃣ Preparação dos Dados

Seu arquivo de dados deve ter **pelo menos**:
- ✅ Uma coluna com valores numéricos (sua série temporal)
- ✅ Preferencialmente uma coluna de data/timestamp
- ✅ Mínimo de 50 observações (idealmente 100+)

**Formatos aceitos:**
- CSV (`.csv`)
- Excel (`.xlsx`, `.xls`)
- DataFrame pandas já carregado

---

### 2️⃣ Ajustes Obrigatórios

#### **Célula 5: Carregar Dados**
```python
# OPÇÃO 1: Carregar de arquivo CSV
df = pd.read_csv('seu_arquivo.csv')

# OPÇÃO 2: Carregar de Excel
df = pd.read_excel('seu_arquivo.xlsx')

# OPÇÃO 3: Criar DataFrame manualmente
df = pd.DataFrame({
    'data': pd.date_range('2020-01-01', periods=200, freq='D'),
    'valor': [...]  # seus dados aqui
})

# DEFINA O NOME DA COLUNA COM OS VALORES
coluna_valor = 'nome_da_sua_coluna'
```

#### **Célula 13: Período Sazonal**
```python
# Escolha conforme seu padrão de dados:
periodo_sazonal = 7   # Dados diários com padrão semanal
# periodo_sazonal = 12  # Dados mensais com padrão anual
# periodo_sazonal = 4   # Dados trimestrais com padrão anual
```

**Como identificar o período sazonal?**
- Dados diários: analise se há padrão que se repete a cada 7 dias (semanal)
- Dados mensais: padrão que se repete a cada 12 meses (anual)
- Dados horários: padrão que se repete a cada 24 horas (diário)

#### **Célula 15: Horizonte de Teste**
```python
# Quantos períodos você quer prever?
h = 10  # Ajuste conforme necessário

# Recomendações:
# - Série pequena (<100 obs): h = 5-10
# - Série média (100-500 obs): h = 10-20
# - Série grande (>500 obs): h = 20-30
```

---

### 3️⃣ Execução

1. **Abra o notebook** no Jupyter/JupyterLab/VS Code
2. **Execute as células sequencialmente** (Shift+Enter)
3. **Aguarde** os resultados de cada etapa

**Tempo estimado:**
- Análise exploratória: ~1 min
- Testes de estacionariedade: ~30 seg
- Grid search SARIMA: ~2-5 min (depende do range)
- Diagnóstico e previsão: ~30 seg

---

### 4️⃣ Interpretação dos Resultados

#### **Estacionariedade (Célula 9)**
```
✅ ESTACIONÁRIA (ambos os testes concordam)
   → Sua série não precisa de diferenciação (d=0, D=0)

⚠️  NÃO ESTACIONÁRIA (ambos os testes concordam)
   → Aplicar diferenciação (d=1 ou d=2)
```

#### **Força da Sazonalidade (Célula 13)**
```
📊 Força da Sazonalidade: 0.7542
   → Sazonalidade FORTE detectada. Recomenda-se usar SARIMA.
```

**Interpretação:**
- **> 0.6:** Sazonalidade FORTE → SARIMA é essencial
- **0.3 - 0.6:** Sazonalidade MODERADA → Considere SARIMA
- **< 0.3:** Sazonalidade FRACA → ARIMA pode ser suficiente

#### **Melhor Modelo SARIMA (Célula 23)**
```
✅ Melhor SARIMA(1, 1, 1)x(1, 1, 1, 7) com BIC = 1234.56
```

**O que significam os parâmetros?**
- **(p, d, q)** = (1, 1, 1): componentes não sazonais
  - p=1: modelo usa 1 lag autoregressivo
  - d=1: aplicou diferenciação de ordem 1
  - q=1: modelo usa 1 lag de média móvel
  
- **(P, D, Q, m)** = (1, 1, 1, 7): componentes sazonais
  - P=1: usa 1 lag sazonal autoregressivo
  - D=1: aplicou diferenciação sazonal de ordem 1
  - Q=1: usa 1 lag sazonal de média móvel
  - m=7: período sazonal de 7

#### **Diagnóstico de Resíduos (Célula 25)**
```
Teste Ljung-Box (lags=20):
p-value = 0.3456

✅ Resíduos são ruído branco (não há autocorrelação significativa)
```

**O que isso significa?**
- **p-value > 0.05:** ✅ Modelo está bem ajustado (bom!)
- **p-value < 0.05:** ⚠️ Resíduos ainda têm padrão (modelo pode melhorar)

#### **Métricas de Performance (Célula 27)**
```
📊 Métricas SARIMA:
   MAE : 5.23
   RMSE: 6.78
   MAPE: 8.45
```

**O que é bom?**
- **MAPE < 10%:** Excelente
- **MAPE 10-20%:** Bom
- **MAPE 20-50%:** Aceitável
- **MAPE > 50%:** Ruim (revisar dados ou modelo)

---

### 5️⃣ Troubleshooting

#### ❌ "Série não converge" ou "BIC = inf"
**Causa:** Série muito volátil ou com valores extremos  
**Solução:** Aplique transformação logarítmica antes:
```python
df['valor_log'] = np.log(df[coluna_valor])
coluna_valor = 'valor_log'
```

#### ❌ "Grid search muito lento"
**Causa:** Range de busca muito grande  
**Solução:** Reduza os ranges na célula 23:
```python
p_range = range(0, 2)  # em vez de (0, 3)
q_range = range(0, 2)
P_range = range(0, 2)
Q_range = range(0, 2)
```

#### ❌ "Resíduos não são ruído branco"
**Causa:** Modelo não capturou todo o padrão  
**Solução:**
1. Teste período sazonal diferente (m=12, m=4, etc.)
2. Aumente os ranges do grid search
3. Verifique se há outliers nos dados

#### ❌ "MAPE muito alto (>50%)"
**Causa:** Modelo inadequado ou dados problemáticos  
**Solução:**
1. Verifique se a série tem tendência forte → aplicar diferenciação
2. Verifique se há quebras estruturais → dividir série em períodos
3. Considere transformação Box-Cox para estabilizar variância

---

### 6️⃣ Exportando Resultados

O notebook já gera visualizações automaticamente. Para salvar:

#### Salvar gráficos:
```python
# Adicione antes de plt.show()
plt.savefig('grafico_sarima.png', dpi=300, bbox_inches='tight')
```

#### Exportar previsões:
```python
# Após célula 27
previsoes_df = pd.DataFrame({
    'periodo': range(1, h+1),
    'valor_real': test.values,
    'valor_previsto': previsoes_sarima
})
previsoes_df.to_csv('previsoes_sarima.csv', index=False)
print("✅ Previsões salvas em previsoes_sarima.csv")
```

---

## 📊 Exemplo Completo

```python
# 1. Carregar dados de vendas diárias
df = pd.read_csv('vendas_diarias.csv')
df['data'] = pd.to_datetime(df['data'])
coluna_valor = 'vendas'

# 2. Ajustar período sazonal (padrão semanal)
periodo_sazonal = 7

# 3. Ajustar horizonte de teste (2 semanas)
h = 14

# 4. Executar todas as células

# 5. Resultado esperado:
# - Série diferenciada (d=1)
# - Sazonalidade forte detectada (>0.6)
# - Modelo SARIMA(1,1,1)(1,1,1,7)
# - MAPE < 15% (bom para vendas)
# - Resíduos = ruído branco ✅
```

---

## 🎯 Boas Práticas

1. ✅ **Sempre visualize** seus dados antes de modelar
2. ✅ **Teste múltiplos períodos sazonais** se não tiver certeza (7, 12, 30)
3. ✅ **Avalie o Top 5 modelos**, não apenas o melhor BIC
4. ✅ **Valide com dados out-of-sample** (holdout adicional)
5. ✅ **Documente** os parâmetros escolhidos e suas justificativas
6. ✅ **Compare com base models** simples (do notebook 1)

---

## 📚 Quando Usar SARIMA?

✅ **Use SARIMA quando:**
- Série tem padrão sazonal claro (semanal, mensal, anual)
- Força da sazonalidade > 0.3
- Precisa de previsões de curto/médio prazo (até 30 períodos)

❌ **Não use SARIMA quando:**
- Série tem múltiplas sazonalidades (diária + semanal)
- Série muito longa (>10.000 obs) → prefira Prophet ou modelos de DL
- Série com quebras estruturais frequentes
- Precisa de interpretabilidade extrema → prefira regressão linear

---

## 🆘 Suporte

Dúvidas ou problemas?
1. Releia este guia
2. Consulte o relatório completo: `REVISAO_NOTEBOOKS.md`
3. Revise os PDFs de referência (Aulas 2-7)

---

**Criado por:** Wolf AI  
**Data:** 14/05/2025  
**Versão:** 1.0
