# Predição de Casos de Chikungunya

## 📋 Descrição Resumida

Este repositório contém uma análise preditiva completa de casos de Chikungunya no município de Dourados, Mato Grosso do Sul. O objetivo é explorar, tratar e modelar dados epidemiológicos de casos de Chikungunya, construindo modelos estatísticos capazes de realizar previsões em horizonte de curto e médio prazo, auxiliando ações de vigilância em saúde e planejamento de recursos.

Os dados são provenientes do SINAN (Sistema de Informação de Agravos de Notificação) e cobrem casos notificados em áreas urbanas, indígenas e totais do município.

---

## 📁 Estrutura do Repositório

| Arquivo | Descrição |
|---------|----------|
| `analise_preditiva_chik.ipynb` | Notebook principal com toda a análise preditiva |
| `README.md` | Documentação do projeto |

---

## 🔧 Funcionalidades Implementadas

### 1. Importação e Configuração de Bibliotecas

O código inicia importando as bibliotecas essenciais para análise de dados e visualização:

- **pandas**: Manipulação e análise de dados tabulares
- **numpy**: Operações numéricas e arrays
- **matplotlib**: Criação de gráficos e visualizações
- **seaborn**: Estilização avançada de gráficos estatísticos
- **scipy.optimize.curve_fit**: Ajuste de curvas não-lineares
- **scipy.interpolate**: Interpolação de dados

As configurações de visualização incluem temas escuros, fontes personalizadas e paletas de cores específicas para representar as diferentes áreas (urbana, indígena, total).

---

### 2. Carregamento dos Dados

Os dados são carregados a partir de arquivos Excel do SINAN localizados em um diretório específico do sistema. O processo inclui:

- Leitura de múltiplas planilhas Excel do diretório `C:/Users/dengu/OneDrive/Documentos/DENGUE/DADOS/SINAN/Dourados.xlsx`
- Criação de um dicionário contendo os dados por área (urbana, indígena, total)
- Identificação automática de intervalos de datas para cada conjunto de dados
- Extração da coluna `dt_notific` (data de notificação) como série temporal principal

---

### 3. Nowcasting (Correção de Subnotificação)

O nowcasting é uma técnica estatística para corrigir o viés de subnotificação nos dados mais recentes. Como os casos demoram a ser notificados, os últimos dias/semanas parecem ter menos casos do que realmente ocorreram.

**Metodologia implementada:**

1. **Cálculo de atrasos de notificação**: Para cada dia, calcula-se a diferença entre a data de notificação e a data atual
2. **Matriz de atrasos**: Construção de uma matriz onde cada célula representa o número de casos notificados em cada dia de atraso
3. **Filtragem**: Apenas os últimos 100 dias de dados são utilizados para estabilidade
4. **Suavização exponencial**: Aplicação de decaimento exponencial para regularizar as contagens de atraso
5. **Fatores de correção**: Cálculo de um fator para cada dia que estima quantos casos adicionais devem ser adicionados
6. **Correção final**: Ajuste das contagens recentes multiplicando pelos fatores de correção

Esta etapa é crítica porque sem ela, os modelos preditivos subestimariam significativamente o número real de casos recentes.

---

### 4. Ajuste de Curva Gaussiana

O modelo principal de predição utiliza o ajuste de uma curva gaussiana (distribuição normal) aos dados epidemiológicos. A gaussiana é apropriada porque surtos de doenças infecciosas frequentemente seguem um padrão de crescimento e declínio simétrico.

**Função gaussiana utilizada:**

```
f(t) = base + amp * exp(-(t - centro)^2 / (2 * sigma^2))
```

Onde:
- **base**: Linha de base de casos (valor mínimo)
- **amp**: Amplitude máxima do pico do surto
- **centro**: Data do pico máximo de casos
- **sigma**: Largura do surto (dispersão temporal)

**Processo de ajuste:**

1. Conversão de datas para índices numéricos (dias desde a primeira data)
2. Definição de limites para os parâmetros (bounds) para garantir ajustes fisicamente plausíveis
3. Chute inicial (p0) baseado em estatísticas descritivas dos dados
4. Uso do `curve_fit` com limites restritos para encontrar os parâmetros ótimos
5. Cálculo da data do pico convertendo o índice numérico de volta para formato de data

---

### 5. Projeção e Previsão Futura

O modelo projeta os casos para os próximos 120 dias (aproximadamente 4 meses) a partir da última data disponível.

**Etapas da projeção:**

1. **Criação de datas futuras**: Geração de uma série de datas a partir do dia seguinte à última data conhecida
2. **Aplicação do modelo gaussiano**: Cálculo dos valores previstos para cada dia futuro
3. **Projeção até decaimento**: Extensão adicional até que os casos projetados caiam abaixo de 0.5
4. **Cálculo de datas-chave**:
   - Data do pico máximo
   - Data de fim do surto (quando casos < 1)
   - Dias até o pico
   - Dias até o fim

