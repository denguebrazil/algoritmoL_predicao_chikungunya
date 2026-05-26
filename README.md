# Algoritmo L: Análise Preditiva e Modelagem Epidemiológica de Chikungunya

Este repositório contém o Algoritmo L, uma ferramenta desenvolvida em Python (Jupyter Notebook) focada no processamento de dados, correção por atraso de notificação (Nowcasting), cálculo de transmissibilidade em tempo real (Rt) e modelagem preditiva de surtos de Chikungunya. O algoritmo foi calibrado e validado utilizando dados oficiais do SINAN para o município de Dourados/MS.

## Arquitetura e Pipeline de Processamento

`[Dados Brutos] -> [Filtros de Agravo/Data] -> [Saneamento Temporal] -> [Nowcasting Corrigido] -> [Estimação de Rt] -> [Projeção Log-Normal]`

1. **Saneamento e Filtragem de Agravos:** Filtragem das notificações de Chikungunya mediante eliminação de falsos positivos e diagnósticos cruzados (exclusão intencional de registros de Dengue e casos Descartados).
2. **Integridade da Série Temporal:** Garantia de linearidade cronológica através de reindexação de todas as semanas epidemiológicas. Semanas sem registros de casos são preenchidas com zero (0), impedindo a supressão de períodos e distorções no cálculo de taxas de crescimento.
3. **Estratificação Étnico-Demográfica:** Separação das séries temporais entre população Urbana (Não Indígena) e População Indígena, respeitando as dinâmicas de transmissão e as diferenças logísticas de notificação de cada localidade.
4. **Nowcasting Empírico:** Correção matemática em tempo real para mitigar o viés do atraso de digitação na Semana Epidemiológica (SE) em aberto e semanas imediatamente anteriores.
5. **Cálculo do Rt (Transmissibilidade):** Estimação do número de reprodução efetivo com base em regressão log-linear local sobre a curva de incidência.
6. **Ajuste de Curva Preditiva (Curve Fitting):** Modelagem não linear via distribuição Log-Normal para projetar a data do pico, volume crítico de casos semanais e o arrasto temporal (cauda longa) estimado do surto.

## Metodologia e Fundamentação Matemática

### Nowcasting Dinâmico (Correção de Atraso)

As notificações enviadas ao SINAN sofrem atrasos logísticos de digitação. Para mitigar a sub-representação de casos na semana atual (SE em aberto), o algoritmo calcula a fração cronológica transcorrida da semana e aplica um fator multiplicador empírico, condicionado a um teto restritivo (cap) para prevenir extrapolações irreais.

* **Urbana** (Atraso logístico menor, ~5 dias): Fator base 1.20
* **Indígena** (Atraso logístico maior, ~10 dias): Fator base 1.45

### Estimação do Número de Reprodução Efetivo (Rt)

Aplica-se uma abordagem de regressão log-linear de taxa de crescimento intrínseca (r). O algoritmo isola os casos corrigidos em uma janela móvel recente (excluindo a semana em aberto para manter a estabilidade estatística) e aplica uma regressão linear sobre o logaritmo dos casos. O Rt é calculado exponenciando a taxa com base no Intervalo Serial da Chikungunya, estimado na literatura em 1.0 semana.

**Classificação de Risco Epidemiológico:**

* **Rt > 1.2:** Alerta Máximo (ALTA TRANSMISSÃO)
* **1.0 < Rt <= 1.2:** Transmissão Ativa (EM CRESCIMENTO)
* **0.8 <= Rt <= 1.0:** Platô de Transmissão (ESTABILIDADE)
* **Rt < 0.8:** Controle de Surto (DESACELERAÇÃO)

### Modelo Preditivo Assimétrico (Log-Normal)

Para projetar o comportamento prospectivo da curva, o código utiliza o método de mínimos quadrados não-lineares (`scipy.optimize.curve_fit`). Em substituição a modelos simétricos gaussianos, implementou-se a função **Log-Normal**. Esta distribuição captura adequadamente a assimetria característica de surtos de arboviroses: uma ascensão exponencial íngreme seguida por uma regressão gradual prolongada (cauda longa).

**Lógica Adaptativa de Limites (Bounds):**
Os parâmetros de ajuste (amplitude, dispersão e locação espacial) são restringidos dinamicamente baseados no volume total de casos acumulados (área sob a curva) e na fase da epidemia:

* **Fase Ascendente:** A amplitude limite é dimensionada para permitir a alocação de um volume projetado superior ao quantitativo atual. O pico é estimado para o futuro (SE atual até SE atual + 15).
* **Fase Descendente (Rt < 0.95):** O modelo identifica a transição de fase, autoriza o retrocesso do parâmetro de localização (reconhecendo o pico no passado cronológico) e direciona o processamento matemático para o ajuste fino da cauda de regressão.

