# Análise de Tendência de Chikungunya

> Modelo epidemiológico para **estimar a tendência futura** de casos de Chikungunya em um município ou estado, corrigindo o atraso de notificação (*nowcasting*), calculando a velocidade de transmissão (Rt) e projetando o comportamento provável do surto nas semanas seguintes.

---

## Sobre o projeto

Diferente de um relatório descritivo — que mostra **o que já aconteceu** —, este projeto foi construído para responder a uma pergunta mais difícil: **"para onde a epidemia está indo?"**

Isso é possível porque casos de Chikungunya não aparecem nos sistemas de vigilância no mesmo dia em que acontecem: existe um **atraso entre o início dos sintomas e a notificação chegar ao sistema**. Esse atraso faz com que as semanas mais recentes pareçam ter "poucos casos" só porque a notificação ainda está em andamento — um efeito conhecido como **subnotificação temporária**. O notebook corrige esse efeito e, a partir dos dados corrigidos, projeta uma curva epidêmica completa.

Em resumo, o notebook `chik_analise_tendencia.ipynb` executa, para um único município por vez:

1. **Cálculo do atraso histórico de notificação** (tempo entre o início dos sintomas e a digitação do caso no sistema);
2. **Nowcasting**: correção matemática das últimas semanas, estimando quantos casos ainda estão "escondidos" pelo atraso;
3. **Cálculo do número de reprodução (Rt)**: velocidade com que a doença está se espalhando naquele momento;
4. **Classificação de alerta epidemiológico** (alta, em crescimento, estabilidade ou desaceleração);
5. **Projeção da curva do surto** usando um modelo estatístico log-normal, indicando pico esperado, duração provável e intervalo de incerteza;
6. **Geração de um gráfico final único**, com a curva histórica, a curva projetada e um painel de indicadores técnicos.