**Recomendações automáticas baseadas nos resultados:**

- **Aumentar vigilância**: Quando o pico ainda não foi atingido
- **Manter ações**: Durante o pico
- **Monitorar tendência**: Após o pico, durante o declínio
- **Ações preventivas**: Quando o surto está se encerrando

---

### 6. Visualizações e Gráficos

O código gera múltiplas visualizações profissionais para cada área (urbana, indígena, total):

#### Gráfico Principal de Casos vs. Modelo

- **Linha sólida**: Casos observados (dados reais)
- **Linha pontilhada**: Casos ajustados pelo modelo gaussiano
- **Barras empilhadas**: Casos previstos para o futuro, coloridos por nível de risco
  - 🔴 Vermelho: Casos > 100 (alto risco)
  - 🟠 Laranja: Casos > 50 (risco moderado)
  - 🟡 Amarelo: Casos > 25 (risco baixo)
  - 🟢 Verde: Casos <= 25 (risco mínimo)
- **Linha verde pontilhada**: Indica a data atual

#### Gráfico de Tendência de Casos (Twin Axes)

- **Eixo esquerdo**: Casos diários (barras empilhadas)
- **Eixo direito**: Média móvel de 7 dias (linha)
- **Linha vertical tracejada**: Data do pico máximo
- **Sombreamento vermelho**: Região de incerteza do modelo (±1 desvio padrão)

#### Gráfico de Distribuição dos Casos

- Histograma de frequência dos casos
- Sobreposição da curva gaussiana ajustada
- Área sombreada cinza: Distribuição dos dados observados

#### Indicadores em Box (Caixas de Resumo)

Cada gráfico inclui uma caixa com os seguintes indicadores:

| Indicador | Descrição |
|-----------|----------|
| Casos Totais Observados | Soma de todos os casos notificados |
| Casos no Pico | Número máximo diário de casos |
| Data do Pico | Data em que ocorreu o máximo |
| Dias até o Pico | Quantos dias faltam (ou passaram) até o pico |
| Dias até o Fim | Estimativa de dias até o fim do surto |
| R² do Modelo | Qualidade do ajuste (R² > 0.9 = excelente) |
| Recomendação | Sugestão de ação baseada no estágio do surto |

---

### 7. Análise Estatística Avançada

O notebook inclui cálculos estatísticos sofisticados:

- **Média móvel**: Suavização de ruído nos dados diários
- **Desvio padrão**: Medida de variabilidade do modelo
- **Coeficiente de determinação (R²)**: Avaliação da qualidade do ajuste do modelo
- **Interpolação**: Preenchimento de valores faltantes usando interpolação quadrática
- **Máscaras booleanas**: Segmentação inteligente dos dados (observados vs. previstos)

---

## 📊 Resultados Esperados

Para cada área analisada (Urbana, Indígena, Total), o modelo fornece:

1. **Curva ajustada**: Representação matemática do comportamento do surto
2. **Previsão futura**: Estimativa de casos para os próximos meses
3. **Datas críticas**: Identificação de pico e fim do surto
4. **Indicadores de qualidade**: R² e outras métricas de desempenho
5. **Recomendações**: Ações sugeridas para gestores de saúde

---

## 🛠️ Requisitos Técnicos

Para executar o notebook localmente, são necessárias as seguintes bibliotecas Python:

```
pandas
numpy
matplotlib
seaborn
scipy
openpyxl  # Para leitura de arquivos Excel
```

---

## 📈 Como Usar

1. Clone o repositório
2. Instale as dependências listadas acima
3. Abra o notebook `analise_preditiva_chik.ipynb` no Jupyter ou VS Code
4. Execute as células em ordem
5. Analise os gráficos e indicadores gerados

---

## ⚠️ Limitações e Considerações

- **Dados históricos**: O modelo assume que o surto segue um padrão gaussiano, o que pode não ser verdade para todos os cenários epidemiológicos
- **Subnotificação**: Embora o nowcasting corrija parcialmente, a subnotificação pode persistir
- **Extrapolacão**: Previsões muito distantes no futuro têm maior incerteza
- **Dados faltantes**: Dias sem notificações são tratados com interpolação
- **Contexto local**: Os resultados são específicos para Dourados, MS

---

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para:
- Reportar bugs
- Sugerir melhorias no modelo
- Adicionar novas funcionalidades
- Melhorar a documentação

---

## 📄 Licença

Este projeto está sob a licença do repositório original. Consulte o arquivo LICENSE para mais detalhes.

---

## 📞 Contato

Repositório mantido por: **denguebrazil**

Para questões relacionadas a dados epidemiológicos de Dengue, Zika e Chikungunya no Brasil.
