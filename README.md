# Previsão da Demanda de Bicicletas Compartilhadas — Seul (Exame B)

**Dupla:** Theo Machado Silveira Mello e Claudio Henrique Lopes Mendes  
**Disciplina:** Probabilidade e Estatística
**Base de dados:** Seoul Bike Sharing Demand (`bike_sharing.csv`)

---

## Visão Geral do Projeto

Este repositório reúne o pipeline completo de análise exploratória, inferência estatística e modelagem preditiva aplicado à demanda horária por aluguel de bicicletas públicas em Seul (`Rented_Bike_Count`), investigando como fatores meteorológicos, sazonais e de calendário determinam o comportamento dos usuários.

O trabalho está estruturado em três etapas progressivas e metodologicamente encadeadas:

1. **Etapa 1 — Análise Exploratória e Higienização Física (`etapa1_eda.ipynb`)**
   * **Reconstrução termodinâmica:** identificação e correção de 20 registros com zeros fisicamente impossíveis em `Humidity`, `Dew_Point_Temperature` e `Temperature` por meio da **fórmula psicrométrica de Magnus**, preservando a consistência física entre as variáveis em vez de descartar amostras.
   * **Saneamento operacional:** filtragem das horas em que o sistema esteve fechado (`Functioning_Day == "No"`, totalizando 234 horas / 12 dias sem operação), evitando confundir indisponibilidade do serviço com demanda nula.
   * **Diagnóstico estrutural:** descoberta da forte autocorrelação serial horária ($r_1 \approx 0{,}90$), do ciclo diário de 24 horas e da assimetria da variável resposta.

2. **Etapa 2 — Testes de Hipótese e Inferência Estatística (`etapa2_testes_hipotese.ipynb`)**
   * **Controle de pseudorreplicação:** consolidação dos dados no **nível diário** ($N = 353$ dias operacionais, via média horária diária para neutralizar os ~20% de horas omitidas para teste cego) nas hipóteses de atributos do dia (H1, H2 e H4), e **pareamento por bloco** `(Estação, Hora)` na hipótese intradiária (H3).
   * **Rigor inferencial:** diagnóstico formal das suposições paramétricas (Shapiro-Wilk, QQ-plots e teste de Levene), substituição por testes não paramétricos e de reamostragem por permutação ($10.000$ iterações) quando violadas, visualização da distribuição nula sob $H_0$, cálculo de tamanhos de efeito e análise de poder estatístico.

3. **Etapa 3 — Engenharia de Atributos e Modelagem Preditiva (`etapa3_modelagem.ipynb`)**
   * Tradução direta das evidências inferenciais em especificações de modelo (transformação logarítmica da resposta, indicadores binários de precipitação, termos não lineares de temperatura, interações horárias e variáveis defasadas).

---

## Resumo dos Resultados (Etapa 2)

| Hipótese | Pergunta Investigada | Abordagem Estatística | Tamanho de Efeito | Decisão ($\alpha = 0{,}05$) |
| :--- | :--- | :--- | :--- | :--- |
| **H1** | A demanda difere entre as estações do ano? | Kruskal-Wallis + pares com correção de Holm ($353$ dias) | $\varepsilon^2 = 0{,}53$ (efeito grande) | **Rejeita $H_0$ ($p < 10^{-39}$):** todas as estações diferem ($\text{Inverno} \ll \text{Primavera} < \text{Outono} < \text{Verão}$). |
| **H2** | Feriados alteram a demanda, controlada a estação? | ANOVA 2 fatores (Tipo II) + permutação estratificada ($353$ dias) | $\eta^2_p = 0{,}007$ ($-14\%$ na demanda) | **Não rejeita $H_0$ ($p \approx 0{,}08$):** a queda ingênua ($p \approx 10^{-14}$) é Paradoxo de Simpson pela concentração de feriados no inverno. |
| **H3** | A chuva reduz a demanda na mesma hora e estação? | Wilcoxon pareado + teste do sinal ($64$ células pareadas) | $r_{\text{rb}} = -1{,}00$ (queda mediana de $88\%$) | **Rejeita $H_0$ ($p < 10^{-10}$):** queda consistente em $63$ de $64$ células, operando como barreira quase binária. |
| **H4** | O calor excessivo reduz a demanda? | Spearman por regime térmico + permutação ($\alpha_{\text{adj}} = 0{,}025$) | $\rho = +0{,}78$ ($< 25\,^\circ\text{C}$)<br>$\rho = -0{,}64$ ($\ge 25\,^\circ\text{C}$, secos) | **Rejeita $H_0$ ($p \le 10^{-4}$):** relação em arco; demanda cresce até $\approx 25\,^\circ\text{C}$ e recua nos dias quentes e secos. |

---

## Diretrizes para a Modelagem Preditiva (Etapa 3)

Os achados estatísticos orientam diretamente a arquitetura preditiva do projeto:

* **Escala da variável resposta:** modelar $\log(Y + 1)$ estabiliza a variância entre estações e acomoda efeitos multiplicativos (como quedas percentuais sob chuva).
* **Precipitação como barreira binária:** incluir a flag `Is_Rainy` (`Rainfall > 0`) captura o choque imediato de $-88\%$ na demanda melhor do que apenas o volume contínuo em milímetros.
* **Não-linearidade térmica:** incorporar termo quadrático (`Temperature²`), *splines* ou algoritmos baseados em árvores para capturar a inversão de tendência acima de $25\,^\circ\text{C}$.
* **Interação de calendário:** testar `Holiday * Hour` em vez de `Holiday` isolada, refletindo a substituição dos picos pendulares (8h e 18h) por uso de lazer em feriados.
* **Dependência temporal:** incluir variáveis defasadas (`lag_1h`, `lag_24h`) e médias móveis para absorver a autocorrelação serial diagnosticada nos resíduos.

---

## Estrutura do Repositório

```text
.
├── notebooks/
│   ├── etapa1_eda.ipynb               # Higienização física (Magnus) e análise exploratória
│   ├── etapa2_testes_hipotese.ipynb   # Testes de hipótese (H1–H4), permutação e poder estatístico
│   └── etapa3_modelagem.ipynb         # Engenharia de atributos e modelos preditivos
├── bike_sharing.csv                   # Conjunto de dados meteorológicos, temporais e de demanda
└── README.md                          # Documentação geral do projeto
```

---

## Como Executar

### Google Colab (Recomendado)
Todos os notebooks são **100% autocontidos**: utilizam apenas bibliotecas pré-instaladas no ambiente padrão do Google Colab (`pandas`, `numpy`, `scipy`, `statsmodels`, `matplotlib`, `seaborn` e `scikit-learn`) e realizam o download automático de `bike_sharing.csv` caso o arquivo não esteja no diretório local.

### Ambiente Local
1. Clone o repositório e acesse a pasta do projeto:
   ```bash
   git clone <url-do-repositorio>
   cd <nome-da-pasta>
   ```

2. Instale as dependências:
   ```bash
   pip install pandas numpy scipy statsmodels matplotlib seaborn scikit-learn jupyter
   ```

3. Inicie o Jupyter e execute os notebooks na ordem das etapas (`etapa1` → `etapa2` → `etapa3`):
   ```bash
   jupyter notebook
   ```
