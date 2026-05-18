#### Algoritmo L: Análise Preditiva e Modelagem Epidemiológica de Chikungunya

Este repositório contém o Algoritmo L, uma ferramenta desenvolvida em Python (Jupyter Notebook) focada no processamento, correção por atraso de notificação (Nowcasting), cálculo de transmissibilidade em tempo real ($R_t$) e modelagem preditiva de surtos de Chikungunya. O algoritmo foi calibrado e validado utilizando dados reais do SINAN para o município de Dourados/MS, referente ao ano de 2026.

---

#### Pipeline

[Dados Brutos SINAN] ➔ [Filtros de Agravo/Data] ➔ [Estratificação População]
                                                          │
[Projeção Gaussiana] ┪ [Estimação de Rt] ┫ [Nowcasting Corrigido (SE Aberta)]

- Saneamento e Filtragem de Agravos: Isolamento das notificações de Chikungunya eliminando falsos positivos, linhas vazias de semanas epidemiológicas e diagnósticos cruzados (exclusão intencional de registros de Dengue e casos Descartados).

- Estratificação Étnico-Demográfica: Separação das séries temporais entre população Urbana (Não Indígena) e População Indígena, respeitando as dinâmicas de transmissão e as diferenças logísticas de notificação de cada localidade.

- Nowcasting Empírico: Correção matemática em tempo real para mitigar o impacto do atraso de digitação na Semana Epidemiológica (SE) em aberto e semanas imediatamente anteriores.

- Cálculo do $R_t$ (Transmissibilidade): Estimação do número de reprodução efetivo com base em regressão log-linear local.

- Ajuste de Curva Preditiva (Curve Fitting): Modelagem não linear via função Gaussiana para projetar a data do pico, volume crítico de casos por semana e a duração estimada do surto.

#### Nuances e Metodologias Matemáticas

1. Nowcasting Dinâmico (Correção de Atraso): As notificações enviadas ao SINAN sofrem atrasos crônicos de digitação. Para evitar o efeito visual de "falsa queda" na semana atual (SE em aberto), o algoritmo calcula a fração transcorrida da semana (fracao_semana) e aplica um fator multiplicador empírico combinado com um teto de segurança (cap) para evitar distorções.

Tabela de Referência Operacional (FATOR_ATRASO_SINAN):
- Urbana (Atraso logístico menor ~5 dias): Fator 1.20
- Indígena (Atraso logístico maior ~10 dias): Fator 1.45

2. Estimação do Número de Reprodução Efetivo ($R_t$): Baseado na metodologia clássica de Wallinga & Lipsitch (2007), o algoritmo isola os casos corrigidos em uma janela móvel de 4 semanas (excluindo a semana em aberto para manter a estabilidade). Aplica-se uma regressão linear sobre o logaritmo dos casos para extrair a taxa de crescimento instantânea ($r$).

O $R_t$ é calculado exponenciando a taxa com base no Intervalo Serial ($T_s$) da Chikungunya, estimado na literatura em 1 semana ($1.0$).

#### Classificação de Risco
- $R_t > 1.2$: Alerta Máximo (ALTA TRANSMISSÃO)
- $1.0 < R_t \le 1.2$: Transmissão Ativa (EM CRESCIMENTO)
- $0.8 \le R_t \le 1.0$: Platô de Transmissão (ESTABILIDADE)
- $R_t < 0.8$: Controle de Surto (DESACELERAÇÃO)

3. Modelo Preditivo Gaussiano de Surto: para prever o comportamento futuro da curva, o código utiliza o método dos mínimos quadrados não-lineares (scipy.optimize.curve_fit) parametrizando uma função matemática Gaussiana

Lógica Adaptativa de Fase (Nuance Crítica do Código): A grande inteligência do script reside em identificar a fase epidêmica atual para condicionar as fronteiras (bounds) do ajuste.

- Se em Fase Ascendente: Obriga a amplitude a ser maior que o pico atual observado e projeta o pico para semanas futuras ($SE_{atual}$ até $SE_{atual} + 15$).

- Se em Fase Descendente ($R_t < 0.95$): Libera o parâmetro para retroagir cronologicamente (identificando que o pico já passou) e trava a amplitude próxima ao pico histórico consolidado, focando o ajuste em desenhar a cauda de regressão.

#### Métricas de Saída e Visualização Especializada
O script gera figuras ricas utilizando matplotlib.gridspec, acoplando gráficos de tendência e um painel analítico de tomada de decisão

Painel Superior (Gráfico Epidêmico Principal):
- Barras Verde-Escuras: Casos consolidados brutos já digitados.
- Barras VerdeClaras: Projeção de Nowcasting com ganho teórico de atraso.
- Barra Amarela Destacada: Identificação visual da semana epidemiológica atual em aberto.
- Linha Tracejada Vermelha: Curva Gaussiana projetada até a SE52 acompanhada de uma banda sombreada (fill_between) que ilustra o intervalo de incerteza do modelo.

Painel Inferior (Rodapé de Monitoramento Operacional):
- Taxa de Ataque Dinâmica: Calculada em tempo real com base no tamanho estimado das respectivas populações (Urbana: 220.367 hab. | Indígena: 23.000 hab.)
- Totalização por Integração: O número final de casos estimados para todo o ano é calculado via integração numérica trapezoidal da curva projetada (np.trapezoid ou np.trapz).
- Janela Temporal do Surto: Cálculo automatizado das semanas de início, duração e fim do surto utilizando um limiar de corte de 5% da cauda.

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

#### Tecnologias Utilizadas
- Python 3.x
- Pandas & NumPy: Engenharia de atributos, filtros relacionais e manipulação vetorial.
- SciPy: Otimização não-linear (curve_fit) e estatística inferencial linear (linregress).
- Matplotlib & Seaborn: Renderização de dashboards gráficos de alta fidelidade visual.

#### Como Executar
Posicione a base de dados do SINAN no mesmo diretório do arquivo: sinan_dourados_base.xlsx.

Execute a esteira completa do notebook analise_preditiva_chik.ipynb.

Os seguintes arquivos intermediários de auditoria serão gerados automaticamente para conferência:
- sinan_dourados_notificados.xlsx
- sinan_dourados_provaveis.xlsx
- sinan_dourados_urbano.xlsx
- sinan_dourados_indigena.xlsx

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

Repositório mantido por: **denguebrazil**
Para questões relacionadas a dados epidemiológicos de Dengue, Zika e Chikungunya no Brasil.