## Métricas de Saída e Visualização Especializada

O script gera renderizações gráficas de alta fidelidade visual através da biblioteca `matplotlib.gridspec`, dividindo a saída em áreas de plotagem e painéis textuais ancorados.

**Painel Gráfico Principal:**

* **Barras Verde-Escuras:** Casos consolidados brutos.
* **Barras Verde-Claras:** Estimativa corrigida por Nowcasting.
* **Barra Amarela:** Demarcação da Semana Epidemiológica atual em aberto.
* **Linha Tracejada Vermelha:** Projeção Log-Normal estendida até a SE52, com preenchimento sombreado indicando o intervalo de incerteza do modelo.

**Painel de Monitoramento (Rodapé):**

* **Taxa de Ataque Observada:** Calculada dinamicamente com base nas estimativas populacionais (Área Urbana: 220.367 hab. | Área Indígena: 23.000 hab.).
* **Integração de Casos Projetados:** O volume total previsto para o ciclo anual é obtido via integração numérica trapezoidal da curva (`np.trapezoid` ou `np.trapz`).
* **Delimitação Temporal do Surto:** Cálculo automático das semanas de início e término operacional, fundamentado em um limiar estrito de atividade (5% da carga projetada no pico).

## Indicadores Consolidados (Console e Gráfico)

Os relatórios emitidos contêm o seguinte escopo de indicadores de tomada de decisão:

| Indicador | Descrição Técnica |
| --- | --- |
| **Casos Brutos Acumulados** | Somatório absoluto de casos notificados e consolidados. |
| **Casos Estimados (Nowcasting)** | Carga epidemiológica projetada com correção de atrasos. |
| **Taxa de Ataque** | Percentual da população acometida com base no censo atual. |
| **Rt Corrente** | Taxa de reprodução efetiva e classificação de status de crescimento. |
| **Pico Projetado** | Semana Epidemiológica (SE) estimada de maior incidência. |
| **Duração do Surto** | Quantidade de semanas compreendidas entre a subida e a queda. |
| **Encerramento Técnico** | SE estimada em que os casos retornarão ao nível de ruído endêmico. |
| **Recomendação** | Diretrizes baseadas na dinâmica de transmissão vigente. |

## Tecnologias Utilizadas

* **Python 3.x**
* **Pandas & NumPy:** Engenharia de dados, reindexação temporal, filtros relacionais e manipulação vetorial.
* **SciPy:** Otimização matemática não-linear espacial (`curve_fit`) e estatística inferencial paramétrica (`linregress`, `lognorm`).
* **Matplotlib & Seaborn:** Configuração e renderização de dashboards de saída analítica.

## Instruções de Execução

1. Posicione a base de dados em formato Excel no mesmo diretório de execução, renomeada para: `sinan_dourados_base.xlsx`.
2. Execute todas as células (`Restart & Run All`) do notebook `analise_preditiva_chik.ipynb`.
3. O pipeline emitirá o painel gráfico final e salvará automaticamente os seguintes arquivos de auditoria intermediária:
* `sinan_dourados_notificados.xlsx`
* `sinan_dourados_provaveis.xlsx`
* `sinan_dourados_urbano.xlsx`
* `sinan_dourados_indigena.xlsx`



## Limitações Técnicas e Considerações

* **Aproximação Determinística:** O modelo Log-Normal oferece uma adequação superior para a dinâmica de transmissão vetorial, contudo, variáveis biológicas não previsíveis, desastres climáticos ou intervenções governamentais abruptas podem alterar o padrão real de difusão.
* **Limites de Convergência:** O algoritmo possui detecção de falha de convergência. Em cenários de incipiente subnotificação estrutural ou extrema dispersão (ruído), a regressão pode ser abortada preventivamente pelo bloco lógico.
* **Extrapolação Temporal:** Projeções estendidas por margens de tempo prolongadas comportam elevado grau de incerteza em detrimento da estabilidade do curto prazo.
* **Escopo Geográfico:** O setup de variáveis populacionais e empíricas deste notebook encontra-se calibrado especificamente para o contexto demográfico do município de Dourados, MS.

## Contribuição e Manutenção

Contribuições ao repositório são encorajadas sob o escopo metodológico. Operações aceitas incluem:

* Reporte e submissão de correções de bugs lógicos.
* Refatoração e aprimoramento do modelo estatístico e matrizes matemáticas.
* Expansão da interoperabilidade de dados do SINAN.
* Aprimoramentos da documentação científica.

**Autoria Original e Propriedade Intelectual:** Algoritmo base concebido por **Dengue Brazil**, focado no processamento de dados epidemiológicos voltados ao monitoramento contínuo de Dengue, Zika e Chikungunya em território nacional.