> **Não é preciso ser programador, estatístico ou epidemiologista para rodar este notebook.** Basta preencher algumas informações do seu município no início do código e executar as células — as instruções completas estão na seção [Como usar](#-como-usar).

> ⚠️ **Este notebook depende do projeto de análise descritiva.** Ele utiliza como base o arquivo `chik_brasil_provaveis_2026_completo.xlsx`, gerado pelos notebooks do repositório de [análise de casos notificados e prováveis de Chikungunya](#). Ou seja, recomenda-se rodar primeiro aquele projeto para gerar essa base antes de utilizar este notebook.

---

## Principais conceitos utilizados (explicados de forma simples)

| Conceito | O que significa, em termos simples |
|---|---|
| **Nowcasting** | Técnica que "corrige o presente": estima quantos casos das últimas semanas ainda vão aparecer no sistema, já que a notificação tem atraso. |
| **Rt (número de reprodução efetivo)** | Indica, em média, quantas pessoas cada caso está transmitindo a doença no momento atual. Rt acima de 1 significa que a epidemia está crescendo; abaixo de 1, que está diminuindo. |
| **Curva log-normal** | Modelo matemático usado para descrever o formato típico de uma curva de epidemia (crescimento rápido, pico e queda mais lenta). É usada aqui para estimar quando o pico de casos deve ocorrer e por quanto tempo o surto deve durar. |
| **Taxa de ataque** | Proporção da população do município que já foi infectada, considerando os casos notificados até o momento. |
| **Intervalo de incerteza** | Faixa de valores (linha sombreada no gráfico) que representa a margem de erro da projeção — toda projeção estatística tem incerteza, e isso é mostrado de forma transparente. |

---

## O que o projeto entrega

Ao final da execução, é gerado um **único gráfico consolidado (.png)**, contendo:

- Barras com os **casos notificados** por semana epidemiológica;
- Barras sobrepostas com a **estimativa corrigida (nowcasting)** para as últimas semanas;
- Destaque visual para a **semana epidemiológica em aberto** (ainda em atualização);
- Uma **curva de projeção log-normal**, com faixa de incerteza, indicando o pico esperado do surto;
- Um **painel de indicadores epidemiológicos**, incluindo: casos acumulados, casos estimados, semana do pico projetado, duração estimada do surto, taxa de ataque, valor de Rt e o status de alerta correspondente;
- Um **painel de notas técnicas**, com a metodologia aplicada, o fator de correção usado e a fonte dos dados.

---

## Fonte de dados utilizada

| Arquivo | Origem | O que contém |
|---|---|---|
| `chik_provaveis_2026.xlsx` | Consolidado a partir do **SINAN** (Sistema de Informação de Agravos de Notificação) | Casos prováveis de Chikungunya, com datas de início de sintomas e de digitação, por município |

> ⚠️ **Atenção — dados sensíveis.** A base contém informações de saúde e não é distribuída neste repositório. Cada usuário deve gerá-la a partir de suas próprias fontes de dados (SINAN), respeitando as normas de proteção de dados e sigilo em saúde vigentes.

---

## Tecnologias usadas

O projeto foi desenvolvido em **Python**, dentro de **Jupyter Notebook**, utilizando as seguintes bibliotecas:

| Biblioteca | Para que serve no projeto |
|---|---|
| [pandas](https://pandas.pydata.org/) | Leitura, filtragem e tratamento da base de dados por município |
| [numpy](https://numpy.org/) | Cálculos numéricos e operações matemáticas do modelo |
| [matplotlib](https://matplotlib.org/) | Construção do gráfico final e do painel de indicadores |
| [seaborn](https://seaborn.pydata.org/) | Estilização visual do gráfico |
| [scipy](https://scipy.org/) | Núcleo estatístico do projeto: ajuste da curva log-normal (`curve_fit`), cálculo de regressão linear para o Rt (`linregress`) e distribuição log-normal (`lognorm`) |
| `datetime` (biblioteca padrão do Python) | Cálculo de datas, atrasos de notificação e data de referência da análise |
| `os` (biblioteca padrão do Python) | Criação automática da pasta onde o gráfico final é salvo |

Não é necessário conhecer essas bibliotecas para usar o projeto — elas trabalham em segundo plano quando o código é executado.

---

## Instalação

### 1. Pré-requisitos

Antes de começar, você precisa ter instalado em seu computador:

1. **Python** (versão 3.9 ou superior) — [baixar aqui](https://www.python.org/downloads/)
2. **Jupyter Notebook** ou **JupyterLab** (ambiente onde o arquivo `.ipynb` é executado)
3. **Git** (opcional, apenas se for clonar o repositório pela linha de comando) — [baixar aqui](https://git-scm.com/downloads)
4. **Editor de Código**, que pode ser o VS Code ou Anaconda

### 2. Baixar o projeto

**Opção A — Sem usar linha de comando (recomendado para quem não é da área técnica):**
1. Clique no botão verde **"Code"** na página do repositório no GitHub;
2. Selecione **"Download ZIP"**;
3. Extraia o arquivo ZIP em uma pasta de fácil acesso no seu computador.

**Opção B — Usando Git (para usuários com experiência técnica):**
```bash
git clone https://github.com/seu-usuario/nome-do-repositorio.git
cd nome-do-repositorio
```

### 3. Instalar as bibliotecas necessárias

Abra o **Prompt de Comando** (Windows), **Terminal** (Mac/Linux) ou o **Anaconda Prompt**, navegue até a pasta do projeto e execute:

```bash
pip install pandas numpy matplotlib seaborn scipy openpyxl
```

> Esse comando instala, de uma só vez, todas as ferramentas que o notebook precisa para funcionar. É necessário ter conexão com a internet apenas nesta etapa.

### 4. Abrir o notebook

Ainda no terminal, dentro da pasta do projeto, execute:

```bash
jupyter notebook
```

Isso abrirá o Jupyter no seu navegador. Basta clicar no arquivo `analise_preditiva_chik.ipynb` para começar.

---

## Estrutura de arquivos esperada

Para que o notebook funcione corretamente, o seguinte arquivo deve estar **na mesma pasta** do notebook antes da execução:

```
📦 pasta-do-projeto
 ┣ 📜 analise_preditiva_chik.ipynb
 ┗ 📜 chik_brasil_provaveis_2026_completo.xlsx   (base consolidada de casos prováveis, gerada previamente)
```

O gráfico final é gerado automaticamente dentro de uma nova pasta, `Tendencia_Chikungunya/`, criada na mesma pasta do notebook.

---

## Como usar

1. Certifique-se de que o arquivo `chik_provaveis_2026.xlsx` está na mesma pasta do notebook;
2. Abra o notebook `chik_analise_tendencia.ipynb` no Jupyter;
3. Preencha o dicionário de variáveis no início do código com as informações do município desejado:

   ```python
   nome_municipio = 'XXXXXXXX'    # NÃO usar acento
   mun_sinan = 000000             # Código IBGE do município
   populacao = 000_000            # População do município (fonte: IBGE) - Separar os milhares por "_"
   data_dados = 'dd/mm/aaaa'      # Data de referência dos dados
   ```

4. Execute todas as células do notebook, de cima para baixo (no menu do Jupyter: **Cell → Run All**);
5. Ao final, o gráfico com a projeção e os indicadores estará salvo na pasta `Tendencia_Chikungunya/`, com o nome `chik_tendencia_<nome_municipio>.png`.

> ATENÇÂO: Como o modelo depende do histórico de casos do município para funcionar bem, recomenda-se que existam **pelo menos algumas semanas consecutivas com casos notificados** antes de confiar na projeção. Municípios com poucos casos ou séries muito curtas podem gerar estimativas menos confiáveis — por isso o gráfico sempre exibe a faixa de incerteza do modelo.

---

## Limitações e uso responsável

Este é um **modelo estatístico de apoio à decisão**, não uma previsão exata do futuro. Algumas limitações importantes:

- A qualidade da projeção depende diretamente da qualidade e completude dos dados notificados no SINAN;
- O modelo assume um comportamento log-normal típico de surtos — cenários atípicos (ex.: mudanças bruscas de política de testagem, sub-registro extremo, intervenções de controle vetorial) podem não ser capturados;
- O fator de nowcasting é uma estimativa baseada no atraso histórico de notificação do próprio município, e pode variar em períodos de sobrecarga do sistema de saúde;
- Os resultados devem ser interpretados, sempre em conjunto com o conhecimento de campo da equipe local, e não como substituto do julgamento técnico.

---

## Como contribuir

Contribuições são muito bem-vindas, seja você da área de dados, epidemiologia e/ou estatística!

1. Faça um **fork** deste repositório;
2. Crie uma branch para sua alteração:
   ```bash
   git checkout -b minha-melhoria
   ```
3. Faça suas alterações e registre-as:
   ```bash
   git commit -m "Descrição clara da alteração realizada"
   ```
4. Envie sua branch para o GitHub:
   ```bash
   git push origin minha-melhoria
   ```
5. Abra um **Pull Request** descrevendo o que foi alterado e por quê.

**Sugestões de contribuição especialmente bem-vindas:**
- Validação estatística do modelo (ex.: comparação entre projeções passadas e casos reais observados depois);
- Suporte para rodar a análise para vários municípios de uma vez;
- Testes com outros modelos de curva epidêmica além da log-normal;
- Documentação e exemplos adicionais para usuários não técnicos.

Se preferir, também é possível contribuir apenas relatando problemas (*issues*) ou sugestões, sem a necessidade de submeter código.

---

## Licença

Este projeto está disponível sob os termos que forem definidos pelo mantenedor do repositório. Recomenda-se incluir uma licença de código aberto (como [MIT](https://opensource.org/licenses/MIT) ou [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.html)) para deixar claro como o código pode ser usado, modificado e distribuído.

> A **licença do código** é independente da **confidencialidade dos dados de saúde** utilizados como insumo. Mesmo que o código seja aberto, os dados do SINAN devem continuar sendo tratados conforme a legislação vigente de proteção de dados e sigilo em saúde.

---

## Créditos

Algoritmo de nowcasting e projeção epidemiológica desenvolvido com foco em apoiar equipes de vigilância na antecipação de tendências de surtos de arboviroses.
